# Troubleshooting

Common errors and solutions for the D-peptide design pipeline.

---

## Boltz2 Errors

### Wrong checkpoint — silent wrong results

**Symptom**: Boltz2 runs without error but IC₅₀ values are unrealistically high or all identical.
**Cause**: Using the default Boltz2 checkpoint instead of the affinity checkpoint.
**Fix**:
```bash
# Always use the affinity checkpoint explicitly
boltz predict input.yaml --checkpoint ~/.boltz/boltz2_aff.ckpt
```
Verify checkpoint exists:
```bash
ls -lh ~/.boltz/boltz2_aff.ckpt
```

---

### CUDA out of memory

**Symptom**: `RuntimeError: CUDA out of memory`
**Fix**: Reduce load:
```bash
boltz predict input.yaml \
    --checkpoint ~/.boltz/boltz2_aff.ckpt \
    --devices 1 \
    --diffusion_samples 1 \
    --sampling_steps 100
```

---

### Boltz2 not found

**Symptom**: `command not found: boltz`
**Fix**: Activate the correct conda environment first:
```bash
source /home/ubuntu/miniconda/bin/activate boltz
which boltz  # should return /home/ubuntu/miniconda/envs/boltz/bin/boltz
```

---

### Affinity JSON missing or empty

**Symptom**: No `affinity*.json` files in output directory.
**Cause**: Input YAML missing `properties.affinity` block.
**Fix**: Ensure your YAML includes:
```yaml
properties:
  - affinity:
      binder: B
```

---

## BoltzGen Errors

### No designs in output folder

**Symptom**: BoltzGen runs without error but output directory is empty.
**Cause**: Common with `--steps inverse_folding` only, or YAML misconfiguration.
**Fix**: Run the full pipeline first:
```bash
boltzgen run design.yaml \
    --output workbench/target \
    --protocol peptide-anything \
    --num_designs 100
```

---

### File path error in design YAML

**Symptom**: `FileNotFoundError` for `.cif` file.
**Cause**: `.cif` paths in BoltzGen YAMLs are relative to the YAML file location, not the working directory.
**Fix**: Place the `.cif` file in the same directory as the YAML, or use an absolute path.

---

### Run interrupted — restart without losing progress

```bash
boltzgen run design.yaml \
    --output workbench/target \
    --protocol peptide-anything \
    --num_designs 1000 \
    --reuse   # resumes from last checkpoint
```

---

## RDKit / SMILES Errors

### Invalid SMILES after chirality inversion

**Symptom**: `Chem.MolFromSmiles()` returns `None`.
**Fix**: Validate before and after inversion:
```python
from rdkit import Chem

def validate_smiles(smiles):
    mol = Chem.MolFromSmiles(smiles)
    if mol is None:
        raise ValueError(f"Invalid SMILES: {smiles}")
    return Chem.MolToSmiles(mol)  # canonical form

l_smiles = "CC(C)[C@@H](N)C(=O)O"
d_smiles = invert_chirality(l_smiles)
d_smiles = validate_smiles(d_smiles)  # raises if invalid
```

---

### PDB to SMILES conversion fails

**Symptom**: `Chem.MolFromPDBFile()` returns `None`.
**Fix**: Try with sanitize=False first, then fix manually:
```python
mol = Chem.MolFromPDBFile("structure.pdb", removeHs=False, sanitize=False)
if mol:
    Chem.SanitizeMol(mol)
```

---

## PyMOL Errors

### Wrong Python environment

**Symptom**: `ModuleNotFoundError: No module named pymol`
**Fix**: Always use the PyMOL-specific Python:
```bash
/home/ubuntu/miniconda/bin/python3 your_pymol_script.py
# NOT /data/miniconda/envs/boltz/bin/python3
```

---

### Ray tracing very slow

**Fix**: Reduce render resolution for testing, use full resolution for final output:
```python
# Testing (fast)
cmd.ray(800, 600)

# Final publication (slow but high quality)
cmd.ray(2400, 1800)
cmd.png("output.png", dpi=300)
```

---

## Word Document Errors

### Replaced image does not show correct figure

**Symptom**: After replacing `image1.png` in the docx, the wrong figure appears.
**Cause**: The image filename in `word/media/` may not be `image1.png`.
**Fix**: Check the actual filename mapping first:
```bash
unzip -p document.docx word/_rels/document.xml.rels | tr ">" "\n" | grep -i image
```

---

### Docx corrupted after replacement

**Symptom**: Word cannot open the file after replacement.
**Cause**: Files were re-zipped incorrectly (e.g. double-nested directory).
**Fix**: Use this exact rezip pattern:
```python
import zipfile, os

with zipfile.ZipFile("output.docx", "w", zipfile.ZIP_DEFLATED) as zout:
    for root, dirs, files in os.walk(tmp_dir):
        for file in files:
            filepath = os.path.join(root, file)
            arcname = os.path.relpath(filepath, tmp_dir)
            zout.write(filepath, arcname)
```
Make sure `arcname` does NOT include the temp directory name itself.

---

## General Tips

- Always test with 1-5 candidates before running a full batch of 500
- Keep L-peptide and D-peptide SMILES in separate CSV columns for traceability
- Use `--override` flag in Boltz2 only when you intentionally want to overwrite existing results
- If a step fails midway through a large batch, use the output directory to identify which YAMLs already completed and skip them
