# RiceGuard Sequential Notebooks

RiceGuard is a notebook-based machine learning workflow for analyzing rice supply chain readiness in West Java, Indonesia. The project starts from the raw rice supply chain dataset, builds exploratory indicators, generates paper-grounded pseudo labels, and trains a classifier to predict readiness classes for supply chain actors.

The workflow is intentionally sequential:

1. `01_data_understanding_visualization.ipynb`
2. `02_preprocessing_pseudo_labeling.ipynb`
3. `03_model_training_evaluation.ipynb`

Notebook 01 produces cleaned and consolidated exploratory outputs, Notebook 02 turns those outputs into a modeling dataset with pseudo labels, and Notebook 03 trains and evaluates classification models.

## Project Overview

This repository focuses on five rice supply chain actors across five West Java regions:

- Actors: Farmer, Middlemen, Retail, Rice Miller, Wholesaler
- Regions: Garut, Indramayu, Karawang, Subang, Tasikmalaya
- Dataset size after consolidation: 787 rows and 31 raw columns

The final modeling task predicts three readiness labels:

- `Recommended`
- `Conditional`
- `Not Recommended`

These labels are not external ground truth. They are generated through weak supervision using literature-derived rules, DEA-style efficiency scoring, R/C ratio feasibility, margin health, crisis stress tests, and cluster support.

## Repository Structure

```text
.
|-- 01_data_understanding_visualization.ipynb
|-- 02_preprocessing_pseudo_labeling.ipynb
|-- 03_model_training_evaluation.ipynb
|-- data/
|   `-- raw/
|       `-- Rice Supply Chain in West Java Province, Indonesia.zip
|-- outputs/
|   |-- *.csv
|   |-- *.json
|   |-- plots/
|   `-- models/
|-- requirements.txt
`-- README.md
```

## Data Source

The notebooks use only the dataset stored at:

```text
data/raw/Rice Supply Chain in West Java Province, Indonesia.zip
```

Dataset citation used in the project metadata:

Budiman (2023), *Rice Supply Chain in West Java Province, Indonesia*, Mendeley Data, DOI: `10.17632/k7c2sgmsj5.1`.

## Paper-Grounded Domain Methodology

The methodology is adapted to the rice supply chain efficiency domain described by the source dataset. Each observation is treated as a decision-making unit for an actor-region combination, then evaluated using financial, operational, and resilience indicators that are available in the dataset.

| Domain concept | How it is applied in this project |
| --- | --- |
| Rice supply chain actors | Farmers, middlemen, rice millers, wholesalers, and retailers are modeled with actor-aware cost, output, asset, and transaction features. |
| DEA efficiency | DEA-style scores are used to approximate relative operational efficiency because the source dataset is designed for supply chain efficiency analysis. |
| R/C financial feasibility | Revenue-to-cost viability is represented with the `R/C >= 1` feasibility threshold, where values below 1 indicate weaker economic feasibility. |
| Margin health | Margin and margin ratio are used to capture whether each actor-region unit remains financially positive after costs. |
| Asset utilization | Output and quantity proxies are compared with available asset or scale proxies to estimate whether resources are being used efficiently. |
| Crisis resilience | Internal stress tests simulate cost increases and output decreases to estimate whether the unit can remain viable under shocks. |
| Weak supervision | Literature-derived rules generate pseudo labels when no external ground-truth readiness labels are available. |
| Cluster support | K-Means clustering adds data-driven support to the rule-based labels without forcing artificial class balance. |

The final readiness score in Notebook 02 combines these components:

| Component | Weight |
| --- | ---: |
| DEA score | 0.30 |
| R/C viability | 0.20 |
| Margin score | 0.15 |
| Crisis resilience | 0.15 |
| Asset utilization | 0.10 |
| Cluster support | 0.10 |

Readiness labels are assigned using these thresholds:

| Label | Score range |
| --- | --- |
| Not Recommended | `< 0.40` |
| Conditional | `0.40 <= score < 0.70` |
| Recommended | `>= 0.70` |

The methodology references used in the notebook metadata include DEA CCR, DEA BCC, R/C financial feasibility, weak supervision, and silhouette-based cluster validation.

## Setup

Create a Python environment and install the required packages:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Then open the notebooks in VS Code, Jupyter Notebook, or JupyterLab and run them in the required order.

## Notebook Pipeline

### 01 - Data Understanding and Visualization

Loads the raw Excel data from the zip file, parses actor-specific sheets into a consolidated long table, creates exploratory indicators, and saves outputs for the next step.

Main outputs:

- `outputs/01_consolidated_raw.csv`
- `outputs/01_summary_by_actor_region.csv`
- `outputs/01_actor_config.json`
- EDA plots in `outputs/plots/`

### 02 - Preprocessing and Pseudo Labeling

Cleans the consolidated data, engineers actor-specific features, computes DEA-style efficiency, applies rule-based weak supervision, adds cluster support, and creates the final readiness labels.

Main outputs:

- `outputs/02_modeling_dataset.csv`
- `outputs/02_labeling_report.csv`
- `outputs/02_actor_feature_summary.csv`
- `outputs/02_data_quality_report.csv`
- `outputs/02_methodological_notes.md`

Current label distribution:

| Readiness label | Count | Share |
| --- | ---: | ---: |
| Recommended | 345 | 43.84% |
| Conditional | 314 | 39.90% |
| Not Recommended | 128 | 16.26% |

### 03 - Model Training and Evaluation

Trains multiple classification models using the pseudo-labeled modeling dataset, compares cross-validation performance, evaluates the best model on a holdout split, exports predictions, and saves the final model.

Main outputs:

- `outputs/03_cv_model_comparison.csv`
- `outputs/03_best_model_holdout_metrics.csv`
- `outputs/03_best_model_classification_report.csv`
- `outputs/03_feature_importance.csv`
- `outputs/models/riceguard_best_model.joblib`

Best holdout model:

| Model | Accuracy | Balanced accuracy | Macro F1 | Weighted F1 |
| --- | ---: | ---: | ---: | ---: |
| Extra Trees | 0.9645 | 0.9588 | 0.9617 | 0.9645 |

## Methodological Notes

- The model learns from pseudo labels, not externally validated field labels.
- R/C feasibility uses the common financial viability threshold `R/C >= 1`.
- DEA-style scoring is included because the source dataset is designed for rice supply chain efficiency analysis.
- Crisis resilience is simulated using internal cost and output shocks only.
- The results should be interpreted as a reproducible research and decision-support prototype, not as a final production recommendation system.

## Reproducing the Results

Run the notebooks in this order:

```text
01_data_understanding_visualization.ipynb
02_preprocessing_pseudo_labeling.ipynb
03_model_training_evaluation.ipynb
```

Each notebook reads artifacts produced by the previous notebook and writes new artifacts to `outputs/`. To regenerate all results from scratch, clear or ignore the existing files in `outputs/` and rerun the notebooks sequentially.
