# Sprint (Complete): Goal #6 – Major-Class Classification (Scopus + Patents, Abstract-only)

Date started: 2026-03-09

## Objective
Produce a Google Colab notebook to classify **Scopus publications** and **US patents** into the **5 major classes** using **ONLY the Abstract**.

## Constraints (from docs)
- Run in Google Colab (free tier)
- Zero cost
- Use open-source Hugging Face models (no paid APIs)
- Classification uses **Abstract only** (not Title / Keywords / CPC / other metadata)
- Preserve all original columns; add classification columns
- Sort outputs by Year, Primary Major Class, Secondary Major Class, Tertiary Major Class

## Inputs
- `raw_data/MANI_KW_PAPERS_scopus.csv`
- `raw_data/MANI_KW_PATENTS_A_weds1969to2009.csv`
- `raw_data/MANI_KW_PATENTS_B_weds2010tonow.csv`
- `classification/MAJOR_CLASSES.md`
- `docs/INSTRUCTIONS.md`
- `docs/AI_CLASSIFICATON_IS421_2026S.md`

## Tasks
- [x] Read project instructions in `docs/` (rendered in-notebook for traceability)
- [x] Parse the 5 major classes from `classification/MAJOR_CLASSES.md`
- [x] Build Colab-ready notebook for Goal #6 (Scopus + patents)
- [x] Add minimal ingest cleaning + explicit drop reporting (per-row `DropReason`)
- [ ] Run notebook end-to-end in Colab and export final CSVs

## Outputs
- Notebook: `goal6_majorclass_classification.ipynb`
- Requirements JSON (from notebook): `goal6_requirements.json`
- Export CSVs (from notebook):
  - `outputs/goal6/goal6_scopus_major_classified.csv`
  - `outputs/goal6/goal6_patents_major_classified.csv`
- Drop reports (from notebook, for auditability):
  - `outputs/goal6/goal6_scopus_dropped_rows.csv`
  - `outputs/goal6/goal6_patents_dropped_rows.csv`

## Notes
- Patents use `Publication Year` in the raw CSVs and are normalized to `Year` in-notebook for sorting and consistency.
- Rows are dropped only for (a) missing/non-numeric year and/or (b) missing/empty abstract; the notebook writes a drop-report CSV with a per-row `DropReason` (`invalid_year`, `empty_abstract`, `invalid_year+empty_abstract`).
- Zero-shot candidate labels are derived from the major class headings; outputs store the major-class name (with `REVIEW / BOOK` normalized to `REVIEW/BOOK` for display consistency).
