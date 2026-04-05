# Sprint (Active): Goal #1 – Scopus Abstract Classification

Date started: 2026-03-01

## Objective
Produce a Google Colab notebook to classify Scopus publications into the predefined ferrofluids class taxonomy using **ONLY the Abstract**.

## Constraints (from docs)
- Run in Google Colab (free tier)
- Zero cost
- Use open-source Hugging Face models (no paid APIs)
- Classification uses **Abstract only** (not Title/Keywords/metadata)
- Preserve all original columns; add classification columns
- Sort output by Year, Primary Class, Secondary Class, Tertiary Class

## Tasks
- [x] Read project instructions in `docs/`
- [x] Load and parse class taxonomy from `classification/FEROFLUIDS_CLASS_DEFINITION.md`
- [x] Build Colab-ready notebook for Goal #1 (Scopus)
- [ ] Run notebook end-to-end in Colab and export final CSV

## Outputs
- Notebook: `goal1_scopus_classification.ipynb`
- Export CSV (from notebook): `outputs/goal1_scopus_classified.csv`

## Notes
- The Scopus CSV appears to include one non-data row (instructional text) immediately after the header; notebook drops rows with non-numeric Year / missing Abstract.
