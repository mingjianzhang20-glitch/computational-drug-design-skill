# Claude Code Skill: computational-drug-design-skill

This repository is a Claude Code Skill for de novo D-peptide inhibitor design.

## How to Use

Before starting any task in this pipeline, read the full skill instructions:
```
Read SKILL.md before proceeding
```

## Skill File Locations

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — read this first |
| `references/boltz2-scoring.md` | Boltz2 IC₅₀ prediction commands |
| `references/pymol-visualization.md` | PyMOL figure generation |
| `references/chirality-inversion.md` | L→D chirality inversion code |
| `references/figure-generation.md` | Publication figure standards |
| `examples/` | Ready-to-run input YAML files |

## Quick Task Reference

| User says | What to do |
|-----------|-----------|
| "Run the full pipeline for target X" | Follow all 7 steps in SKILL.md |
| "Score IC₅₀ for these SMILES" | Read references/boltz2-scoring.md |
| "Invert chirality of this peptide" | Read references/chirality-inversion.md |
| "Visualize the binding pocket" | Read references/pymol-visualization.md |
| "Generate a figure" | Read references/figure-generation.md |

## Environment

- Boltz2 / data processing: `/data/miniconda/envs/boltz/bin/python3`
- PyMOL: `/home/ubuntu/miniconda/bin/python3`
- Boltz2 checkpoint: `~/.boltz/boltz2_aff.ckpt`
- Activate boltz env: `source /home/ubuntu/miniconda/bin/activate boltz`
