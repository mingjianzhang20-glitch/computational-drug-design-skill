# Figure Generation

Standards for generating publication-quality figures.

---

## Output Directory
Final_Reports/figures/AS_v3/

---

## AS Style Parameters

```python
CARD_EDGE = "#C9A77B"
COLORS = {"NDUFA9": "#2166AC", "NOTCH1": "#D6604D", "2VSM": "#4DAC26"}
FILLS  = {"NDUFA9": "#D1E5F0", "NOTCH1": "#FDDBC7", "2VSM": "#D9F0D3"}
```

---

## Figure Inventory

| Figure | File | Description |
|--------|------|-------------|
| Fig1 | `Fig1_Pipeline.png` | Pipeline flowchart |
| Fig2 | `Fig2_Targets_Introduction.png` | 3x2 grid: overview + pocket |
| Fig3 | `Fig3_DPeptide_distributions.png` | Score distributions |
| Fig4 | `Fig4_LvsD_comparison.png` | L vs D comparison |
| Fig5 | `Fig5_Benchmark_SAR.png` | Benchmark and SAR |
| Fig6 | `Fig6_LvsD_overlay.png` | L/D structural overlay |

---

## Fig2 Layout
Row 1: NDUFA9 overview | NDUFA9 pocket (S10, S11, L61, G62)
Row 2: NOTCH1 overview | NOTCH1 pocket (Y85, Y88, D99, Q100, G101)
Row 3: 2VSM overview   | 2VSM pocket   (G299, E315, G316, Q340, T341)

Scripts: `gen_fig2_final_v2.py`, `add_fig2_pocket_labels.py`

---

## Fig6 Layout

- Left: full protein overview with dashed box on pocket, arrow pointing right
- Right: L-peptide (cyan) + D-peptide (tv_orange) overlay in pocket

Scripts: `gen_fig6_final_v2.py`, `add_fig6_labels_v2.py`

---

## Saving Figures

```python
fig.savefig("FigX_Name.png", dpi=300, bbox_inches="tight", facecolor=fig.get_facecolor())
fig.savefig("FigX_Name.pdf", bbox_inches="tight", facecolor=fig.get_facecolor())
```

---

## Inserting Figures into Word Document

```python
import zipfile, shutil, os, tempfile

docx_path  = "Final_Reports/DPeptide_Paper_v26_formatted.docx"
new_image  = "Final_Reports/figures/AS_v3/Fig1_Pipeline.png"
image_name = "image1.png"  # filename inside word/media/

with tempfile.TemporaryDirectory() as tmp:
    with zipfile.ZipFile(docx_path, "r") as z:
        z.extractall(tmp)
    shutil.copy(new_image, os.path.join(tmp, "word", "media", image_name))
    out_path = docx_path.replace(".docx", "_v2.docx")
    with zipfile.ZipFile(out_path, "w", zipfile.ZIP_DEFLATED) as z:
        for root, dirs, files in os.walk(tmp):
            for file in files:
                fp = os.path.join(root, file)
                z.write(fp, os.path.relpath(fp, tmp))

print(f"Saved: {out_path}")
```

---

## Image Mapping

| Figure | rId | Filename in word/media/ |
|--------|-----|------------------------|
| Fig1 | rId9 | image1.png |

To find all mappings:
```bash
unzip -p document.docx word/_rels/document.xml.rels | grep -o "rId[0-9]*.*image[^\"]*"
```
