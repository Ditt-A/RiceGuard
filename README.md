# RiceGuard Sequential Notebooks

RiceGuard is a sequential notebook workflow for analyzing rice supply chain readiness in West Java, Indonesia. The project starts from the raw rice supply chain dataset, builds exploratory and actor-specific indicators, creates paper-grounded pseudo labels, and trains models to predict readiness classes.

Run the notebooks in order:

1. `01_data_understanding_visualization.ipynb`
2. `02_preprocessing_pseudo_labeling.ipynb`
3. `03_model_training_evaluation.ipynb`

Notebook 01 prepares and audits the source data, Notebook 02 builds the pseudo-labeled modeling dataset, and Notebook 03 evaluates two modeling modes before exporting the final operational model.

## Project Overview

The current revised workflow uses the v5 parser and retains valid rows even when the raw DMU value is blank. Those rows receive sequential actor-region DMU values and are tracked in the parser audit.

- Consolidated rows: 815
- Raw consolidated columns: 34
- Modeling columns after preprocessing: 79
- Actors: Farmer, Middlemen, Retail, Rice Miller, Wholesaler
- Regions: Garut, Indramayu, Karawang, Subang, Tasikmalaya
- Auto-assigned DMU rows: 28

Actor counts in the current consolidated dataset:

| Actor | Rows |
| --- | ---: |
| Farmer | 400 |
| Rice Miller | 105 |
| Middlemen | 104 |
| Wholesaler | 104 |
| Retail | 102 |

The target variable is `readiness_label` with three pseudo-label classes:

- `Recommended`
- `Conditional`
- `Not Recommended`

These labels are not external ground truth. They are generated from literature-derived rules, DEA-style efficiency, R/C feasibility, margin health, crisis stress tests, and cluster support.

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

The notebooks use the local dataset stored at:

```text
data/raw/Rice Supply Chain in West Java Province, Indonesia.zip
```

Dataset citation used in the project metadata:

Budiman (2023), *Rice Supply Chain in West Java Province, Indonesia*, Mendeley Data, DOI: `10.17632/k7c2sgmsj5.1`.

## Setup

Create a Python environment and install the core requirements:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Notebook 03 currently enables optional booster models when installed. To reproduce the latest saved run that selected LightGBM for the operational model, also install:

```powershell
python -m pip install lightgbm xgboost catboost
```

If those optional packages are unavailable, the notebook can still run with the scikit-learn model zoo, but the selected best model and metrics may differ.

## Paper-Grounded Domain Methodology

The methodology is adapted to the rice supply chain efficiency domain. Each row is treated as an actor-region decision-making unit and evaluated using financial, operational, and resilience indicators available in the dataset.

| Domain concept | Application in this project |
| --- | --- |
| Rice supply chain actors | Actor-aware features are built for farmers, middlemen, rice millers, wholesalers, and retailers. |
| DEA-style efficiency | A fast DEA-style frontier approximation estimates relative operational efficiency. It is not exact LP DEA. |
| R/C financial feasibility | `R/C >= 1` is used as the feasibility threshold for revenue-to-cost viability. |
| Margin health | Margin and margin ratio capture whether an actor-region unit remains financially positive after costs. |
| Asset utilization | Output and quantity proxies are compared against available scale or asset proxies. |
| Crisis resilience | Internal stress tests simulate cost increases and output decreases. |
| Weak supervision | Literature-derived rules create pseudo labels because external readiness labels are unavailable. |
| Cluster support | K-Means clustering supports the rule-based labels without forcing class balance. |

Notebook 02 currently selects `k = 3` clusters with silhouette score `0.2925`. The low-to-moderate silhouette is reported transparently because clustering is used only as a support signal, not as the main target.

The readiness score combines these components:

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

Method references used in the notebook metadata include Budiman (2023), Charnes, Cooper, and Rhodes (1978), Banker, Charnes, and Cooper (1984), Putra et al. (2024), Ratner et al. (2017), and Rousseeuw (1987).

## Notebook Pipeline

### 01 - Data Understanding and Focused Visualization

Loads the raw Excel data from the zip file, parses actor sheets into a long consolidated table, retains valid rows with missing raw DMU values, and creates exploratory indicators and visualizations.

Main outputs:

- `outputs/01_consolidated_raw.csv`
- `outputs/01_summary_by_actor_region.csv`
- `outputs/01_parser_audit.csv`
- `outputs/01_actor_config.json`
- `outputs/01_eda_metadata.json`
- EDA plots in `outputs/plots/`

### 02 - Preprocessing, Weak Labeling, Clustering, and Pseudo Labeling

Cleans the consolidated data, engineers actor-specific features, calculates DEA-style frontier efficiency, applies literature-derived rule components, adds cluster support, and creates the final `readiness_label`.

Main outputs:

- `outputs/02_modeling_dataset.csv`
- `outputs/02_labeling_report.csv`
- `outputs/02_actor_feature_summary.csv`
- `outputs/02_data_quality_report.csv`
- `outputs/02_cluster_selection.csv`
- `outputs/02_cluster_profile.csv`
- `outputs/02_methodological_notes.md`
- `outputs/02_preprocessing_metadata.json`

Current label distribution:

| Readiness label | Count | Share |
| --- | ---: | ---: |
| Conditional | 374 | 45.89% |
| Not Recommended | 268 | 32.88% |
| Recommended | 173 | 21.23% |

### 03 - Model Training, Two-Mode Evaluation, and Export

Notebook 03 evaluates two feature modes:

| Mode | Purpose | Feature set |
| --- | --- | --- |
| `operational_prediction` | Deployment-oriented prediction | Uses operational/raw features and excludes final readiness score, rule component scores, cluster support, votes, and survival flags. |
| `rule_replication` | Sanity check for rule reproducibility | Uses operational features plus rule-derived signals, so scores are expected to be higher. |

The final exported model uses `operational_prediction`, because it avoids directly feeding label-construction features back into the model.

Main outputs:

- `outputs/03_all_modes_cv_model_comparison.csv`
- `outputs/03_holdout_metrics_by_mode.csv`
- `outputs/03_revised_modeling_summary.json`
- `outputs/03_operational_prediction_classification_report.csv`
- `outputs/03_operational_prediction_holdout_predictions.csv`
- `outputs/03_operational_permutation_feature_importance.csv`
- `outputs/03_rule_replication_classification_report.csv`
- `outputs/03_rule_replication_holdout_predictions.csv`
- `outputs/models/riceguard_operational_best_model.joblib`
- `outputs/models/riceguard_operational_best_model_metadata.json`

Current holdout results:

| Mode | Best model | Accuracy | Balanced accuracy | Macro F1 | Weighted F1 | Features |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| `operational_prediction` | LightGBM | 0.9363 | 0.9286 | 0.9343 | 0.9362 | 52 |
| `rule_replication` | Extra Trees | 0.9755 | 0.9766 | 0.9746 | 0.9755 | 74 |

Current cross-validation leaders:

| Mode | Best model | CV folds | Macro F1 mean | Balanced accuracy mean | Selection score |
| --- | --- | ---: | ---: | ---: | ---: |
| `operational_prediction` | LightGBM | 5 | 0.9303 | 0.9303 | 0.9303 |
| `rule_replication` | Extra Trees | 5 | 0.9559 | 0.9573 | 0.9565 |

The selection score is:

```text
0.60 * MacroF1 + 0.40 * BalancedAccuracy
```

## Important Interpretation Notes

- The target is a pseudo label generated by Notebook 02, not externally validated field truth.
- Rule-replication performance is useful as a sanity check, but it is not the deployment target.
- Operational performance is the more realistic metric because label-construction features are excluded.
- Crisis resilience uses internal cost and output shocks only. It does not use external price, road, conflict, policy, or population data.
- Existing model metrics should be interpreted as reproducible research results, not as production-grade recommendation guarantees.

## Reproducing the Results

Run the notebooks sequentially:

```text
01_data_understanding_visualization.ipynb
02_preprocessing_pseudo_labeling.ipynb
03_model_training_evaluation.ipynb
```

Each notebook reads artifacts produced by the previous notebook and writes updated files to `outputs/`. To regenerate all outputs, rerun the notebooks from Notebook 01 through Notebook 03 using the same environment and optional booster packages.
