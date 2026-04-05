# Explanation: `goal7_topic_modeling.ipynb`

## What this notebook is for (Goal #7)
This notebook performs **topic modeling** *within each major class* to create a **sub-classification** that helps distinguish how Scopus papers vs patents differ in their “themes” even when they share the same broad major class.

It is designed to be run **after Goal #6**, using the Goal #6 outputs as inputs.

In plain terms:
- Goal #6 tells you *what kind of activity* a document represents (Material / Computation / …).
- Goal #7 then tries to discover *what sub-topics* exist inside that activity class, and whether those sub-topics are more common in patents vs papers.

## Inputs (files read)
From Goal #6 outputs:
- `outputs/goal6/goal6_scopus_major_classified.csv`
- `outputs/goal6/goal6_patents_major_classified.csv`

## Outputs (files written)
All Goal #7 outputs are written under `outputs/goal7/`.

Required CSV deliverables:
- `goal7_scopus_topic_subclassified.csv`
- `goal7_patents_topic_subclassified.csv`

Optional artifacts (only if you enable them):
- per-major-class plots under `outputs/goal7/major_<code>/` (UMAP scatter, prevalence bars, heatmaps, etc.)
- `goal7_lda_topic_summary.csv` (only if LDA baseline is enabled)

## Audience-friendly summary of “topic modeling”
Topic modeling is an unsupervised method that groups documents by shared vocabulary/themes.
- It does not require labeled training data.
- It is exploratory: topics are discovered from patterns in the text.

This notebook uses **BERTopic**, which combines:
1) a model that converts text into numeric vectors (“embeddings”),
2) a clustering algorithm that groups similar vectors,
3) a simple word-count model that summarizes each cluster with its top terms.

## Step-by-step: what the code does
### 0) Colab-only runtime setup
The notebook explicitly requires Colab. It:
- installs BERTopic and supporting libraries (`sentence-transformers`, `umap-learn`, `hdbscan`)
- installs plotting/export libraries (`seaborn`, `plotly`, `kaleido`)
- sets a fixed random seed (`SEED = 421`)
- defines a `CONFIG` dictionary controlling text length thresholds, topic model parameters, and outputs

### 1) Load Goal #6 CSV inputs and validate schema
It reads the two Goal #6 CSVs and checks that required columns exist, including:
- `Serial Number`, `Title`, `Abstract`
- `Primary Major Class`, `Primary Major Label`, `Primary Major Score`

### 2) Normalize schemas and build a unified corpus
The notebook constructs a unified table with consistent fields across sources:
- `doc_id` = `<source>:<serial>`
- `source` = `scopus` or `patent`
- `year`
- `major_class`, `major_label`, `major_score`
- `text_raw` = **Abstract only**

It also deduplicates documents using a hash of normalized abstract text (this removes exact duplicates after whitespace normalization).

### 3) Text cleaning pipeline
The cleaning step produces a `text_clean` field used for topic modeling. It:
- lowercases, normalizes Unicode, removes punctuation
- removes very short tokens
- removes standard English stopwords plus domain-specific boilerplate stopwords (different sets for patents vs papers)

It then drops documents that are too short (thresholds controlled by `CONFIG['min_chars']` and `CONFIG['min_tokens']`).

### 4) Stratify by major class and filter for sufficient sample size
Topic models need enough documents to form stable clusters.

For each major class, the notebook counts documents by source and keeps only those classes meeting thresholds:
- at least `min_docs_total` total documents
- at least `min_docs_per_source` from patents and from Scopus

This prevents running topic modeling on tiny subsets where results would be unreliable.

### 5) TF‑IDF + SVD baseline (quick sanity check)
Before fitting topic models, the notebook optionally visualizes whether papers and patents separate at all in a simple text-vector space:
- TF‑IDF vectorization (weighted word features)
- 2D projection using Truncated SVD

This is not the final method; it is a “do we see any structure?” check.

### 6) Topic modeling per major class (BERTopic)
For each retained major class, the notebook fits BERTopic on the combined corpus (patents + papers together).

Implementation details (high-level):
- **Embeddings:** `SentenceTransformer('all-MiniLM-L6-v2')` converts each abstract into a numeric vector.
- **Dimensionality reduction:** UMAP reduces embedding dimensionality to make clustering easier.
- **Clustering:** HDBSCAN groups documents into clusters (topics). Documents that don’t fit well become outliers (`topic_id = -1`).
- **Topic summaries:** a CountVectorizer extracts top words/phrases per topic.

The notebook records:
- per-topic summaries (topic size, top terms)
- per-document topic assignments and a topic probability proxy

### 7) Quantify “organic separation” between patents and papers (JSD)
Within each major class, the notebook compares how topic prevalence differs by source.

It computes **Jensen–Shannon divergence (JSD)** between the topic distribution of patents and the topic distribution of Scopus papers.
- Higher JSD means the two sources “use different topics” more strongly within that major class.

It also labels topics by skew:
- patent-dominant
- scopus-dominant
- mixed

### 8–10) Inspection + visualization (optional)
The notebook provides helpers to:
- print top terms and example snippets per topic
- generate plots per major class (UMAP scatter, prevalence bars, heatmaps, topics-over-time)
- optionally save interactive BERTopic HTML visualizations

By default, the visualization run is disabled (`RUN_VISUALIZATIONS = False`) to avoid long runtimes.

### 11) Export the two required CSVs (sub-classification)
Finally, the notebook writes exactly two CSVs:
- `goal7_scopus_topic_subclassified.csv`
- `goal7_patents_topic_subclassified.csv`

It adds one new column:
- `Primary Major Subclass`

This is derived from topic assignments within the primary major class. A stable label is built as:
- `Topic <id>: <top_terms>`

Documents not included in modeling (e.g., due to short text or filtered-out classes) are labeled `Not modeled`. Outliers are labeled `Outlier/Other`.

### 12) Optional LDA baseline
A simplified topic-model baseline (LDA) can be enabled for sanity checking. It can export an additional CSV summary, but it is disabled by default.

## How to describe this in a research paper (methods wording)
Suggested wording:
- We performed within-class topic modeling using BERTopic on abstract-only corpora stratified by major class.
- Patents and publications were modeled jointly within each class to yield shared topic definitions.
- Topic separation by source was quantified using Jensen–Shannon divergence on topic prevalence distributions.
- Each document received a topic-based subclass label within its primary major class.

## Important constraints and limitations
- **Exploratory method:** topics are discovered, not “true labels.” Interpretation requires domain judgment.
- **Parameter sensitivity:** BERTopic/UMAP/HDBSCAN settings can change topic structure.
- **Outliers:** documents assigned to `topic_id = -1` are treated as “other” and may include heterogeneous text.
- **Abstract-only:** titles and metadata are excluded intentionally.

## Reproducing a run
1) Run Goal #6 first to generate the two input CSVs.
2) In Colab, run Goal #7 cells in order:
   - setup + installs
   - load inputs
   - build corpus + clean
   - stratify + fit BERTopic
   - export

Optionally enable `RUN_VISUALIZATIONS` for saved plots.
