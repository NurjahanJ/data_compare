# Explanation: `goal2_patents_classification.ipynb`

## What this notebook is for (Goal #2)
This notebook classifies US patent records into a predefined **ferrofluids taxonomy** using **only the Abstract text**.

Compared to Goal #1 (Scopus), the core classification method is the same, but the data ingestion differs because patents arrive as **two CSV exports** that must be merged.

## Audience-friendly summary of the method
The notebook uses **zero-shot text classification** (no training on your dataset). A pre-trained language model ranks which taxonomy labels best match each abstract.

You can think of it as an automated “best-fitting category” system:
- input: a patent abstract
- options: a fixed list of taxonomy labels
- output: the top 3 best matches + a traceable reasoning string

## Inputs (files read)
From this repository:
- `raw_data/MANI_KW_PATENTS_A_weds1969to2009.csv`
- `raw_data/MANI_KW_PATENTS_B_weds2010tonow.csv`
- `classification/FEROFLUIDS_CLASS_DEFINITION.md`
- `docs/INSTRUCTIONS.md` and `docs/AI_CLASSIFICATON_IS421_2026S.md`

## Outputs (files written)
Main deliverable:
- `outputs/goal2_patents_classified.csv`

Diagnostics and reproducibility artifacts:
- `goal2_requirements.json` (written at repo root)
- `outputs/goal2_metrics_primary_class_counts.csv`
- `outputs/goal2_metrics_ambiguous_top25.csv`
- `outputs/goal2_edge_cases.csv`
- `outputs/goal2_run_config.json`

Optional refinement output:
- `outputs/goal2_bart_refinement_spotcheck.csv`

## Step-by-step: what the code does
### 0) Colab setup
The notebook is designed for Google Colab:
- mounts Google Drive
- changes directory to the project folder

### 1) Runtime setup (dependencies + configuration)
It installs/pins dependencies similarly to Goal #1 (to avoid breaking Colab), then defines `CONFIG` including:
- both patent CSV paths
- taxonomy path
- output CSV path
- model name (`typeform/distilbert-base-uncased-mnli`)
- batching settings

A fixed seed (`SEED = 421`) is set for reproducibility.

### 2) Display instructions (traceability)
The notebook renders the assignment constraints from `docs/` so the “Abstract only” rule stays visible.

### 3) Record requirements to JSON
It writes `goal2_requirements.json` describing the required output columns, sorting, and constraints.

### 4) Merge patent CSVs and clean minimal issues
The ingestion step:
- loads patent export A and B
- concatenates them into a single dataset
- checks required columns exist: `Publication Year`, `Title`, `Abstract`
- normalizes year into a shared `Year` column
- drops rows where year is non-numeric or abstract is empty
- creates a required `Serial Number` column (1..N)

#### What `Abstract_clean` means
`Abstract_clean` is a minimally cleaned version of the raw patent `Abstract` column. It is created to make sure:
- the model sees **only** the abstract text (abstract-only constraint)
- blank/missing abstracts can be filtered out consistently

In the notebook it is built by filling missing values with an empty string, converting to string, and trimming surrounding whitespace.

It is **not** a full text-normalization pipeline; it does not remove stopwords, punctuation, or apply lemmatization.

`Abstract_clean` is used for the classifier input, then dropped from the final export as a helper-only column.

### 5) Parse the ferrofluids taxonomy
The notebook reads `classification/FEROFLUIDS_CLASS_DEFINITION.md` and parses the class list.
It builds candidate label strings (code + class text) for the classifier.

### 6) Zero-shot classification (Abstract-only)
Core logic:
- initialize a Hugging Face zero-shot classification pipeline
- pass only the `Abstract_clean` text to the model
- classify in batches
- extract top-3 labels per abstract

The notebook then outputs:
- numeric class codes and descriptions for Primary/Secondary/Tertiary
- a `Reasoning` string listing top labels with scores

### 7) Diagnostics (not “accuracy”)
No gold-standard labels are provided, so the notebook computes review-oriented diagnostics:
- primary class counts
- ambiguity proxy `margin_12 = Top1_score - Top2_score`

The “most ambiguous” rows are saved for spot checking.

### 8) Optional refinement pass (BART spot-check)
This step optionally reruns the **most ambiguous** patent rows using a heavier MNLI model, then compares those results against the baseline model.

What it does:
- **Baseline model (full dataset):** `typeform/distilbert-base-uncased-mnli`
- **Refinement model (subset only):** `facebook/bart-large-mnli`

How rows are selected:
- The notebook computes `margin_12 = Top1_score - Top2_score` from the baseline run.
- It selects the lowest-margin rows (default `REFINE_N = 200`), i.e., cases where the baseline model’s top two labels were nearly tied.

What BART does in this context:
- BART runs its **own** abstract-only zero-shot classification over the same candidate labels.
- It evaluates the abstract against the **full candidate label set** (the entire taxonomy label list in `candidate_labels`), not just the baseline model’s top-3.
- The notebook then keeps BART’s **top-3** labels/scores for the comparison CSV.
- The notebook writes a **baseline vs BART** comparison table to `outputs/goal2_bart_refinement_spotcheck.csv` so you can review disagreements.

Whether BART changes the final output:
- If `APPLY_REFINEMENT = False`, results are **not applied** (compare-only).
- If `APPLY_REFINEMENT = True`, BART’s predictions **overwrite the baseline predictions only for the selected rows**.

### 9) Edge-case table
Ambiguous results are joined back to original patent rows (including the abstract) to create a manual review table.

### 10) Export the final CSV
The final export:
- preserves all original columns
- appends the required classification fields
- sorts by year then by primary/secondary/tertiary class
- writes `outputs/goal2_patents_classified.csv`

It also writes `outputs/goal2_run_config.json` and `outputs/goal2_edge_cases.csv`.

## How to describe this in a research paper (methods wording)
Suggested wording:
- Patent abstracts were categorized using an open-source **MNLI zero-shot classifier**.
- Taxonomy labels were derived from a provided ferrofluids class-definition document.
- For each patent, the top-3 predicted classes were recorded along with a score-based reasoning trace.
- Because no hand-labeled reference set was available, we report diagnostic summaries and manually inspect low-margin cases.

## Important constraints and limitations
- **Abstract-only constraint:** title, CPC, and other metadata must not influence predictions.
- **No ground truth:** reported scores are model confidences, not measured accuracy.
- **Label text sensitivity:** small edits to taxonomy wording can change classification behavior.

## Reproducing a run
In Colab, run cells in order:
1) Mount Drive + `%cd`
2) Runtime setup
3) Merge/clean patents
4) Parse taxonomy
5) Run zero-shot classification
6) Export

Optionally run the refinement and edge-case sections for review.
