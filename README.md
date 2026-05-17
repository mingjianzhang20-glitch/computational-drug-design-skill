# computational-drug-design-skill

A Claude Code Skill for computational drug design using the Boltz2 IC₅₀ prediction pipeline. Designed for computational biology and drug design researchers who want to use AI-assisted structure prediction and affinity scoring to design high-affinity D-peptide inhibitors.

---

## What This Skill Does

This skill guides Claude Code through the full de novo D-peptide inhibitor design pipeline — from target protein input to ranked IC₅₀ predictions — using a pseudo-mirror strategy for chirality inversion.

It supports any target type compatible with Boltz2, including protein–peptide, protein–small molecule, protein–antibody, and nucleic acid complexes.

---

## Pipeline

    Target Protein
           |
           v
    AlphaFold3  ──  Binding pocket prediction
           |
           v
    BoltzGen  ──  De novo L-peptide candidate library generation
           |
           v
    5-mer Sliding Window Cut  ──  Fragment extraction from full sequences
           |
           v
    L → D Chirality Inversion  ──  Pseudo-mirror strategy
           |
           v
    SMILES Generation  ──  3D structure → 1D chemical encoding
           |
           v
    Boltz2 IC₅₀ Scoring  ──  Affinity ranking & IC₅₀ prediction

---

## Quick Start

1. Install all dependencies (see Dependency Installation Links below)
2. Clone this skill:
```bash
git clone https://github.com/mingjianzhang20-glitch/computational-drug-design-skill.git
```
3. Tell Claude Code to read the skill before starting:
```
Read computational-drug-design-skill/SKILL.md before starting
```
4. Give Claude Code your target and run:
```
Run the full D-peptide design pipeline for target protein <YOUR_TARGET>
```

---

## Repository Structure

```
computational-drug-design-skill/
├── README.md                          # This file
├── SKILL.md                           # Claude Code skill instructions (read this first)
├── CLAUDE.md                          # Auto-loaded by Claude Code
├── LICENSE                            # MIT
├── references/
│   ├── boltz2-scoring.md              # Boltz2 IC₅₀ prediction reference
│   ├── pymol-visualization.md         # PyMOL figure generation standard
│   ├── chirality-inversion.md         # L→D chirality inversion algorithm
│   └── figure-generation.md           # Publication figure standards
└── examples/
    ├── README.md                      # How to use the examples
    ├── ndufa9_boltz2_affinity.yaml    # NDUFA9 Boltz2 input example
    ├── notch1_boltz2_affinity.yaml    # NOTCH1 Boltz2 input example
    ├── 2vsm_boltz2_affinity.yaml      # 2VSM Boltz2 input example
    └── ndufa9_boltzgen_design.yaml    # BoltzGen design template
```

---

## Hardware Requirements

| Component | Requirement |
|-----------|-------------|
| GPU | NVIDIA RTX 4090 × 2 (recommended) |
| VRAM | ≥ 24 GB per GPU |
| CUDA | Compatible with RTX 4090 (Ada Lovelace architecture) |
| RAM | ≥ 64 GB system RAM recommended |

> ⚠️ Several dependencies (including Boltz2 and BoltzGen) have versions pinned to RTX 4090 compatibility. Running on older GPU architectures may require manual version adjustments and is not guaranteed to work.

---

## Software Dependencies

| Tool | Purpose | Notes |
|------|---------|-------|
| **Boltz2** | IC₅₀ affinity scoring | Requires `boltz2_aff.ckpt` checkpoint |
| **BoltzGen** | De novo peptide generation | Pinned version for RTX 4090 |
| **AlphaFold3** | Structure prediction & pocket identification | Requires weight application |
| **PyMOL** | Structure visualization & figure generation | Open-source or commercial |
| **Python ≥ 3.9** | Data processing & scripting | |
| **matplotlib** | Publication figure generation | |
| **python-docx** | Word document manipulation | |
| **RDKit** | SMILES generation & cheminformatics | |

### Python Environments

This skill uses two separate conda environments:

- `/data/miniconda/envs/boltz/bin/python3` — Boltz2, BoltzGen, data processing
- `/home/ubuntu/miniconda/bin/python3` — PyMOL visualization

---

## Dependency Installation Links

These tools must be installed manually before using this skill:

### Boltz2
- GitHub: https://github.com/jwohlwend/boltz
- Checkpoint: https://huggingface.co/boltz-community/boltz-2
- Install: `pip install boltz`
- Place affinity checkpoint at: `~/.boltz/boltz2_aff.ckpt`

### BoltzGen
- GitHub: https://github.com/HannesStark/boltzgen
- Install: `pip install boltzgen`
- Or from source: `git clone https://github.com/HannesStark/boltzgen && pip install -e .`
- ⚠️ Versions are pinned for RTX 4090. Check the releases page for a compatible version.

### AlphaFold3
- GitHub: https://github.com/google-deepmind/alphafold3
- ⚠️ Requires application for model weights: https://forms.gle/svvpY4u2jsHEwWYS6
- Follow the official installation guide carefully.

### PyMOL
- Open-source: https://github.com/schrodinger/pymol-open-source
- conda: `conda install -c conda-forge pymol-open-source`

### RDKit
- Docs: https://www.rdkit.org/docs/Install.html
- conda (recommended): `conda install -c conda-forge rdkit`

### Python packages
```bash
pip install matplotlib python-docx pandas
```

---

## Recommended Setup Order

1. Create conda environment with Python ≥ 3.9
2. Install RDKit via conda
3. Install PyMOL via conda
4. Install BoltzGen (check RTX 4090 compatible version)
5. Install Boltz2 and download `boltz2_aff.ckpt` checkpoint
6. Apply for AlphaFold3 model weights
7. Clone this skill and follow `SKILL.md`

---

## Validated Targets

| Target | Best D-peptide | IC₅₀ |
|--------|---------------|------|
| NDUFA9 | d-LGRMG | 45.2 nM |
| NOTCH1 | d-SSQCF | 574.2 nM |
| 2VSM | d-GITLGGGS | 44.8 nM |

---

## Reference

> A Pseudo-Mirror Strategy Enables De Novo Computational Design of High-Affinity D-Peptide Inhibitors

---

## License

MIT License
