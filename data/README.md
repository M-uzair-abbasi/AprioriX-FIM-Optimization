# Datasets

This folder is where the FIMI benchmark datasets are stored after download.

## Auto-download

The datasets are downloaded automatically by `notebooks/notebook1_data_eda.ipynb`. No manual action needed.

## Manual download

If automatic download fails, you can manually download from the FIMI Repository:

| Dataset | URL | Size |
|---------|-----|------|
| Chess | http://fimi.uantwerpen.be/data/chess.dat | ~340 KB |
| Connect | http://fimi.uantwerpen.be/data/connect.dat | ~9 MB |
| Accidents | http://fimi.uantwerpen.be/data/accidents.dat | ~33 MB |

Place the `.dat` files directly in this `data/` folder.

## Dataset Format

Each `.dat` file is a plain-text transactional dataset where:
- Each line represents one transaction
- Items are space-separated integers
- Example: `1 5 12 33 47 89`

## Why not committed to git?

Raw data files are excluded via `.gitignore` because:
1. They're large (>40 MB total)
2. They're hosted at the canonical FIMI URL
3. Re-downloading guarantees consistency

## Citation

If you use these datasets, cite:
> Frequent Itemset Mining Dataset Repository, http://fimi.uantwerpen.be/data/
