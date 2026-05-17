# computational-drug-design-skill

A Claude Code Skill for computational drug design using the Boltz2 IC₅₀ prediction pipeline. Designed for computational biology and drug design researchers who want to use AI-assisted structure prediction and affinity scoring to design high-affinity D-peptide inhibitors.

---

## What This Skill Does

This skill guides Claude Code through the full de novo D-peptide inhibitor design pipeline — from target protein input to ranked IC₅₀ predictions — using a pseudo-mirror strategy for chirality inversion.

It supports any target type compatible with Boltz2, including protein–peptide, protein–small molecule, protein–antibody, and nucleic acid complexes.

---

## Pipeline

**1. Target Input**
Accepts any target protein sequence or PDB structure. A binding-region sequence is specified or auto-identified.

**2. L-Peptide Generation (BoltzGen)**
Generates candidate L-peptide sequences targeting the binding region using BoltzGen's diffusion-based protein design model.

**3. Chirality Inversion (RDKit)**
Converts each L-peptide to its D-peptide mirror image by inverting all chiral centers (CW ↔ CCW) using RDKit. The resulting D-SMILES carry explicit stereochemistry markers (`[C@@H]` for D-form residues).

**4. Affinity Scoring (Boltz2)**
Submits each D-peptide SMILES as a ligand to Boltz2 with `--affinity_checkpoint`. Outputs:
- `affinity_pred_value` → IC₅₀ (nM) = 10^value × 1000
- Structural metrics (iptm, ptm, pLDDT)

**5. Ranking**
Peptides ranked by predicted IC₅₀ (ascending = best binding).

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Chirality inversion** | RDKit CW↔CCW flip of all α-carbon stereocenters |
| **Explicit stereochemistry** | All D-peptide SMILES carry `[C@@H]` markers validated by RDKit |
| **Affinity checkpoint** | Direct IC₅₀ prediction via Boltz2 `--affinity_checkpoint` |
| **Structural validation** | iPTM/ptm/pLDDT from Boltz2 confidence JSON |
| **Multi-target** | Works with any Boltz2-compatible target (protein–peptide, small molecule, etc.) |

---

## Quick Start

1. **Install dependencies** (Python ≥ 3.10):
   ```
   pip install rdkit-pypi biopython pandas openpyxl pyyaml
   ```

2. **Install Boltz2** (see [Boltz2 installation guide](https://github.com/jwohlwend/boltz)):
   ```bash
   conda create -n boltz -c conda-forge python=3.10
   conda activate boltz
   pip install boltz
   boltz setup --model boltz2
   ```

3. **Add skill to Claude Code** (see [skill installation docs](https://docs.anthropic.com/en/docs/claude-code/skills))

4. **Run the pipeline:**
   ```bash
   python scripts/batch_target_boltz.py --target YOUR_TARGET_SEQ --output output/
   ```
   Or use the Claude Code skill directly:
   ```
   /design-d-peptide --target YOUR_TARGET_SEQ
   ```

---

## Repository Structure

```
computational-drug-design-skill/
├── README.md
├── LICENSE
├── .gitignore
└── scripts/
    ├── batch_target_boltz.py       # Batch Boltz2 affinity scoring
    ├── boltz_affinity_pipeline/
    │   ├── run_affinity_pipeline.py  # Affinity prediction pipeline
    │   └── parse_affinity.py          # IC₅₀ extraction
    └── rdkit_tools/
        └── invert_chirality.py       # L→D chirality inversion
```

---

## Chirality Inversion Algorithm

```python
from rdkit import Chem
from rdkit.Chem import rdchem

def invert_chirality(sequence):
    """Convert L-peptide to D-peptide SMILES."""
    mol = Chem.MolFromSequence(sequence.upper())
    Chem.AssignStereochemistry(mol, cleanIt=True, force=True)
    for atom in mol.GetAtoms():
        ct = atom.GetChiralTag()
        if ct == rdchem.ChiralType.CHI_TETRAHEDRAL_CW:
            atom.SetChiralTag(rdchem.ChiralType.CHI_TETRAHEDRAL_CCW)
        elif ct == rdchem.ChiralType.CHI_TETRAHEDRAL_CCW:
            atom.SetChiralTag(rdchem.ChiralType.CHI_TETRAHEDRAL_CW)
    return Chem.MolToSmiles(mol, isomericSmiles=True)
```

---

## Contributing

Contributions are welcome. Please open an issue or submit a pull request.

---

## License

MIT