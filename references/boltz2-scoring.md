# Boltz2 IC₅₀ Scoring

Detailed reference for running Boltz2 affinity predictions in the D-peptide design pipeline.

---

## Input YAML Format

Each candidate requires one YAML file. The format uses `version: 1` with a protein chain (id: A) and a peptide ligand as SMILES (id: B).

### NDUFA9 example

```yaml
version: 1
sequences:
  - protein:
      id: A
      sequence: DPVITNPDAKLSVQDTQVK
  - ligand:
      id: B
      smiles: CC(C)[C@@H](NC(=O)[C@H](NC(=O)[C@H](N)CCCCN)[C@H](C)O)C(=O)N[C@H](CC(N)=O)C(=O)N[C@H](Cc1ccc(O)cc1)C(=O)O
properties:
  - affinity:
      binder: B
```

### NOTCH1 example

```yaml
version: 1
sequences:
  - protein:
      id: A
      sequence: CSPGAPQGPPFPGAPGPKQDYRCGGSFCMGPGCGKVGTCVDIWNSGPCLAVISGAAPDGFLSELGAGVPCRAVCSSVGCCRWGLPCSSTVCPCLVGAGFGFHVQWLCRTCVNTRDLRQWEERLGRPCPRVRDVDLLDRPWGPCLRWGPAPWRLSRDGPCGWDHWSCCASQLVGCCPVAGWRCVDIWDPCLRAGVCCRLVGANGFDRCWTCGAPGWGTCVWDWADCPRAADGPCRAVDWADPCLARGVCSPDGFCADLVCQDLWGPQGANGFVGCCDPGGTCVAGWDRDLSGPCADPWTCADGTCGTCVWDGSDWTPCHV
  - ligand:
      id: B
      smiles: CC[C@@H](C)[C@@H](NC(=O)[C@H](N)CO)C(=O)N[C@@H](C(=O)N[C@H](Cc1ccc(O)cc1)C(=O)N[C@H](CC(C)C)C(=O)O)C(C)C
properties:
  - affinity:
      binder: B
```

---

## Running Predictions

### Environment setup

```bash
source /home/ubuntu/miniconda/bin/activate boltz
```

### Single candidate

```bash
boltz predict candidate_00000.yaml \
    --checkpoint ~/.boltz/boltz2_aff.ckpt \
    --out_dir ./affinity_output \
    --devices 2 \
    --accelerator gpu \
    --diffusion_samples 1 \
    --sampling_steps 200
```

### Batch scoring

```bash
source /home/ubuntu/miniconda/bin/activate boltz

for yaml in yamls/*.yaml; do
    boltz predict "$yaml" \
        --checkpoint ~/.boltz/boltz2_aff.ckpt \
        --out_dir ./affinity_output \
        --devices 2 \
        --accelerator gpu \
        --diffusion_samples 1 \
        --sampling_steps 200 \
        --override
done
```

### Key flags

| Flag | Default | Description |
|------|---------|-------------|
| `--checkpoint` | — | Always use `~/.boltz/boltz2_aff.ckpt` |
| `--out_dir` | `./predictions` | Output directory |
| `--devices` | 1 | Number of GPUs; use 2 for dual RTX 4090 |
| `--accelerator` | gpu | Always use gpu |
| `--diffusion_samples` | 1 | Increase for diversity |
| `--sampling_steps` | 200 | Higher = more accurate, slower |
| `--override` | false | Overwrite existing results |

---

## Parsing Results

```python
import json, glob
import pandas as pd

results = []
for f in glob.glob('affinity_output/**/affinity*.json', recursive=True):
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

## Validated Results

| Target | Best D-peptide | IC₅₀ |
|--------|---------------|------|
| NDUFA9 | d-LGRMG | 45.2 nM |
| NOTCH1 | d-SSQCF | 574.2 nM |
| 2VSM | d-GITLGGGS | 44.8 nM |

---

## Common Errors

**Wrong checkpoint**: Always use `~/.boltz/boltz2_aff.ckpt`. The default checkpoint produces wrong results silently.

**CUDA out of memory**: Reduce `--diffusion_samples` to 1 and `--devices` to 1.

**Invalid SMILES**: Validate before submitting:
```python
from rdkit import Chem
assert Chem.MolFromSmiles(smiles) is not None, f"Invalid SMILES: {smiles}"
```
