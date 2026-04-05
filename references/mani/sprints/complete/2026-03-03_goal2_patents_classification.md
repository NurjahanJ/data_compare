# Sprint (Active): Goal #2 – Patent Abstract Classification

Date started: 2026-03-03

## Objective
Produce a Google Colab notebook to classify the US patent dataset into the predefined ferrofluids class taxonomy using **ONLY the Abstract**.

## Constraints (from docs)
- Run in Google Colab (free tier)
- Zero cost
- Use open-source Hugging Face models (no paid APIs)
- Classification uses **Abstract only** (not Title/CPC/other metadata)
- Preserve all original columns; add classification columns
- Sort output by Year, Primary Class, Secondary Class, Tertiary Class

## Inputs
- `raw_data/MANI_KW_PATENTS_A_weds1969to2009.csv`
- `raw_data/MANI_KW_PATENTS_B_weds2010tonow.csv`
- `classification/FEROFLUIDS_CLASS_DEFINITION.md`
- `docs/INSTRUCTIONS.md`
- `docs/AI_CLASSIFICATON_IS421_2026S.md`

## Tasks
- [x] Read project instructions in `docs/`
- [x] Confirm patent CSV schemas + required columns
- [x] Build Colab-ready notebook for Goal #2 (patents)
- [ ] Run notebook end-to-end in Colab and export final CSV

## Outputs
- Notebook: `goal2_patents_classification.ipynb`
- Export CSV (from notebook): `outputs/goal2_patents_classified.csv`
- Run config snapshot (from notebook): `outputs/goal2_run_config.json`
- Requirements JSON (from notebook): `goal2_requirements.json`

## Notes
- Patent CSVs use `Publication Year` (normalized to `Year` in-notebook for sorting and consistency).
