# L → D Chirality Inversion

Reference for the pseudo-mirror strategy used to convert L-peptide candidates to D-peptide inhibitors.

---

## Concept

D-peptides use the mirror-image form of each amino acid. The pseudo-mirror strategy inverts all chiral centers in the SMILES string, producing a D-peptide enantiomer of the original L-peptide candidate.

Key advantages of D-peptides:
- Resistant to proteolytic degradation
- Longer in vivo half-life
- Identical binding geometry when target is mirrored

---

## SMILES Chirality Inversion

```python
def invert_chirality(smiles: str) -> str:
    """
    Invert all chiral centers in a SMILES string.
    @ becomes @@ and @@ becomes @
    Converts an L-peptide to its D-enantiomer.
    """
    result = []
    i = 0
    while i < len(smiles):
        if smiles[i] == '@':
            if i + 1 < len(smiles) and smiles[i + 1] == '@':
                result.append('@')
                i += 2
            else:
                result.append('@@')
                i += 1
        else:
            result.append(smiles[i])
            i += 1
    return ''.join(result)
```

### Example

```python
l_smiles = "CC(C)[C@@H](NC(=O)[C@H](N)CO)C(=O)O"
d_smiles  = invert_chirality(l_smiles)
# -> "CC(C)[C@H](NC(=O)[C@@H](N)CO)C(=O)O"
```

---

## Batch Inversion Script

```python
from rdkit import Chem
import pandas as pd

def invert_chirality(smiles):
    result = []
    i = 0
    while i < len(smiles):
        if smiles[i] == "@":
            if i + 1 < len(smiles) and smiles[i+1] == "@":
                result.append("@"); i += 2
            else:
                result.append("@@"); i += 1
        else:
            result.append(smiles[i]); i += 1
    return "".join(result)

df = pd.read_csv("l_peptide_smiles.csv")
df["d_smiles"] = df["l_smiles"].apply(invert_chirality)
df["valid"] = df["d_smiles"].apply(lambda s: Chem.MolFromSmiles(s) is not None)
df[df["valid"]].to_csv("d_peptide_smiles.csv", index=False)
```

---

## 5-mer Sliding Window

```python
def sliding_window_5mer(sequence: str) -> list:
    return [sequence[i:i+5] for i in range(len(sequence) - 4)]

# Example
fragments = sliding_window_5mer("LGRMGKPAT")
# -> ["LGRMG", "GRMGK", "RMGKP", "MGKPA", "GKPAT"]
```

---

## SMILES from PDB Structure

```python
from rdkit import Chem

def pdb_to_smiles(pdb_path: str) -> str:
    mol = Chem.MolFromPDBFile(pdb_path, removeHs=False, sanitize=True)
    if mol is None:
        raise ValueError(f"Could not parse PDB: {pdb_path}")
    return Chem.MolToSmiles(mol)
```

---

## Validation

```python
from rdkit import Chem

def validate_and_canonicalize(smiles: str) -> str:
    mol = Chem.MolFromSmiles(smiles)
    if mol is None:
        raise ValueError(f"Invalid SMILES: {smiles}")
    return Chem.MolToSmiles(mol)
```
