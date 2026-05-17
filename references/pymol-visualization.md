# PyMOL Visualization

Standard PyMOL workflow for generating publication-quality figures.

---

## Environment

```bash
/home/ubuntu/miniconda/bin/python3
```

Output directory: `Final_Reports/figures/AS_v3/`

---

## Standard Rendering Settings

```python
cmd.set('ray_shadows', 0)
cmd.set('ambient', 0.65)
cmd.set('direct', 0.55)
cmd.set('reflect', 0.0)
cmd.set('specular', 0.08)
cmd.set('surface_color', 'gray80')
cmd.set('transparency', 0.15)
```

---

## Color Conventions

| Element | Color |
|---------|-------|
| Protein surface | `gray80` |
| Binding pocket | `lightblue` |
| L-peptide sticks | `cyan` |
| D-peptide sticks | `tv_orange` |
| NDUFA9 label | `#2166AC` |
| NOTCH1 label | `#D6604D` |
| 2VSM label | `#4DAC26` |

---

## Pocket Residues

| Target | Pocket residues |
|--------|----------------|
| NDUFA9 | S10, S11, L61, G62 |
| NOTCH1 | Y85, Y88, D99, Q100, G101 |
| 2VSM | G299, E315, G316, Q340, T341 |

---

## Standard Workflow

```python
from pymol import cmd

# Load and show surface
cmd.load('structure.pdb', 'protein')
cmd.hide('everything', 'protein')
cmd.show('surface', 'protein')
cmd.color('gray80', 'protein')
cmd.set('transparency', 0.15, 'protein')

# Highlight pocket
cmd.select('pocket', 'protein and resi 10+11+61+62')
cmd.show('surface', 'pocket')
cmd.color('lightblue', 'pocket')

# Load peptide
cmd.load('peptide.pdb', 'peptide')
cmd.hide('everything', 'peptide')
cmd.show('sticks', 'peptide')
cmd.color('tv_orange', 'peptide')  # D-peptide; use cyan for L-peptide

# Render
cmd.set('ray_shadows', 0)
cmd.set('ambient', 0.65)
cmd.set('direct', 0.55)
cmd.set('reflect', 0.0)
cmd.set('specular', 0.08)
cmd.ray(2400, 1800)
cmd.png('output.png', dpi=300)
```

---

## AS Style Parameters (matplotlib assembly)

```python
CARD_EDGE = '#C9A77B'
COLORS = {'NDUFA9': '#2166AC', 'NOTCH1': '#D6604D', '2VSM': '#4DAC26'}
FILLS  = {'NDUFA9': '#D1E5F0', 'NOTCH1': '#FDDBC7', '2VSM': '#D9F0D3'}
```

Assembly scripts:
- Fig2: `gen_fig2_final_v2.py`
- Fig6: `gen_fig6_final_v2.py`
