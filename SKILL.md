# SKILL.md — computational-drug-design-skill

This skill guides Claude Code through the full de novo D-peptide inhibitor design pipeline, or any individual step within it. Read this file before executing any pipeline command.

---

## Environment Setup

Always activate the boltz conda environment before running Boltz2 or BoltzGen commands:

```bash
source /home/ubuntu/miniconda/bin/activate boltz
```

Two Python interpreters are used depending on the task:

| Task | Python path |
|------|-------------|
| Boltz2, BoltzGen, data processing | `/home/ubuntu/miniconda/envs/boltz/bin/python3` |
| PyMOL visualization | `/home/ubuntu/miniconda/bin/python3` |

Boltz2 binary: `/home/ubuntu/miniconda/envs/boltz/bin/boltz`  
Boltz2 checkpoint: `~/.boltz/boltz2_aff.ckpt`

---

## Task Classification

When the user gives a request, first determine which mode applies:

- **Full pipeline** — user provides a new target protein and wants end-to-end D-peptide candidates with IC₅₀ scores
- **Single step** — user wants to run one specific stage (e.g. only IC₅₀ scoring, only chirality inversion)

For single-step requests, jump directly to the relevant section below. For full pipeline, execute all steps in order.

---

## Full Pipeline

### Step 1 — Target Protein Input

Collect from the user:
- Target protein name
- Protein sequence (FASTA or plain amino acid string)
- Optional: known binding pocket residues

### Step 2 — AlphaFold3 Pocket Prediction

Use AlphaFold3 to predict the 3D structure and identify the binding pocket.

Output: PDB file of the target protein structure, pocket residue list.

Store results under:
```
structures/<TARGET_NAME>/alphafold/
```

### Step 3 — BoltzGen Peptide Generation

Generate de novo L-peptide candidates conditioned on the binding pocket using BoltzGen.

#### Design specification YAML

Create a YAML file describing the target and what to design. File references are interpreted relative to the YAML file's directory:

```yaml
entities:
  # Target protein extracted from a .cif file
  - file:
      path: <TARGET>.cif
      include:
        - chain:
            id: A
  # Peptide binder to design (length range)
  - protein:
      id: B
      sequence: 5..10   # design peptides of length 5–10
```

First validate your YAML before running:

```bash
boltzgen check <YOUR_DESIGN>.yaml
```

#### Run BoltzGen

```bash
boltzgen run <YOUR_DESIGN>.yaml \
    --output workbench/<TARGET_NAME> \
    --protocol peptide-anything \
    --num_designs 1000 \
    --budget 50
```

Key flags:

| Flag | Notes |
|------|-------|
| `--protocol` | Use `peptide-anything` for peptide binders |
| `--num_designs` | Number of intermediate candidates (recommend 10,000–60,000 for production) |
| `--budget` | Final diversity-optimized set size |
| `--steps` | Optionally run specific steps only: `design inverse_folding folding analysis filtering` |
| `--reuse` | Resume interrupted runs without losing progress |
| `--cache` | Override default model cache path `~/.cache` (~6 GB download on first run) |

#### Pipeline steps available

| Step | Description |
|------|-------------|
| `design` | Generate candidates using diffusion model |
| `inverse_folding` | Redesign sequences using inverse folding model |
| `folding` | Re-fold binders with target using Boltz-2 |
| `analysis` | Compute quality metrics on folded structures |
| `filtering` | Filter and rank designs; select best candidates |

To run only specific steps (e.g. re-run filtering with adjusted thresholds):

```bash
boltzgen run <YOUR_DESIGN>.yaml \
    --output workbench/<TARGET_NAME> \
    --protocol peptide-anything \
    --steps filtering \
    --refolding_rmsd_threshold 3.0 \
    --filter_biased=false \
    --additional_filters 'ALA_fraction<0.3' 'filter_rmsd_design<2.5' \
    --alpha 0.2
```

Store results under:
```
structures/<TARGET_NAME>/boltzgen/
```

### Step 4 — 5-mer Sliding Window Cut

Extract 5-mer fragments from generated full-length peptide sequences.

```python
def sliding_window_5mer(sequence):
    return [sequence[i:i+5] for i in range(len(sequence) - 4)]
```

Run with:
```bash
/home/ubuntu/miniconda/envs/boltz/bin/python3 sliding_window.py \
    --input structures/<TARGET_NAME>/boltzgen/candidates.fasta \
    --output structures/<TARGET_NAME>/fragments/5mers.txt
```

### Step 5 — L → D Chirality Inversion

Invert all L-amino acids to D-amino acids using the pseudo-mirror strategy. In SMILES notation, this means inverting all chiral centers (swap `@` ↔ `@@`).

```python
def invert_chirality(smiles):
    result = []
    i = 0
    while i < len(smiles):
        if smiles[i] == '@':
            if i + 1 < len(smiles) and smiles[i+1] == '@':
                result.append('@')   # @@ → @ (net: swap)
                i += 2
                continue
            else:
                result.append('@@')  # @ → @@
                i += 1
                continue
        result.append(smiles[i])
        i += 1
    return ''.join(result)
```

### Step 6 — SMILES Generation

Convert D-peptide 3D structures to SMILES strings using RDKit.

```python
from rdkit import Chem
mol = Chem.MolFromPDBFile('structure.pdb', removeHs=False)
smiles = Chem.MolToSmiles(mol)
```

Store all SMILES in:
```
structures/<TARGET_NAME>/smiles/d_peptides.txt
```

### Step 7 — Boltz2 IC₅₀ Scoring

#### Input YAML format

Create one YAML file per candidate:

```yaml
version: 1
sequences:
  - protein:
      id: A
      sequence: <TARGET_PROTEIN_SEQUENCE>
  - ligand:
      id: B
      smiles: <D_PEPTIDE_SMILES>
properties:
  - affinity:
      binder: B
```

Save YAMLs to:
```
structures/<TARGET_NAME>/boltz_affinity/yamls/
```

#### Run affinity prediction

```bash
source /home/ubuntu/miniconda/bin/activate boltz

boltz predict structures/<TARGET_NAME>/boltz_affinity/yamls/candidate_0000.yaml \
    --checkpoint ~/.boltz/boltz2_aff.ckpt \
    --out_dir structures/<TARGET_NAME>/boltz_affinity/output/ \
    --devices 2 \
    --accelerator gpu \
    --diffusion_samples 1 \
    --sampling_steps 200
```

Key flags:

| Flag | Default | Notes |
|------|---------|-------|
| `--checkpoint` | — | Always use `~/.boltz/boltz2_aff.ckpt` |
| `--out_dir` | `./predictions` | Set per target |
| `--devices` | 1 | Use 2 for dual RTX 4090 |
| `--accelerator` | gpu | Always gpu |
| `--diffusion_samples` | 1 | Increase for more diversity |
| `--sampling_steps` | 200 | Higher = more accurate, slower |
| `--override` | false | Set true to overwrite existing results |

#### Batch scoring

To score multiple candidates at once:

```bash
source /home/ubuntu/miniconda/bin/activate boltz

for yaml in structures/<TARGET_NAME>/boltz_affinity/yamls/*.yaml; do
    boltz predict "$yaml" \
        --checkpoint ~/.boltz/boltz2_aff.ckpt \
        --out_dir structures/<TARGET_NAME>/boltz_affinity/output/ \
        --devices 2 \
        --accelerator gpu \
        --override
done
```

#### Parse IC₅₀ results

```python
import json, glob, pandas as pd

results = []
for f in glob.glob('structures/<TARGET>/boltz_affinity/output/**/affinity*.json', recursive=True):
    with open(f) as fh:
        data = json.load(fh)
    results.append({
        'candidate': f,
        'ic50_nM': data.get('affinity_pred_value'),
        'confidence': data.get('affinity_probability')
    })

df = pd.DataFrame(results).sort_values('ic50_nM')
df.to_csv('candidates_ranked.csv', index=False)
print(df.head(10))
```

---

## Single Step Reference

If the user only wants one step, go directly to:

| User request | Section |
|---|---|
| "Score IC₅₀ for these SMILES" | Step 7 |
| "Invert chirality of this peptide" | Step 5 |
| "Generate SMILES from PDB" | Step 6 |
| "Extract 5-mers from this sequence" | Step 4 |
| "Visualize the binding pocket" | PyMOL Visualization below |

---

## PyMOL Visualization

Use `/home/ubuntu/miniconda/bin/python3` for all PyMOL work.

Standard rendering settings:

```python
cmd.set('ray_shadows', 0)
cmd.set('ambient', 0.65)
cmd.set('direct', 0.55)
cmd.set('reflect', 0.0)
cmd.set('specular', 0.08)
cmd.set('surface_color', 'gray80')
cmd.set('transparency', 0.15)
```

Pocket highlight color: `lightblue`  
Peptide sticks color: `tv_orange`

Save figures to:
```
Final_Reports/figures/AS_v3/
```

---

## Output Structure

```
structures/
└── <TARGET_NAME>/
    ├── alphafold/          # AlphaFold3 predicted structures
    ├── boltzgen/           # BoltzGen L-peptide candidates
    ├── fragments/          # 5-mer sliding window output
    ├── smiles/             # D-peptide SMILES strings
    └── boltz_affinity/
        ├── yamls/          # One YAML per candidate
        └── output/         # Boltz2 prediction results
Final_Reports/
└── figures/
    └── AS_v3/              # All publication figures
```

---

## Validated Results

| Target | Pocket residues | Best D-peptide | IC₅₀ |
|--------|----------------|---------------|------|
| NDUFA9 | S10, S11, L61, G62 | d-LGRMG | 45.2 nM |
| NOTCH1 | Y85, Y88, D99, Q100, G101 | d-SSQCF | 574.2 nM |
| 2VSM | G299, E315, G316, Q340, T341 | d-GITLGGGS | 44.8 nM |

---

## Error Handling

**CUDA out of memory**: Reduce `--diffusion_samples` to 1 and `--devices` to 1.

**Invalid SMILES**: Validate with RDKit before submitting to Boltz2:
```python
from rdkit import Chem
assert Chem.MolFromSmiles(smiles) is not None, f"Invalid SMILES: {smiles}"
```

**Checkpoint not found**: Verify `~/.boltz/boltz2_aff.ckpt` exists. Do not use the default Boltz2 checkpoint for affinity prediction — it will silently produce wrong results.

**Wrong Python environment**: If `import pymol` fails, switch to `/home/ubuntu/miniconda/bin/python3`. If `import boltz` fails, switch to `/home/ubuntu/miniconda/envs/boltz/bin/python3`.
