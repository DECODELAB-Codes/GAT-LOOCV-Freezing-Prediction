# GAT-LOOCV-Freezing-Prediction

This repository contains a full implementation of a **Graph Attention Network (GAT)** model for predicting freezing behavior during fear extinction, using graph representations of single-unit electrophysiological data. The model uses per-neuron features and functional connectivity edge weights to build per-animal graphs and evaluate performance using **Leave-One-Out Cross-Validation (LOOCV)**.

The pipeline is implemented in a single Jupyter Notebook and includes optional **edge attribute integration**, optional **Optuna-based hyperparameter tuning** using 5-fold cross-validation, extensive diagnostic visualizations, and segment-specific prediction pipelines.

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

- `neuron_features.xlsx`: Neuron-level features (e.g., firing rate, ISI metrics, PC1/PC2 scores) with `Unique ID` per animal.
- `edge_weights.xlsx`: Normalized mean edge weights for each neuron pair (first and last 600s).
- `labels.xlsx`: Animal-level freezing percentages (`UniqueID`) and group label (e.g., control, ELS-Veh, ELS-ACTH).

Each animal is represented as a graph where:

| Component      | Description                                  |
| -------------- | -------------------------------------------- |
| **Nodes**      | Neurons                                      |
| **Features**   | Firing rate, ISI metrics, PC scores (per node) |
| **Edges**      | Functional connectivity (mean dot products)  |
| **Edge Attr**  | Optional — normalized weights (0–1)          |
| **Graph Label**| Freezing % (continuous, 0–100)               |

---

## How to Run

### 1. Install dependencies

# Step 1: Install core PyTorch (CPU-only)
pip install torch==2.2.0 torchvision==0.17.0 torchaudio==2.2.0

# Step 2: Install torch-geometric for PyTorch 2.2.0 (CPU-only)
pip install torch-geometric -f https://data.pyg.org/whl/torch-2.2.0+cpu.html

# Step 3: Install project dependencies
pip install -r requirements.txt


### 2. Prepare input files

Ensure you have:

* neuron_features_path = "path/to/neuron_features.xlsx"
* edge_weights_path = "path/to/edge_weights.xlsx"
* labels_path = "path/to/labels.xlsx"

Update their file paths inside `gat_loocv_model.ipynb`.

### 3. Choose GAT Model Type

Toggle this flag to control model architecture:

use_edge_weights = True  # Set False to ignore edge attributes

### 4. Choose tuning mode

* To **manually specify best hyperparameters**, set `run_optuna = False` and use preselected `best_params_*` values.
* To **run Optuna tuning**, set `run_optuna = True`.

### 5. Run the notebook

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
| Model 1 (default)| Standard GAT using GATConv                             |
| Model 2 (opt)    |	Edge-aware GAT using TransformerConv                  |
| Readout function | `global_mean_pool` followed by sigmoid scaled to 0–100 |

---

## Cross-Validation Strategy

| Purpose                | Method                                 |
| ---------------------- | -------------------------------------- |
| Hyperparameter tuning  | K-Fold (5-fold) via Optuna             |
| Final model evaluation | Leave-One-Out Cross-Validation (LOOCV) |

---

## Output

* Prediction vs. actual freezing plots (color-coded by group)
* Per-fold and average loss curves (training + validation)
* Aggregated LOOCV R² score
* Printout of individual predictions for each animal

---

## Notes

* **Optuna tuning** performs 5-fold CV and minimizes validation loss minus R².
* **LOOCV** is used for final generalization testing, independent of tuning
* All features are normalized using `StandardScaler`.
* Model outputs are squashed using a `sigmoid()` and scaled to 0–100 to match freezing percentage scale.
* Edge attribute support is optional via use_edge_weights toggle.

---

