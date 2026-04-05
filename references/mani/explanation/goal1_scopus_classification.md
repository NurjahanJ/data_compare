# Explanation: `goal1_scopus_classification.ipynb`

## What this notebook is for (Goal #1)
This notebook classifies Scopus publications into a predefined **ferrofluids taxonomy** using **only the Abstract text**. It produces, for every publication, a **Primary**, **Secondary**, and **Tertiary** class code (plus human-readable descriptions) and a short **Reasoning** string.

The intended use is a research workflow where:
- you have a fixed taxonomy (the project-provided ferrofluids class definitions),
- you have many documents without hand-labeled classes,
- and you want a reproducible, auditable automated first pass.

## Audience-friendly summary of the method
The notebook uses a technique called **zero-shot text classification**.
- “Zero-shot” means we do **not** train a new model on your dataset.
- Instead, we use a pre-trained language model (from Hugging Face) that can decide which of several **candidate labels** best matches a piece of text.

Here, the candidate labels are built directly from your taxonomy file, so the model is effectively asked:
> “Is this abstract about *[class label]*?”

It scores each possible class label and returns the highest-scoring ones.

## Inputs (files read)
From this repository:
- `raw_data/MANI_KW_PAPERS_scopus.csv` (Scopus publications)
- `classification/FEROFLUIDS_CLASS_DEFINITION.md` (taxonomy: codes + descriptions)
- `docs/INSTRUCTIONS.md` and `docs/AI_CLASSIFICATON_IS421_2026S.md` (assignment constraints shown in-notebook)

## Outputs (files written)
Main deliverable:
- `outputs/goal1_scopus_classified.csv`

Diagnostics and reproducibility artifacts:
- `goal1_requirements.json` (written at repo root)
- `outputs/goal1_metrics_primary_class_counts.csv`
- `outputs/goal1_metrics_ambiguous_top25.csv`
- `outputs/goal1_edge_cases.csv`
- `outputs/goal1_run_config.json`

Optional refinement output:
- `outputs/goal1_bart_refinement_spotcheck.csv`

## Step-by-step: what the code does
### 0) Colab setup
The notebook is written to run in **Google Colab**:
- mounts Google Drive
- changes directory to the project folder

### 1) Runtime setup (dependencies + configuration)
The notebook installs Python packages in a way that tries not to break Colab’s preinstalled scientific stack:
- pins `pandas==2.2.2`, `numpy<2.2`, `scipy<1.13` once (then forces a runtime restart)
- installs Hugging Face tooling (`transformers`, `accelerate`, `sentencepiece`, `tqdm`)

It then defines a `CONFIG` dictionary with:
- input paths
- output paths
- the model name (`typeform/distilbert-base-uncased-mnli`)
- batch size and a simple hypothesis template (`"This text is about {}."`)

A fixed random seed is set for reproducibility (`SEED = 421`) for NumPy and PyTorch.

### 2) Display instructions (traceability)
The notebook reads the `docs/` instruction files and renders them in the notebook so the rules (especially “Abstract only”) remain visible while running.

### 3) Record requirements to JSON
It writes a compact, machine-readable description of Goal #1 requirements to `goal1_requirements.json`. This is useful for auditability: the deliverable schema and constraints are recorded alongside the code run.

### 4) Load Scopus CSV and clean minimal issues
The dataset ingest step:
- reads the Scopus CSV as strings
- checks that `Year`, `Title`, and `Abstract` exist
- drops rows where year is not numeric or abstract is empty
- adds a required `Serial Number` column (1..N)

Importantly, the code creates `Abstract_clean` and uses that for classification.

#### What `Abstract_clean` means
`Abstract_clean` is a minimally cleaned version of the raw `Abstract` field designed to enforce the “abstract-only” constraint and avoid obvious input issues.

In the notebook it is created by:
- filling missing abstracts with an empty string
- converting the value to a string
- trimming leading/trailing whitespace (`.str.strip()`)

It is **not** an advanced NLP preprocessing step (no stemming/lemmatization, no stopword removal, no punctuation normalization). Its main purposes are:
- ensure the classifier receives only abstract text (and never title/metadata)
- allow reliable filtering of empty abstracts

After classification, `Abstract_clean` is treated as a helper column and is dropped from the final exported CSV.

### 5) Parse the ferrofluids taxonomy from Markdown
The notebook reads `classification/FEROFLUIDS_CLASS_DEFINITION.md` and extracts a list of classes.

Implementation details:
- it detects class rows by a two-digit code at the start of a line
- it builds a “candidate label” string that includes code + major + description

This is done so model predictions can be mapped unambiguously back to numeric class codes and descriptions.

### 6) Zero-shot classification (Abstract-only)
The core classification step:
- creates a Hugging Face `pipeline(task='zero-shot-classification')`
- provides the **full set of candidate labels** (one per taxonomy entry)
- runs the pipeline in batches over the list of abstracts

For each abstract, the model returns a ranked list of labels with confidence-like scores.
The notebook then:
- takes the **top 3** labels
- maps them back to:
  - `Primary Class`, `Secondary Class`, `Tertiary Class`
  - `Primary Desc`, `Secondary Desc`, `Tertiary Desc`
- constructs a `Reasoning` string that records the top-3 labels and scores

### 7) Diagnostics (not “accuracy”)
Because there is no hand-labeled ground truth in the assignment, the notebook does **not** compute accuracy or F1.

Instead, it computes:
- a distribution table of primary classes
- an “ambiguity” proxy: `margin_12 = Top1_score - Top2_score`

Small margins mean the model found two classes similarly plausible, so these are good candidates for human review.

### 8) Optional refinement pass (BART spot-check)
This step optionally reruns the **most ambiguous** rows using a heavier MNLI model, then compares those results against the baseline model.

What it does:
- **Baseline model (full dataset):** `typeform/distilbert-base-uncased-mnli`
- **Refinement model (subset only):** `facebook/bart-large-mnli`

How rows are selected:
- The notebook computes an ambiguity proxy `margin_12 = Top1_score - Top2_score` from the baseline run.
- It selects the rows with the **smallest margins** (default `REFINE_N = 200`). These are the cases where the baseline model’s top two labels were nearly tied.

What BART does in this context:
- BART runs its **own** abstract-only zero-shot classification over the *same* candidate labels and hypothesis template, but only for the selected subset.
- Importantly, it scores the abstract against the **entire candidate label set** (i.e., the full taxonomy label list in `candidate_labels`), not just the baseline model’s top-3.
- The notebook then records BART’s **top-3** labels/scores (Primary/Secondary/Tertiary) for side-by-side comparison.
- The notebook builds a **side-by-side comparison table** (baseline vs BART) and writes it to `outputs/goal1_bart_refinement_spotcheck.csv` for manual review.

Whether BART changes the final output:
- If `APPLY_REFINEMENT = False`, this is **compare-only** (no changes to the baseline classifications).
- If `APPLY_REFINEMENT = True`, the notebook **overwrites the baseline predictions only for the selected rows** with BART’s predictions, then proceeds with export.

### 9) Edge-case table
The notebook joins the ambiguous classifications back to the original Scopus rows (including the full abstract) to create a manual review table.

### 10) Export the final CSV
Finally, the notebook:
- concatenates the original dataset with the classification columns
- sorts by year then primary/secondary/tertiary class
- drops helper columns (`Year_num`, `Abstract_clean`)
- writes `outputs/goal1_scopus_classified.csv`

It also writes a run configuration snapshot and the edge-case table.

## How to describe this in a research paper (methods wording)
A reasonable plain-language description is:
- We applied **abstract-only zero-shot classification** using an open-source MNLI model.
- Candidate labels were constructed from a project-defined taxonomy (codes + descriptions).
- For each document, we recorded the top-3 predicted classes and a traceable score-based reasoning string.
- Since no gold labels were available, we report diagnostic summaries (class distributions and ambiguity margins) rather than accuracy metrics.

## Important constraints and limitations
- **Abstract-only constraint:** the notebook intentionally ignores title/keywords/metadata.
- **No ground truth:** model scores are not “accuracy”; they are internal confidence signals.
- **Taxonomy phrasing matters:** because labels come from the taxonomy text, edits to definitions can change results.
- **Computational cost:** zero-shot over many labels can be slow; batching is used to reduce overhead.

## Reproducing a run
In Colab, run cells in order:
1) Mount Drive + `%cd`
2) Runtime setup (installs + `CONFIG`)
3) Data ingest + taxonomy parsing
4) Zero-shot classification
5) Export

If you enable refinement, run that after diagnostics.
