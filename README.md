# GAT-LOOCV-Freezing-Prediction

This repository contains a full implementation of a **Graph Attention Network (GAT)** model for predicting freezing behavior during fear extinction, using graph representations of single-unit electrophysiological data. The model uses per-neuron features and functional connectivity edge weights to build per-animal graphs and evaluate performance using **Leave-One-Out Cross-Validation (LOOCV)**.

The repository also includes optional **Optuna-based hyperparameter tuning** using 5-fold cross-validation, extensive diagnostic visualizations, and segment-specific prediction pipelines.

---

## 📁 Repository Structure

```
GAT-FreezingBehavior-LOOCV/
│
├── gat_loocv_model.ipynb          # Full GAT pipeline: data loading, graph creation, LOOCV, Optuna
├── README.md                      # This file
├── requirements.txt               # Python dependencies
├── .gitignore                     # Files to exclude from Git tracking
├── LICENSE                        # MIT License for open reuse
```

---

## Inputs

* `neuron_features.xlsx`: Contains per-neuron features (e.g., firing rate, ISI metrics, PC1/PC2 scores) with `Unique ID` indicating animal identity.
* `edge_weights.xlsx`: Contains normalized mean edge weights between neuron pairs across time windows.
* `labels.xlsx`: Contains percentage freezing behavior for each animal (`UniqueID`) and group assignment.

Each animal is represented as a graph where:

* **Nodes** = neurons
* **Node features** = electrophysiological and PCA-derived features
* **Edges** = functional connectivity between neurons (edge weights)
* **Graph label** = percent freezing (behavioral output)

---

## How to Run

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Prepare input files

Ensure you have:

* `neuron_features.xlsx`
* `edge_weights.xlsx`
* `labels.xlsx`

Update their file paths inside `gat_loocv_model.ipynb`.

### 3. Choose tuning mode

* To **manually specify best hyperparameters**, set `run_optuna = False` and use preselected `best_params_*` values.
* To **run Optuna tuning**, set `run_optuna = True`.

### 4. Run the notebook

Execute all cells sequentially. You can run only the first or last 600s model by toggling:

```python
run_first600 = True
run_last600 = True
```

---

## Model Overview

| Component        | Details                                                |
| ---------------- | ------------------------------------------------------ |
| Graph type       | Undirected, one graph per animal                       |
| Nodes            | Neurons                                                |
| Node features    | Firing rate, ISI stats, PC1/PC2 scores                 |
| Edges            | Mean dot products of firing rate vectors               |
| Edge attributes  | Normalized weights (0–1)                               |
| Graph label (y)  | Freezing % (continuous)                                |
| Readout function | `global_mean_pool` followed by sigmoid scaled to 0–100 |

---

## Cross-Validation Strategy

| Purpose                | Method                                 |
| ---------------------- | -------------------------------------- |
| Hyperparameter tuning  | K-Fold (5-fold) via Optuna             |
| Final model evaluation | Leave-One-Out Cross-Validation (LOOCV) |

---

## Output

* LOOCV-predicted vs. actual plots (group-colored)
* Per-fold and average training/validation loss curves
* Aggregated R² values
* Individual model predictions per animal

---

## Notes

* **Optuna tuning** performs 5-fold CV and minimizes validation loss minus R².
* **LOOCV** is used for final generalization testing.
* All features are normalized using `StandardScaler`.
* Model outputs are squashed using a `sigmoid()` and scaled to 0–100 to match freezing percentage scale.

---

