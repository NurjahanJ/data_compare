# Explanation: `goal6_majorclass_classification.ipynb`

## What this notebook is for (Goal #6)
This notebook performs a **higher-level classification** of both Scopus publications and patents into **five major classes**:
1) Material
2) Computation
3) Experimentation
4) Application
5) Review/Book

Like Goals #1 and #2, classification is **Abstract-only** and uses a **zero-shot** approach (no training on a labeled dataset).

## Why “major classes” matter
The earlier goals classify into a detailed ferrofluids taxonomy (many fine-grained classes). Goal #6 instead provides a coarse-grained labeling that is useful for:
- comparing the research landscape across broad activity types,
- stratifying later analyses (Goal #7 topic modeling is built on these major classes),
- providing a simpler view that non-specialists can interpret more easily.

## Inputs (files read)
From this repository:
- `raw_data/MANI_KW_PAPERS_scopus.csv`
- `raw_data/MANI_KW_PATENTS_A_weds1969to2009.csv`
- `raw_data/MANI_KW_PATENTS_B_weds2010tonow.csv`
- `classification/MAJOR_CLASSES.md` (definitions of the 5 major classes)
- `docs/INSTRUCTIONS.md` and `docs/AI_CLASSIFICATON_IS421_2026S.md`

## Outputs (files written)
All Goal #6 outputs are written under `outputs/goal6/`.

Main deliverables:
- `goal6_scopus_major_classified.csv`
- `goal6_patents_major_classified.csv`

Audit/diagnostic artifacts:
- `goal6_scopus_dropped_rows.csv`
- `goal6_patents_dropped_rows.csv`
- `goal6_scopus_metrics_primary_major_class_counts.csv`
- `goal6_patents_metrics_primary_major_class_counts.csv`
- `goal6_scopus_metrics_ambiguous_top25.csv`
- `goal6_patents_metrics_ambiguous_top25.csv`
- `goal6_scopus_edge_cases.csv`
- `goal6_patents_edge_cases.csv`
- `goal6_run_config.json`

Also written at repo root:
- `goal6_requirements.json`

## Step-by-step: what the code does
### 0) Colab setup
The notebook mounts Google Drive and changes into the project directory.

### 1) Runtime setup (dependencies + configuration)
It pins/install dependencies in Colab and defines paths. Compared with Goals #1/#2, it additionally installs `datasets`, which is used for a more efficient batched pipeline iteration.

A key implementation detail: the notebook chooses `ROOT` robustly:
- prefers `/content/drive/MyDrive/mani` if it exists,
- otherwise uses the current working directory.

### 2) Display instructions (traceability)
The notebook reads and displays the instruction files from `docs/`.

### 3) Record requirements to JSON
It writes `goal6_requirements.json` so the expected schema and constraints are recorded alongside the run.

### 4) Load and clean datasets (Scopus and Patents)
The notebook loads:
- Scopus CSV (expects `Year`, `Title`, `Abstract`)
- Patents CSVs A and B (expects `Publication Year`, `Title`, `Abstract`)

Cleaning is intentionally minimal and designed for auditability:
- year is coerced to numeric (`Year_num`)
- abstract is stripped (`Abstract_clean`)
- rows are dropped only when year is invalid and/or abstract is empty

#### What `Abstract_clean` means
`Abstract_clean` is a helper column containing a minimally cleaned version of `Abstract`:
- missing values are filled with an empty string
- values are coerced to string
- leading/trailing whitespace is removed

Its purpose is pragmatic (consistent filtering + abstract-only input), not linguistic preprocessing. No stopword removal, lemmatization, or punctuation normalization is performed at this stage.

Crucially, dropped rows are not silently discarded:
- each dropped row is labeled with a `DropReason` (`invalid_year`, `empty_abstract`, or both)
- drop reports are written to CSV (`goal6_*_dropped_rows.csv`)

After cleaning, each dataset gets a required `Serial Number` column (1..N).

### 5) Parse the five major-class definitions
The notebook reads `classification/MAJOR_CLASSES.md` and extracts definitions under headings like `## MATERIAL`.

It then creates candidate labels for classification such as:
- `MATERIAL: <definition text>`
- `COMPUTATION: <definition text>`
- …

Each major class is also mapped to a numeric code 1–5.

### 6) Zero-shot classification (Abstract-only; 5 labels)
The notebook initializes a Hugging Face zero-shot classification pipeline using:
- model: `typeform/distilbert-base-uncased-mnli`

For each dataset (Scopus and patents), it:
- feeds **only** `Abstract_clean`
- classifies against the 5 candidate labels
- extracts top-3 predictions and their scores

The output columns include:
- `Primary Major Class`, `Primary Major Label`, `Primary Major Score`
- `Secondary ...`
- `Tertiary ...`

An ambiguity proxy is computed:
- `margin_12 = Primary Major Score - Secondary Major Score`

### 7) Export deliverables + diagnostics
For each dataset, the notebook:
- concatenates original columns + major-class columns
- sorts by year then class codes
- drops helper columns (`Year_num`, `Abstract_clean`)
- writes the CSV to `outputs/goal6/`

It also saves:
- class count tables
- ambiguous top-25 tables
- edge-case tables joined back to original metadata
- `goal6_run_config.json`

## How to describe this in a research paper (methods wording)
Suggested wording:
- Documents (Scopus and patents) were assigned to one of five major activity classes using an open-source MNLI **zero-shot classifier**.
- Candidate labels were constructed from project-provided class definitions.
- For each document we recorded the top-3 predicted major classes with associated model scores.
- Because no labeled reference set was available, model evaluation focused on diagnostic summaries and manual inspection of low-margin cases.

## Important constraints and limitations
- **Abstract-only constraint:** titles and metadata are intentionally excluded.
- **Interpretation of scores:** scores help rank alternatives but are not validated accuracy.
- **Small label set:** five categories make the model’s job easier, but nuance inside each category is not captured.

## Reproducing a run
In Colab, run cells in order:
1) Mount Drive + `%cd`
2) Runtime setup
3) Data loading + drop reports
4) Parse `MAJOR_CLASSES.md`
5) Run classification
6) Export

Goal #7 expects the two exported CSVs from this notebook.
