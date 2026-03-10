# Profit-Aware Credit Risk Model
## MSIN0097 Predictive Analytics — Individual Coursework

End-to-end credit default classification using the Home Credit Default Risk dataset, with a profit-maximising decision threshold under an asymmetric cost matrix (FN:FP = 5:1).

---

## Repository contents

| File | Description |
|------|-------------|
| `credit_default.ipynb` | Main notebook — all six steps plus optional appendices |
| `application_train.parquet` | Raw dataset (307,511 rows × 122 cols) — converted from CSV to reduce file size; see Data section below |
| `requirements.txt` | Python package versions |
| `agent_usage_log.md` | Agent Usage Log + Decision Register (coursework appendix) |

---

## Data

**Source:** [Home Credit Default Risk](https://www.kaggle.com/c/home-credit-default-risk/data) (Kaggle, public licence)

The notebook samples 100,000 rows from `application_train.parquet` at runtime using a stratified sample (`random_state=42`) to preserve the ~8% default rate. The parquet file was converted from the original `application_train.csv` to reduce file size (`pd.read_csv` → `df.to_parquet`).

Place `application_train.parquet` in the same directory as the notebook before running.

---

## Environment

Python **3.13.5** (Anaconda base environment).

Install all dependencies:

```bash
pip install -r requirements.txt
```

Or, if using conda:

```bash
conda install --file requirements.txt
```

> **Note:** The SHAP global importance cell (§6.1) uses XGBoost's native `pred_contribs=True` API rather than `shap.TreeExplainer`. This avoids a known incompatibility between XGBoost ≥ 3.0 and SHAP ≤ 0.47 (`base_score` parsing error). No workaround is needed if the versions in `requirements.txt` are used.

---

## How to run

1. Clone or download this repository.
2. Place `application_train.parquet` in the project directory.
3. Install dependencies (see above).
4. Open `credit_defult.ipynb` in JupyterLab or VS Code.
5. Select the **base** kernel (Python 3.13.5).
6. Run all cells in order: **Kernel → Restart Kernel and Run All Cells**.

Expected runtime: ~5–10 minutes (dominated by `GridSearchCV` in Steps 5.2a and 5.2c).

---

## Notebook structure

| Step | Heading | Contents |
|------|---------|----------|
| 0 | Environment setup | Imports, constants, random seed (`RANDOM_SEED = 42`) |
| 1 | Problem framing | Target definition, cost matrix, success metrics |
| 2 | EDA | Class imbalance, missingness, feature correlations |
| 3 | Data preparation | 60/20/20 stratified split, feature engineering, two preprocessing pipelines, multicollinearity check |
| 4 | Model comparison | LR, RF, XGBoost, MLP (baseline + stabilized) — validation AUC and profit |
| 5 | Fine-tuning & evaluation | LR and XGBoost CV tuning, profit-maximising threshold, calibration, test-set evaluation, sensitivity analysis |
| 6 | Final solution | SHAP global importance, employment feature deep-dive, model card |
| Appendix | EXT_SOURCE ablation | Quantifies signal loss if bureau scores are removed |
| Appendix | Segmented threshold | Compares global vs segment-specific vs oracle threshold policies |

---

## Key results (test set, n ≈ 20,000)

| Metric | Value |
|--------|-------|
| Model | XGBoost (tuned) |
| AUC-ROC | 0.7479 |
| Test profit | 10,753 units |
| Δ vs approve-all naive | +437 |
| Decision threshold (t*) | 0.285 (found on validation set) |
| Approval rate | 97.6% |
| Default recall | 12.3% |

---

## Reproducibility notes

- All random operations use `RANDOM_SEED = 42`.
- The stratified sample, train/val/test split, cross-validation folds, and XGBoost are all seeded.
- The test set is evaluated exactly once, at the fixed threshold found on the validation set.
- `AMT_GOODS_PRICE` is dropped in Step 3 (ρ ≈ 0.987 with `AMT_CREDIT`); rationale in §3.7 and `agent_usage_log.md` Entry 19.
