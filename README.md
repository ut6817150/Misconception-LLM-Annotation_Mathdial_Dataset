# Misconception annotation pipeline

This repository prepares MathDial annotation sets, evaluates misconception-
annotation prompts, runs the selected Kimi K3 extraction, and writes valid
cached annotations into the final misconception-labelled datasets.

Modelling is intentionally out of scope. The original paper's supplied data is
retained under `data/`, but its BKT and other knowledge-tracing implementations
live in a separate modelling repository.

## Repository structure

```text
.
├── 00_annotation_data_preprocessing.ipynb               # Builds annotation sets and reformats MathDial
├── 01_annotation_extraction_naive_various_models.ipynb  # Compares candidate annotation models
├── 02_annotation_extraction_validation_kimi_k3.ipynb    # Evaluates P0–P12 and selects P11
├── 03_annotation_extraction_evaluation_kimi_k3.ipynb    # Runs P11 twice on held-out evaluation data
├── 04_annotation_extraction_train_test_kimi_k3.ipynb    # Extracts P11 annotations for train and test
├── 05_annotation_cache_to_misconception_dataset.ipynb   # Writes valid cache into final datasets
├── artifacts/                                           # Versioned experiment inputs and outputs
│   ├── annotation_dev_val_and_eval_sets/                # Gold subsets and dialogue-ID manifests
│   ├── annotation_prompts/                              # Prompt templates P0–P12
│   ├── codebooks/                                       # Full misconception codebook versions
│   └── extraction_cache/                                # Per-dialogue model responses
├── data/                                                # Source, intermediate, and final datasets
│   ├── src/                                             # Original source material
│   ├── annotated/                                       # Paper-provided MathDial annotations
│   └── misconception/                                   # Final misconception-labelled datasets
└── scripts/                                             # Shared pipeline code
    ├── annotation/                                      # Prompting, extraction, validation, and scoring
    └── data_management/                                 # Loading, formatting, and cache application
```

## Notebook workflow

Run the notebooks in numerical order:

1. **Data preprocessing** constructs the development, validation, and
   evaluation sets and reformats the paper-provided MathDial data.
2. **Naive extraction** compares candidate models on a shared P0 prompt and
   validation subset.
3. **Kimi K3 validation** evaluates prompt versions P0–P12 and documents the
   selection of P11.
4. **Kimi K3 evaluation** runs two independent P11 annotations against the
   held-out evaluation set.
5. **Train/test extraction** applies P11 to the complete reformatted MathDial
   train and test splits.
6. **Cache application** validates available P11 cache records and writes their
   annotations into `data/misconception/mathdial_{train,test}.csv`.

Valid cache records are reused. Missing, unreadable, or invalid records may
trigger paid API requests when extraction is enabled, so inspect each
notebook's run switches before executing it.

### Why codebook v11 was selected

P11/codebook v11 was selected on the validation set, where it achieved the
highest overall performance among P0--P12: macro F1 of 0.829, accuracy of
0.955, and Krippendorff's alpha of 0.876. P12 improved F1 for the rare
principles family but regressed in the other four families, reducing overall
macro F1 to 0.759, accuracy to 0.920, and alpha to 0.773. The later held-out
evaluation checked rather than determined this choice: P11 produced two fully
valid 36-dialogue runs with nearly identical overall accuracy (0.959 and
0.960) and alpha/kappa (0.881 in both), although macro F1 varied from 0.870 to
0.937. P11 was therefore retained as the production prompt because validation
showed the strongest broad family performance and evaluation showed stable
overall agreement with the gold annotations.

## Setup

Create an environment and install the annotation-only dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

API-backed notebooks read credentials from environment variables or the local
`.env` file:

```text
OPENROUTER_API_KEY=...
MOONSHOT_API_KEY=...
```

The `.env` file is ignored by Git and must never be committed.

Run notebooks from the repository root so that `artifacts/`, `data/`, and
`scripts/` resolve consistently.

## Data and artifacts

- `data/src/` contains source MathDial material and its accompanying metadata.
- `data/annotated/` contains the original paper-provided annotations used as
  input to this pipeline.
- `data/misconception/` contains the reformatted and misconception-labelled
  train/test datasets.
- `artifacts/annotation_dev_val_and_eval_sets/` contains the pinned annotation
  subsets and their dialogue-ID manifests.
- `artifacts/annotation_prompts/` and `artifacts/codebooks/` contain the versioned
  annotation specifications.
- `artifacts/extraction_cache/` contains raw model responses separated by split,
  model, prompt, dialogue, and—where applicable—evaluation run.

The extraction cache and dataset CSVs are large. Moving them does not reduce
existing Git history; use Git LFS or external artifact storage if repository
clone size becomes a concern.
