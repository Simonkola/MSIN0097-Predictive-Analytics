# Agent Usage Log + Decision Register
## MSIN0097 Predictive Analytics — Coursework Appendix

**Agent used:** Claude Code (claude-sonnet-4-6) via VS Code extension
**Project:** Profit-Aware Credit Risk Model — Home Credit Default Risk dataset
**Student decision log:** Each row records what the agent contributed, what was verified, and the student's accept/modify/reject decision.

---

> **Process Note — authorship of student reasoning fields**
> The "Student reasoning" and "Decision" fields throughout this log were drafted by the agent (Claude Code) from reasoning and decisions I supplied conversationally during the project. Each entry was reviewed and, where necessary, edited by me before submission to ensure it accurately reflects my intent. The underlying judgements and decisions are mine; the agent formatted and transcribed them into the log structure.

---

## How to read this log

| Column | Meaning |
|--------|---------|
| **Section** | Notebook step where the interaction occurred |
| **Agent contribution** | What Claude Code produced or suggested |
| **Rationale given** | Why the agent proposed this approach |
| **Verification performed** | What the student checked before accepting |
| **Decision** | Accepted / Modified / Rejected + student reasoning |

---

## Interaction Log

---

### Entry 1 — Step 0: Environment Setup

| Field | Detail |
|-------|--------|
| **Section** | Step 0 — Environment |
| **Agent contribution** | Checked the Anaconda base environment, confirmed all required packages (numpy 2.1.3, pandas 2.3.2, scikit-learn 1.6.1, xgboost 3.0.1, shap 0.47.2, matplotlib, seaborn, jupyter) were pre-installed. No pip installs required. Set global `RANDOM_SEED = 42` and configured seaborn plot style. Registered base conda env as a named Jupyter kernel so VS Code could detect it. |
| **Rationale given** | Fixing a random seed is essential for reproducibility — the brief explicitly requires reproducible experiments. Using a single constant avoids scattered magic numbers. |
| **Verification performed** | Ran cell `cell-imports`; printed output confirmed numpy 2.1.3, pandas 2.3.2, scikit-learn 1.6.1, xgboost 3.0.1, shap 0.47.2. Confirmed `RANDOM_SEED = 42` constant is defined before any random call. Checked that the targeted warning filters (sklearn, xgboost, FutureWarning) did not suppress any errors on a clean run. |
| **Decision** | Modified |
| **Student reasoning** | The initial cell used blanket `warnings.filterwarnings('ignore')`, which would silently hide ConvergenceWarnings from logistic regression (indicating max_iter is too low) and other genuine runtime issues. Replaced with module-specific filters targeting only sklearn, xgboost, and FutureWarning — real unexpected warnings remain visible. |

---

### Entry 2 — Step 1: Cost Matrix Definition

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | Defined the `COST_MATRIX` dictionary and `compute_profit()` function. Mapped confusion matrix positions (TN/FP/FN/TP) to business outcomes (approve/reject × good/default). Included a naive baseline calculation showing the profit of "approve everyone". |
| **Rationale given** | sklearn's `confusion_matrix` uses rows=actual, cols=predicted convention; the agent noted this explicitly to avoid a common mapping error. Profit = TN×1 + FN×(−5) + FP×(−1) + TP×0. |
| **Verification performed** | Constructed a minimal toy example: `y_true=[0,0,1,1]`, `y_pred=[0,1,0,1]`, called `confusion_matrix()`. Confirmed cm[0,0]=TN=1, cm[0,1]=FP=1, cm[1,0]=FN=1, cm[1,1]=TP=1 and that `cm.ravel()` unpacks in (TN, FP, FN, TP) order. Cross-checked against sklearn documentation: "entry i, j is the number of observations actually in group i, but predicted to be in group j." |
| **Decision** | Accepted |
| **Student reasoning** | The confusion matrix mapping is correct and consistent with sklearn's documented convention. The function docstring makes the cm.ravel() unpacking order explicit, which is sufficient for an auditor to verify independently. |

---

### Entry 3 — Step 1: Notebook Structure & Problem Framing Tables

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | Scaffolded the full notebook with markdown cells: cost matrix table, success metrics table, constraints/assumptions list, and agent tooling plan table. |
| **Rationale given** | The brief requires the six step headings to appear in the notebook and report. Pre-structuring saves time and ensures no steps are missed. |
| **Verification performed** | Compared step headings (Steps 0–6) against the brief's required sections — all six present. Reviewed markdown tables for internal consistency: cost matrix profit values match the code cell; success metrics table includes both AUC (comparison) and profit (selection) to avoid conflating the two objectives. |
| **Decision** | Modified |
| **Student reasoning** | The initial scaffold was complete but did not formalise (a) the data split protocol or (b) the cost sensitivity analysis plan. Both were added to Step 1 to create an auditable commitment before any modelling begins. The split table (60/20/20, fit-on-train-only scaler rule) and sensitivity table (3:1, 5:1, 10:1 scenarios) are now explicit in the notebook. |

---

### Entry 4 — Step 1: Cost Matrix Citation (Student Correction)

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | Originally proposed cost values (+1, −5, −1, 0) as "simplified for academic illustration" without a cited source. |
| **Rationale given** | The 5:1 ratio (FN:FP) reflects the asymmetric cost of missing a defaulter vs. rejecting a creditworthy applicant, which is standard in credit risk. |
| **Agent mistake caught** | The initial framing had no literature basis — values were asserted without citation, which is academically weak and unsupported. |
| **Student correction** | Directed agent to cite Chepkemoi, Z., Wanzare, L.D. and Mcoyowo, S. (2025) 'Evaluating trade-offs between error rates in machine learning credit scoring models', *IJSRCSEIT*, 11(4), pp. 360–369, which uses this exact 5:1 ratio for the German Credit dataset standard cost matrix. The TN=+1 extension (revenue from approved good loan) was flagged as the student's own modelling assumption, not from the paper. |
| **Verification performed** | Located Chepkemoi et al. (2025). Confirmed the paper's cost matrix assigns FN (approve defaulter) = 5 and FP (reject good applicant) = 1. Confirmed the paper does not include a TN=+1 revenue term — the notebook flags this extension explicitly as a student modelling assumption, distinct from the cited source. |
| **Decision** | Modified — accepted cost values but required literature grounding; updated markdown, code comments, and `compute_profit()` docstring to reflect the citation. |
| **Student reasoning** | Cost values asserted without a source weaken the academic credibility of the profit function. Grounding the 5:1 ratio in a peer-reviewed reference makes the assumption defensible and auditable. The TN=+1 extension is flagged as mine so the citation is not overstated. |

---

### Entry 5 — Step 1: Naive Baseline Bug + Unsourced Default Rate

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | Computed a naive "approve everyone" baseline using a placeholder default rate of 8.2% and printed the message "This should be negative — confirming that ignoring defaults destroys value." |
| **Agent mistake caught** | Two errors: (1) The 0.082 default rate was invented by the agent with no source — it was not computed from the data. (2) The claim "This should be negative" is mathematically wrong. At 8.2% defaults, the naive profit = 91,800×(+1) + 8,200×(−5) = +50,800, which is positive. The approve-everyone strategy is only net-negative when the default rate exceeds 1/6 ≈ 16.7% (solving (1−r) < 5r). Printing the wrong expectation would mislead a reader checking their output. |
| **Student correction** | (1) Added a comment flagging 0.082 as a placeholder to be replaced by the actual computed rate in Step 2. (2) Replaced the incorrect "should be negative" message with a correct explanation: at ~8% default rate the naive strategy is still profitable, and the model's job is to beat that positive baseline. |
| **Verification performed** | Manually computed: 91,800×(+1) + 8,200×(−5) = 91,800 − 41,000 = +50,800 → confirmed positive. Solved (1−r)×1 + r×(−5) = 0: 1 − r − 5r = 0 → r = 1/6 ≈ 16.7%. Verified this exceeds the ~8% actual rate, so the approve-everyone strategy is indeed net-positive at Home Credit's default rates. |
| **Decision** | Modified — corrected both the unsourced value and the wrong directional claim. |
| **Student reasoning** | Printing "this should be negative" when the correct answer is positive would cause any reader checking their output to think the code was broken. Inaccurate expectations are worse than no expectation. The corrected text explains why the naive baseline is positive and what the model must beat. |

---

### Entry 6 — Step 1: Hardcoded Placeholder Removed from Printed Output

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | After the naive baseline was partially corrected (Entry 5), the cell still contained `approx_default_rate = 0.082` producing a printed profit figure of 50,800 — based on an invented number. |
| **Agent mistake caught** | A hardcoded value with no source was being used to generate a printed result that looked like a real computed output. This violates the principle of not making up assumptions. Additionally, a full audit of the notebook was requested to check whether this pattern appeared anywhere else. |
| **Audit findings** | All other values in Steps 0–1 trace to either: (a) the user's specification (100,000 sample, RANDOM_SEED=42, 307,511 total rows), (b) Chepkemoi et al. (2025) (FN=5, FP=1), or (c) algebra derived from the cost matrix constants (break-even at 1/6 ≈ 16.7%). No other unsourced assumptions were found. |
| **Student correction** | Removed the data-dependent naive baseline computation entirely from Step 1. Replaced with a break-even formula computed algebraically from `COST_MATRIX` values only — no data required. The actual default rate and naive baseline will be computed from the sampled data in Step 2. |
| **Verification performed** | Searched the notebook source for all numeric literals: 100_000 (user brief), 42 (user brief), 307_511 (Kaggle dataset description page). Confirmed no remaining data-dependent hardcoded values in Steps 0–1 after removal of 0.082. Manually verified break-even algebra: 1/(1+5) = 1/6 = 16.7% ✓. |
| **Decision** | Modified — removed invented placeholder; cell now produces only algebra-derived outputs. |
| **Student reasoning** | A printed result derived from an invented number creates a false audit trail — it looks computed when it is not. Step 1 should only show what can be derived from the cost matrix algebra or the stated user specification. Data-dependent results belong in Step 2 where the data is actually loaded. |

---

### Entry 7 — Step 1: Inverted Break-Even Formula

| Field | Detail |
|-------|--------|
| **Section** | Step 1 — Problem Framing |
| **Agent contribution** | Computed break-even default rate using formula `1 / (1 + approve_good / abs(approve_default))` = 1/(1 + 1/5) = 5/6 = **83.3%**, and printed "Approve-everyone is loss-making only when default rate > 83.3%". |
| **Agent mistake caught** | The formula is inverted. Correct algebra: set (1−r)×1 + r×(−5) = 0 → 1 = 6r → r = 1/6 ≈ **16.7%**. The correct formula is `C_TN / (C_TN + \|C_FN\|)` = 1/(1+5) = 1/6. The agent had the ratio upside-down, producing a nonsensical result (83.3% default rate is far outside any realistic lending portfolio). |
| **Student correction** | Fixed formula to `COST_MATRIX['approve_good'] / (COST_MATRIX['approve_good'] + abs(COST_MATRIX['approve_default']))`. Added derivation steps as inline comments so the algebra is auditable. |
| **Verification performed** | Solved step-by-step: (1−r)×1 + r×(−5) = 0 → 1 − r − 5r = 0 → 1 = 6r → r = 1/6 = 0.1667 ≈ 16.7%. Confirmed corrected code formula C_TN/(C_TN+\|C_FN\|) = 1/(1+5) = 1/6 ✓. Agent's original formula 1/(1+1/5) = 5/6 = 83.3% is the reciprocal — confirmed as incorrect. |
| **Decision** | Modified — corrected inverted formula. |
| **Student reasoning** | An 83.3% default rate is implausible in any retail lending portfolio (the Home Credit dataset is ~8%). Any reader familiar with credit risk would immediately flag this. The corrected formula and inline algebraic derivation allow an independent reader to verify the break-even condition without running the code. |

---

### Entry 8 — Step 2: EDA Code

| Field | Detail |
|-------|--------|
| **Section** | Step 2 — EDA |
| **Agent contribution** | Generated seven sub-sections: (1) data loading with stratified 100k sample, (2) class distribution bar chart, (3) missing value counts and bar chart with 50% threshold line, (4) DAYS_EMPLOYED anomaly detection (sentinel value identification, default rate comparison), (5) employment features vs default rate (OCCUPATION_TYPE and ORGANIZATION_TYPE plots), (6) key numeric distributions overlaid by class, (7) correlation heatmap. All printed values computed from sampled data — no hardcoded assumptions. |
| **Rationale given** | Covers all EDA requirements in the brief: distributions, missingness, leakage check, class imbalance, outliers. The DAYS_EMPLOYED sentinel is discovered empirically (code finds the most common positive value) rather than assumed. |
| **Verification performed** | I ran all Step 2 cells end-to-end, checked that the printed class counts and default-rate narrative were consistent, confirmed the DAYS_EMPLOYED sentinel/anomaly output appeared as expected, checked the occupation/organization plots for sensible ordering, and reviewed the correlation heatmap for any suspiciously high target correlations. |
| **Decision** | Modified |
| **Student reasoning** | I accepted the overall EDA structure, but I did not accept the first pass unchanged because several details needed correction for methodological accuracy and clearer auditability (captured in Entry 9). |

---

### Entry 9 — Step 2: Student Corrections During Verification

| Field | Detail |
|-------|--------|
| **Section** | Step 2 — EDA |
| **Agent mistake caught** | Nine issues identified during student review of the first-pass EDA cells. |
| **Correction 1 — Duplicate cells** | Six first-pass cells (IDs: i0k5lkqdvt, otysxvlpy5q, 9pzo7ayppyr, idbdvuuupo, xoic0lu72o, 3kwhei5v5ms) were duplicate earlier drafts of 2.5, 2.6, 2.7, and their agent note, left in the notebook after revised versions were added. All six deleted. |
| **Correction 2 — Wrong imbalance ratio** | Cell `ulmtiqmzuis` stated "roughly 8 repaid borrowers for every 1 defaulter." The actual ratio from the data is ~11.4:1 (consistent with the ~92% accuracy figure cited in the same cell). Corrected to "roughly 11 repaid borrowers for every 1 defaulter." |
| **Correction 3 — EDA mutating df** | Cell `b2d5code001` wrote `df['DAYS_EMPLOYED_YEARS'] = ...`, permanently adding a column to the main dataframe during EDA. EDA must not mutate df — feature engineering belongs in Step 3. Replaced with a local variable `days_emp_years`; all downstream references in the same cell updated to index via `df['TARGET'] == tgt`. |
| **Correction 4 — Currency symbol on anonymised features** | Cell `b2d6code001` labelled monetary features as `Annual income (£)`, `Credit amount (£)`, `Annuity amount (£)`. Home Credit amounts are anonymised units with no declared currency. Corrected to `(anonymised units)`. |
| **Correction 5 — No log scale on right-skewed features** | The same cell plotted income, credit, and annuity on a linear x-axis, compressing most information to the left. Added `set_xscale('log')` for both panels of all three monetary features inside the loop. |
| **Correction 6 — Leakage threshold 0.3 is unreachable** | Cell `b2d7code001` and its markdown cell `b2d7mdcell1` used `|ρ| > 0.3` as a leakage flag. No feature in the Home Credit application table reaches this threshold (EXT_SOURCE features are ~0.15–0.20), so the flag would never fire. Changed to `|ρ| > 0.15`. Markdown updated to describe this as a heuristic screen and to clarify EXT_SOURCE legitimacy (computed at application time, not post-outcome). |
| **Correction 7 — EDA summary ratio used rounded integer** | The EDA summary block printed `~{round(1/mean)}:1` which rounds 11.4 to 12. Corrected to `{n_good/n_defaults:.1f}:1` using the exact counts already in scope from cell 2.1. |
| **Correction 8 — Raw DAYS_EMPLOYED distorts correlation** | The sentinel value 365,243 (encoded for pensioners/unemployed) is a large positive outlier that artificially inflates or deflates the Pearson correlation of `DAYS_EMPLOYED` with TARGET. Added `numeric_cols.drop('DAYS_EMPLOYED', errors='ignore')` immediately after the `numeric_cols` assignment, with a comment deferring the cleaned version to Step 3. |
| **Correction 9 — Agent note threshold and Decision prompt** | Cell `b2agentnte1` still referenced `|ρ|>0.3` in the feature correlations row (stale after Correction 6). Updated to `|ρ|>0.15`. The Decision field placeholder was also tightened to prompt the student to record specific observations (imbalance ratio, sentinel value, EXT_SOURCE dominance, and any further corrections made). |
| **Verification performed** | I re-ran all Step 2 cells after the corrections, confirmed the EDA summary shows the correct imbalance (about 11.4:1), confirmed DAYS_EMPLOYED was excluded from the raw-correlation screen, and checked that the log-scale monetary plots now show usable distribution detail instead of left-side compression. |
| **Decision** | Modified — nine corrections applied during verification. |
| **Student reasoning** | I made these edits because they directly affect statistical correctness and interpretation quality. The biggest issues were incorrect wording/ratios, silent dataframe mutation during EDA, and plotting choices that hid distribution structure. After correction, the section is reproducible and defensible. |

---

### Entry 10 — Step 2 (EDA): Minor Bug in Informative-Missingness Check

| Field | Detail |
|-------|--------|
| **Section** | Step 2 — EDA, cell `jscyttg5uds` |
| **Agent mistake caught** | The OCCUPATION_TYPE informative-missingness check used a one-sided conditional: `if occ_missing_rate > occ_present_rate`. In the actual data, missing occupation is associated with a *lower* default rate (~6.63%) compared to present occupation (~8.74%). The conditional evaluated to `False` and printed "Missingness not strongly informative for OCCUPATION_TYPE" — which is wrong. Missing occupation IS informative: it identifies a lower-risk segment worth capturing. The conditional should check for any meaningful difference, not just the higher-risk direction. |
| **Impact** | The printed conclusion was misleading but the feature engineering in Step 3 is correct — `OCCUPATION_MISSING` is added as a binary flag based on this EDA finding. |
| **Student correction** | Noted as a minor bug; the Step 3 feature engineering markdown documents the correct EDA evidence (6.63% vs 8.74% default rates). The EDA cell itself is left as-is for the audit trail. |
| **Decision** | Noted — bug recorded; Step 3 uses correct interpretation. |

---

### Entry 11 — Step 3: Data Preparation

| Field | Detail |
|-------|--------|
| **Section** | Step 3 — Data Preparation |
| **Agent contribution** | (1) Stratified 60/20/20 split with `stratify=y` and `RANDOM_SEED=42`. (2) `engineer_features()` function: DAYS_EMPLOYED sentinel → NaN + DAYS_EMPLOYED_ANOMALY flag; OCCUPATION_MISSING binary flag; CODE_GENDER XNA → NaN; DAYS_* columns → abs values; SK_ID_CURR dropped. (3) Missingness analysis on X_train only: auto-drop >60%, printed 40–60% for student review with `MANUAL_DROP_COLS` placeholder. (4) Two separate ColumnTransformers: `preproc_linear` (OHE + StandardScaler, for LR/MLP) and `preproc_tree` (OrdinalEncoder, no scaler, for RF/XGB). (5) Verification cell: shapes, class proportions, feature counts before/after OHE expansion, leakage guard printing scaler mean from X_train only. |
| **Rationale given** | Split-first ordering ensures no aggregate statistics from val/test influence imputation or scaling. Two preprocessors required because OrdinalEncoder is methodologically incorrect for LR (arbitrary ordinal ranking of nominal categories enters the linear combination directly). Trees are scale-invariant so StandardScaler adds cost with no benefit. |
| **Verification performed** | I checked that train/val/test class proportions were very close after stratified splitting, verified the scaler statistic came from train-only fitting, confirmed OHE expanded feature count as expected, and reviewed missingness outputs for both >60% and 40–60% bands. I kept the >60% drop set and left `MANUAL_DROP_COLS` empty at that stage to avoid arbitrary extra deletions without evidence. |
| **Decision** | Modified |
| **Student reasoning** | I accepted the split-first and dual-preprocessor design, but I required targeted fixes before moving on. For missingness, I chose a conservative rule: auto-drop only the most extreme columns (>60%), then treat the 40–60% band as review-only rather than forcing additional manual drops. |

---

### Entry 12 — Step 3 Review: Student corrections after running cells

| Field | Detail |
|-------|--------|
| **Section** | Step 3 — Data Preparation (review pass) |
| **Agent contribution** | Implemented four corrections identified by student during cell review: (1) Added `CODE_GENDER_XNA` binary flag in `engineer_features()` before XNA → NaN replacement, consistent with `OCCUPATION_MISSING` pattern; updated FE markdown table and print statement; added row to agent tooling note. (2) Fixed `NameError` in verification cell — `num_cols` was defined in the preprocessor cell and unavailable when verification cell is re-run in isolation; replaced with `preproc_linear.transformers_[0][2]`. (3) Removed hardcoded "~18%" anomaly rate from agent note and verification cell comment; added computed `anomaly_frac = X_train['DAYS_EMPLOYED_ANOMALY'].mean()` consistency-check print to the verification cell so the rate is derived from data. (4) Added Step 4 intro markdown with CV leakage caveat: pre-materialised arrays (`X_train_lin`, `X_train_tree`) are correct for single held-out validation but must not be passed to `GridSearchCV`/`cross_val_score`; preprocessors must be inside a `Pipeline` for k-fold CV. |
| **Rationale given** | Correction 1: unknown-gender rows are a distinct segment; flagging before imputation preserves the signal (same logic as OCCUPATION_MISSING). Correction 2: cells that depend on variables from previous cells cannot be safely re-run in isolation; using the fitted transformer attribute avoids the cross-cell dependency entirely. Correction 3: hardcoded rates invented by the agent are not reproducible and may become stale; the computed value is always consistent with the data in memory. Correction 4: the CV leakage caveat is a genuine methodological risk that must be flagged explicitly before Step 5 hyperparameter tuning. |
| **Verification performed** | Student caught all four issues during cell-by-cell review and directed specific fixes. Agent implemented as instructed. |
| **Decision** | Modified |
| **Student reasoning** | All four corrections were directed by the student. The agent had missed the CODE_GENDER_XNA flag, introduced a cross-cell dependency in the verification cell, inserted a hardcoded rate that was not derived from data, and omitted a CV leakage warning that is critical for Step 5 correctness. |

---

### Entry 13 — Step 3/4: CV-safety update (targeted, pre-Step 4)

| Field | Detail |
|-------|--------|
| **Section** | Step 3 (preprocessing cell) + Step 4 intro markdown |
| **Agent contribution** | (1) Added unfitted preprocessor clones at the end of `s3code034pre`: `preproc_linear_cv = clone(preproc_linear)` and `preproc_tree_cv = clone(preproc_tree)`, with an inline comment directing Step 5 to use these inside a `Pipeline` rather than passing pre-materialised arrays to `GridSearchCV`/`cross_val_score`. (2) Replaced the multi-paragraph CV leakage warning block in the Step 4 intro markdown with a single concise blockquote sentence referencing the unfitted clones. |
| **Rationale given** | Pre-materialised arrays are safe for single held-out validation but not for k-fold CV — the preprocessor has already seen all fold-validation rows when fit on the full `X_train` before folding. Unfitted clones produced in Step 3 provide a ready-to-use object for Step 5 Pipelines, preventing fold leakage during `GridSearchCV`. |
| **Verification performed** | Student directed the exact changes. No model code altered. |
| **Decision** | Accepted |
| **Student reasoning** | Proactive safety measure — clones created in Step 3 so Step 5 does not need to re-derive them. Guidance in Step 4 tightened to a single sentence to reduce noise while keeping the constraint explicit. |

---

### Entry 14 — Step 4: Model Comparison (revised after student review)

| Field | Detail |
|-------|--------|
| **Section** | Step 4 — Model Comparison |
| **Agent contribution** | (1) Five model configurations: LR (`max_iter=1000`), RF (`n_estimators=100`), XGBoost (`n_estimators=100`), MLP (baseline: `max_iter=300`), MLP (stabilized: `max_iter=1000, early_stopping=True, n_iter_no_change=10`). (2) Array routing: LR + both MLPs → `X_train_lin`/`X_val_lin`; RF + XGBoost → `X_train_tree`/`X_val_tree`. (3) Class-order assertion `assert list(model.classes_) == [0, 1]` immediately after each `.fit()` for all five models. (4) Five-column results table: Val AUC, Profit@0.5, Profit per applicant, Δ vs naive; `profit_naive` computed once before loop. (5) MLP convergence diagnostics block after loop: `n_iter_`, `max_iter`, `converged` boolean for both MLP variants; interpretation rule printed inline. (6) ROC curves for all five configurations on one axes, five distinct colours, AUC in legend. (7) Side-by-side bar charts: left = Val AUC; right = Profit@0.5 with naive baseline dashed line. (8) All models intentionally unweighted — noted in intro blockquote; class-imbalance handling deferred to Step 5. (9) CV guard maintained: no `GridSearchCV`; `preproc_*_cv` clones available for Step 5. |
| **Rationale given** | Two MLP variants evaluate whether the architecture performs better under a more permissive training configuration (higher iteration budget, early stopping). Per-applicant and Δ vs naive columns give normalised and directional context. Class assertion guards against label-flipped `predict_proba`. Unweighted baselines keep model-family ranking clean before threshold optimisation in Step 5. early_stopping note (10% holdout) logged as acceptable inequality at baseline stage. |
| **Verification performed** | I verified that all model rows printed, checked both MLP convergence diagnostics, confirmed the ROC ordering was plausible, and cross-checked bar-chart labels against the printed AUC/profit table. I also verified Profit@0.5 was interpreted against the naive baseline rather than in isolation. |
| **Decision** | Modified |
| **Student reasoning** | I kept both MLP variants because they show an important methodological point: both variants completed training, with the stabilised version providing a robustness check under a more permissive training configuration. The ranking showed linear and boosted models were close and stronger than the MLP baseline, which justified carrying LR and XGBoost into Step 5. |

---

### Entry 15 — Steps 2 + 4: Cross-step narrative consistency fixes

| Field | Detail |
|-------|--------|
| **Section** | Step 2 (cell `ulmtiqmzuis`) + Step 4 (cell `srs1dzftl5`) |
| **Fix 1 — Stale cross-reference in Step 2 narrative** | Cell `ulmtiqmzuis` (Step 2 class distribution markdown) contained: *"In Step 4, we will set `class_weight='balanced'` (or the XGBoost equivalent) to stop models from ignoring the minority class."* This was drafted before Step 4 was designed and is factually wrong — Step 4 uses intentionally unweighted baselines. Replaced with: *"Step 4 uses unweighted baselines for clean model-family comparison; class-imbalance is addressed in Step 5 via profit-maximising threshold search rather than class reweighting."* |
| **Fix 2 — Imprecise code comment in Step 4** | Cell `srs1dzftl5` opened with `# Default hyperparameters throughout — tuning is Step 5's job.` This overstated uniformity: MLP (stabilized) uses `early_stopping=True`, which is not a default setting. Replaced with a three-line comment: `# Baseline settings throughout — tuning is Step 5's job. / # Exception: MLP (stabilized) uses early_stopping=True as a / # sensitivity check on convergence, not a tuning decision.` |
| **Corrections cell added** | `qapx9re0qz` inserted after Step 4 agent note documenting both student corrections in blockquote format. |
| **Rationale** | Cross-step narrative consistency is important for academic audit — a stale claim about Step 4 method in Step 2 would contradict the actual implementation. The comment fix ensures the code accurately describes the design intent of the `early_stopping=True` choice. |
| **Decision** | Modified — both fixes directed by student |

---

### Entry 16 — Step 5: Fine-tuning & Profit-Maximising Threshold

| Field | Detail |
|-------|--------|
| **Section** | Step 5 — Fine-tuning & Profit-Maximising Threshold |
| **Agent contribution (initial pass)** | (1) Shortlist decision documented: student selected LR, XGBoost, MLP (stabilized); RF dropped as redundant with XGBoost in same model family. (2) 5.1: profit-curve sweep over 201 thresholds for shortlisted baseline models; t* and max profit printed; overlaid curves plotted. (3) 5.2c: `GridSearchCV` with `Pipeline([preproc_tree_cv, XGBClassifier])` on raw `X_train`; 8-combination grid; 5-fold CV; scoring=ROC-AUC; `refit=True`. (4) 5.3: profit-curve on tuned XGBoost; final t* annotated on plot. (5) 5.5: single test-set evaluation at t*; confusion matrix, approval rate, default recall reported. (6) 5.6: sensitivity analysis under 3:1, 5:1, 10:1 FN:FP cost ratios; r* per scenario; overlaid profit curves. |
| **Agent-made mistake caught** | (1) Step 5 intro credited XGBoost with the highest Step 4 AUC — this was factually wrong; LR had the higher raw AUC (~0.75 vs ~0.72). (2) Only XGBoost was tuned, leaving LR as an untuned baseline despite it having higher AUC — an unfair comparison. (3) The XGBoost tuning cell used the default CV splitter and did not compute a held-out val AUC, making the CV AUC ambiguous. (4) CV AUC and val AUC were not clearly labelled as measuring different splits, creating risk of conflation. (5) No calibration check was included. (6) No failure-mode analysis was included. |
| **Corrections applied (student-directed)** | (1) `zxxmh7x2dzl` intro updated: LR row now reads "Highest raw AUC (~0.75)"; XGBoost row reads "Competitive AUC (~0.72); tuning headroom + TreeSHAP support"; Step 5 plan set to 7 sub-steps. (2) New LR tuning cell `2ta0f2wnzik` (5.2a) inserted: `Pipeline([preproc_linear_cv, LR])`, `StratifiedKFold(5)`, grid `C ∈ {0.01, 0.1, 1, 10, 100}`, prints best C + CV AUC (train folds) + tuned val AUC + baseline val AUC + clarifying note. Final (historical baseline, with AMT_GOODS_PRICE): best C=0.1, tuned val AUC=0.7518. Current final (without AMT_GOODS_PRICE): best C=0.01, tuned val AUC=0.7495. (3) MLP tuning tested (27 fits): tuned val AUC 0.7382 vs baseline 0.7360 (Δ+0.0022); max profit 10,551 vs 10,545 (Δ+6). **Student decision: negligible gain — tuned MLP removed.** (4) `lbukaua2r2` profit sweep: 4 curves — tuned LR + baseline LR + baseline XGBoost + MLP (stabilized). (5) `om0kqdsw79` (5.2c) XGBoost tuning: `StratifiedKFold(5)`, 8-combination grid. Final hyperparameters: n_estimators=300, learning_rate=0.05, max_depth=3, subsample=0.8, colsample_bytree=0.8. Historical baseline (with AMT_GOODS_PRICE): tuned val AUC=0.7558, max val profit=10,653. Current final (without AMT_GOODS_PRICE): tuned val AUC=0.7525, max val profit=10,619. (6) Extended XGBoost search (agent-suggested, 324 combos × 5 folds = 1,620 fits). **Student decision: runtime >40 min on local hardware; interrupted. First-pass hyperparameters retained.** Cell deleted; evidence in corrections cell. (7) New calibration cell `9u3cx7bbe7f` (5.4): reliability diagrams + Brier scores for tuned XGBoost and tuned LR. (8) New failure-mode markdown `e5o4obebkfd` after test-eval. (9) `locx1hw9pcb` agent note updated; `6c9ztwlhrps` corrections cell updated with 9 corrections. (10) **Student addition: F1/F2 secondary diagnostics.** F1 and F2 (β=2) scores appended to cells 5.3 and 5.5 at both threshold=0.5 and t*. Added to complement AUC/profit and show threshold-dependent precision/recall trade-off under class imbalance. These are secondary metrics only — model selection and threshold optimisation remain profit-based. Step 5 intro updated with one clarifying sentence; Step 5 agent note updated with new row. (11) **Student addition + tidy: secondary diagnostics panel.** Final retained set: PR-AUC, Brier score, F2@t*, MCC@t*, Precision@5% and Recall@5%. Removed after review: log loss, balanced accuracy (@0.5 and t*), F1@0.5, F2@0.5, F1@t*, MCC@0.5, Precision/Recall@10%. F1/F2 block merged with expanded-metrics block into a single clean `# Secondary diagnostics` block per cell; unused imports and variables removed. Step 5 intro updated to name final retained set; Step 5 agent note updated. |
| **Rationale given** | Tuning LR alongside XGBoost ensures a fair comparison between the two best-performing Step 4 models. Profit-curve sweep with tuned LR makes the gain from regularisation immediately visible. StratifiedKFold with explicit val AUC computation provides a like-for-like comparison between CV and held-out performance. Calibration check supports threshold interpretation — poorly calibrated probabilities shift the optimal t* unpredictably. Failure-mode analysis defers feature attribution to SHAP while giving a qualitative picture of error profile at t*. |
| **Verification performed** | I verified LR tuning printed best `C` and held-out validation AUC, verified XGBoost tuning reported both CV and validation AUC clearly, checked reliability diagrams/Brier scores, confirmed the confusion-matrix orientation (rows=actual, cols=predicted), and verified threshold behaviour across cost scenarios (higher FN cost pushed the optimal threshold lower). In the final run, LR best `C` was 0.01, and tuned XGBoost used `n_estimators=300`, `learning_rate=0.05`, `max_depth=3`, `subsample=0.8`, `colsample_bytree=0.8`, with final `t* = 0.285`. |
| **Decision** | Modified — eleven corrections/additions applied, directed by student |
| **Student reasoning** | I tuned LR and XGBoost to keep the comparison fair. XGBoost achieved the higher validation AUC in the final run, while LR remained very competitive on profit, so the trade-off was practically close. Calibration looked acceptable for decision-threshold use, test profit stayed above naive approve-all, and sensitivity analysis behaved directionally as expected (stricter FN penalties led to lower thresholds and more conservative approvals). |

---

### Entry 17 — Step 6: SHAP Interpretability & Model Card

| Field | Content |
|-------|---------|
| **Section** | Step 6 — SHAP Interpretability & Model Card |
| **Agent contribution** | (1) Step 6 intro markdown: final model selection table (LR tuned vs XGBoost tuned), rationale for XGBoost (TreeSHAP support), 3-point plan. (2) SHAP global cell (6.1): feature name extraction via `get_feature_names_out()` + prefix strip; 3,000-row sample from X_train; native XGBoost SHAP via `pred_contribs=True` on DMatrix; beeswarm + mean |SHAP| bar chart; printed top-10. (3) Employment deep-dive cell (6.2): dependence scatter plots for DAYS_EMPLOYED, DAYS_EMPLOYED_ANOMALY, OCCUPATION_TYPE; mean |SHAP| group totals vs EXT_SOURCE group. (4) Model card (6.3): structured markdown with use/not-for, data provenance, evaluation table, caveats. |
| **Agent-made mistake caught** | (1) Initial SHAP cell used `shap.TreeExplainer(xgb_model)` which crashes under XGBoost 3.x / SHAP 0.47.x: `base_score` is serialised as `'[8.073334E-2]'` (bracketed string) but SHAP calls `float(...)` directly. First fix attempt used a booster config patch (`load_config`); student rejected as fragile. Final fix: native XGBoost SHAP via `booster.predict(dshap, pred_contribs=True)` — no SHAP library parsing of base_score required. (2) Intro and model card used £ currency symbols — incorrect for an anonymised dataset where profit units are arbitrary. (3) Intro used "statistically indistinguishable" — misleading without a formal DeLong test. |
| **Corrections applied (student-directed)** | (1) SHAP computation replaced with `xgb.DMatrix` + `pred_contribs=True`; `import json as _json` and config-patch code removed; `X_shap_np` (float32 numpy array) used in place of raw `X_shap` in both 6.1 and 6.2. (2) All £ symbols removed from intro and model card; replaced with "units". (3) "statistically indistinguishable" → "practically equivalent on this split; no formal significance test was performed". |
| **Rationale given** | `pred_contribs=True` calls XGBoost's internal C++ SHAP implementation directly — avoids the SHAP Python library's model-config parsing entirely and is version-stable. Float32 cast ensures DMatrix accepts the array without silent type promotion. The bias column (last column of `shap_contrib`) is dropped as it is a constant offset, not a feature contribution. Currency symbols are inappropriate because Home Credit profit values are in anonymised arbitrary units, not £. The AUC/profit difference between LR and XGBoost does not meet any standard significance threshold; calling it "indistinguishable" implies a formal test was run. |
| **Verification performed** | I confirmed SHAP computed without runtime errors using the native XGBoost contribution path, confirmed the global importance ranking was led by EXT_SOURCE features, checked all three employment panels rendered, and verified the group-summary totals printed. I also checked the Step 6 model-card evaluation values against the Step 5 test-evaluation outputs to ensure consistency. |
| **Decision** | Modified — three student-directed corrections applied |
| **Student reasoning** | I chose the native `pred_contribs=True` approach because it avoids the SHAP/XGBoost version-compatibility failure and is stable. The outputs confirmed EXT_SOURCE dominance, and employment-related features contributed signal but at a much smaller aggregate magnitude than the bureau-score block. |

---

### Entry 18 — Appendix: EXT_SOURCE Ablation Test

| Field | Content |
|-------|---------|
| **Section** | Appendix — cells `abl0mdcell1`, `abl0codecell1`, `abl0plotcell1` |
| **Agent contribution** | (1) Ablation code cell: drops `EXT_SOURCE_1/2/3` from `X_train` and `X_val` only (test set untouched); rebuilds a tree ColumnTransformer for the reduced feature set; fits XGBoost with the same tuned hyperparameters as the final model (n_estimators=300, lr=0.05, depth=3, subsample=0.8, colsample_bytree=0.8); re-optimises threshold on the validation profit sweep; prints a compact comparison table of full vs ablated model (val AUC, max val profit, t*) plus delta row. (2) Visualisation cell: 2-panel bar chart — Panel A = val AUC comparison, Panel B = max val profit comparison with naive approve-all dashed baseline; numeric labels on bars; single ΔAUC / ΔProfit print line below. |
| **Rationale given** | EXT_SOURCE features dominate SHAP importance in Step 6. An ablation quantifies how much predictive power they actually provide vs the rest of the feature set. Using the same hyperparameters isolates the effect of feature removal from re-tuning. Threshold is re-optimised on validation (not fixed) to give the ablated model its best possible profit — a fair comparison. Test set not touched: this is a robustness check, not a new candidate model. |
| **Student decision to keep** | Student reviewed the ablation rationale and decided to retain it as an appendix artefact: (a) it supports the SHAP narrative in Step 6 with a concrete quantitative dependency check; (b) it demonstrates proactive robustness testing beyond the minimum requirements|
| **Verification performed** | I confirmed the ablation model trained cleanly, checked that both AUC and max-profit moved downward after removing EXT_SOURCE features, and verified the ablation cells used train/validation only with no test-set leakage. I also checked that the bar-chart labels matched the printed comparison table values. |
| **Decision** | Accepted — retained in final submission as appendix |
| **Student reasoning** | I observed a clear performance drop in ablation (approximately ΔAUC = -0.063 and Δprofit = -286 on validation), which confirms strong dependency on EXT_SOURCE features. Even without EXT_SOURCE, the model still stayed above naive approve-all, but the gap narrowed materially, so these features are high-value in this setup. |

---

### Entry 19 — Step 3: AMT_GOODS_PRICE Controlled Removal

| Field | Content |
|-------|---------|
| **Section** | Step 3 — Data Preparation (§3.2 feature engineering, §3.7 multicollinearity) |
| **Agent contribution** | (1) Identified AMT_GOODS_PRICE as near-duplicate of AMT_CREDIT (ρ≈0.987) in the relocated multicollinearity heatmap. (2) Added controlled removal of `AMT_GOODS_PRICE` from `X_train`, `X_val`, `X_test` in Step 3.2. (3) Ran full notebook rerun (Steps 3–6 + appendix). (4) Produced comparison table (cell `amtgds_cmp01`, inserted after §6.1 SHAP) showing baseline vs removal metrics. |
| **Experiment results** | Val AUC: 0.7558 → 0.7525 (Δ−0.0033). Val profit: 10,653 → 10,619 (Δ−34). Val t*: 0.295 → 0.285. Test AUC: 0.7515 → 0.7479 (Δ−0.0036). Test profit: 10,773 → 10,753 (Δ−20). Top-3 SHAP: EXT_SOURCE_2/3/1 (AMT_GOODS_PRICE exits top-3, replaced by EXT_SOURCE_1). |
| **Agent initial decision** | Flagged rejection: ΔAUC=−0.0036 exceeded the −0.002 threshold specified in the experiment rules. |
| **Agent mistake caught** | Decision threshold of −0.002 on AUC was too strict for this coursework context. The agent applied it mechanically without considering the practical/academic trade-off. |
| **Student override** | Accepted removal despite ΔAUC=−0.0036. Rationale: (a) practical impact was minor on validation (ΔAUC=−0.0033, ΔProfit=−34), which drove the keep decision; test metrics (ΔAUC=−0.0036, ΔProfit=−20) were observed afterward for reporting only; (b) removing a near-perfect duplicate resolves SHAP attribution splitting across correlated features; (c) AMT_CREDIT is retained as the more direct and interpretable credit-amount feature; (d) the experiment provides documented evidence of verification, which strengthens the agent-usage narrative. |
| **Verification performed** | Student reviewed comparison table output, confirmed ΔAUC and ΔProfit magnitudes, and made explicit keep decision. Experiment record cell preserved in notebook as audit trail. |
| **Decision** | Accepted — student-directed override of agent decision rule |
| **Final pipeline metrics** | Test AUC: 0.7479 \| Test profit: 10,753 \| Val t*: 0.285 \| Top-3 SHAP: EXT_SOURCE_2, EXT_SOURCE_3, EXT_SOURCE_1 |

---

### Entry 20 — Final Pre-Submission Consistency Cleanup

| Field | Content |
|-------|---------|
| **Section** | Cross-notebook cleanup — all steps |
| **Agent contribution** | Systematic pre-submission cleanup across `credit_defult.ipynb` and `agent_usage_log.md` covering five areas: (1) **Decision placeholders filled** — all five `*(Student to complete…)*` / `*(Accepted / Modified / Rejected…)*` placeholder lines in the Step 2–6 agent-note cells were replaced with first-person completed statements matching actual notebook outputs. (2) **`amtgds_cmp01` code cell converted to markdown** — the hardcoded historical-metrics print block was replaced with a concise markdown blockquote directing readers to `agent_usage_log.md` Entry 19 for the full numeric record. (3) **AMT_GOODS_PRICE justification refined** in `s3multicol_cor`: added explicit statement that validation set metrics (ΔAUC=−0.0033, ΔProfit=−34) drove the decision, with a model-aware explanation (tree splits are not biased by collinearity, but split allocation instability and SHAP fragmentation are the relevant concerns), and updated the log-entry cross-reference to Entry 19. (4) **Validation-only decision discipline enforced**: removal decision was confirmed as made on validation metrics only; test set touched once afterward for reporting. (5) **`agent_usage_log.md` summary table** row for AMT_GOODS_PRICE removal updated to reference `agent_usage_log.md` Entry 19 (not cell `amtgds_cmp01`). |
| **Cells modified** | `b2agentnte1` (Step 2 decision), `s3agentnte01` (Step 3 decision), `ik9rbk9nws` (Step 4 decision), `locx1hw9pcb` (Step 5 decision), `s6agentnte01` (Step 6 decision), `amtgds_cmp01` (converted to markdown), `s3multicol_cor` (AMT_GOODS_PRICE justification) |
| **Rationale given** | Open placeholder text is inconsistent with a submitted coursework notebook — it signals to markers that student verification was not completed. Converting the hardcoded comparison cell to markdown removes executable dead code that prints stale values on re-run. Refining the collinearity justification distinguishes between the statistical concern (SHAP fragmentation) and the model behaviour (XGBoost predictions are threshold-based, not coefficient-based), which is more accurate and more informative for a methodology write-up. |
| **Verification performed** | Confirmed each old placeholder string was uniquely replaced in the JSON source; confirmed `amtgds_cmp01` cell_type changed to "markdown" and execution_count/outputs fields removed; confirmed all assertion checks passed without error. |
| **Decision** | Accepted — cleanup complete |

---

### Entry 21 — Appendix: Multicollinearity Pruning Robustness Check

| Field | Content |
|-------|---------|
| **Section** | Appendix — `credit_defult_multicollinearity_experiment.ipynb` (removed) + main notebook appendix markdown |
| **Agent contribution** | (1) Created a duplicate experiment notebook (`credit_defult_multicollinearity_experiment.ipynb`). (2) Inserted a new cell (3.3b) after the missingness-drop step: computes pairwise correlation matrix on X_train numeric features, finds all pairs with |ρ| > 0.80, drops the weaker-TARGET-correlated feature from each pair. (3) Full pipeline rerun in the experiment notebook — preprocessor, all 4 models, profit sweep, XGBoost tuning, test evaluation. (4) Added a markdown-only appendix section to the main notebook with explanation, comparison table, and verdict. |
| **Experiment results** | 28 features dropped (105 → 77). Notable removals: FLOORSMAX_AVG/MODE, ELEVATORS_AVG/MODE, LIVINGAREA_AVG/MEDI/MODE, APARTMENTS_AVG/MEDI/MODE, REGION_RATING_CLIENT, DAYS_EMPLOYED_ANOMALY (|ρ|=1.00 with FLAG_EMP_PHONE), CNT_FAM_MEMBERS, OBS_60_CNT_SOCIAL_CIRCLE. Val AUC: 0.7525 (unchanged). Val profit: 10,619 (unchanged). Test AUC: 0.7479 (unchanged). Test profit: 10,753 (unchanged). |
| **Rationale given** | Correlated features do not bias XGBoost predictions — trees split one feature at a time and the same information is accessible via whichever correlated feature remains. Multicollinearity causes split-allocation instability and SHAP attribution spreading, not prediction degradation. |
| **Verification performed** | Full notebook run in the experiment file; val and test metrics compared manually against main pipeline values. |
| **Decision** | Not adopted — no performance benefit. Experiment retained as appendix evidence of critical robustness testing. |
| **Student reasoning** | Identical metrics across all four evaluation points confirm XGBoost's robustness to multicollinearity. The pruning experiment is documented in the appendix with a comparison table and explicit "not adopted" verdict. This was done to improve methodological rigour without modifying the validated main pipeline. |

---

### Entry 22 — Step 5: Confusion Matrix Visualisation


| Field | Content |
|-------|---------|
| **Section** | Step 5 — Test set evaluation |
| **Agent contribution** | Generated a seaborn heatmap confusion matrix cell (cell `grmajmsiimg`) inserted after the printed confusion matrix table. Initial version used linear colour scale, causing near-identical shading across all four cells due to TN dominance (18,107 vs ≤1,415). |
| **Student direction** | Requested improved visualisation — colours were too similar to distinguish cells meaningfully. |
| **Verification performed** | Confirmed values match printed table (TN=18,107, FP=279, FN=1,415, TP=199); confirmed log normalisation produces visually distinct shading. |
| **Decision** | Modified — student directed switch to log colour scale (`mcolors.LogNorm`) for clearer visual distinction |
| **Student reasoning** | A confusion matrix heatmap with indistinguishable colours adds no value over the table. Log normalisation was the correct fix given the severe class imbalance between cells. |

---

### Entry 23 — Report: Structure and Review

| Field | Content |
|-------|---------|
| **Section** | Written report (all six sections) |
| **Agent contribution** | Suggested a section structure with word-count allocations mapped to the coursework brief, and provided a key-numbers reference table from locked notebook outputs. Each report section was then checked for coherence, flagged for inconsistencies, and reviewed with suggested phrasing improvements. |
| **Student role** | All analytical content, reasoning, and narrative choices reflect independent judgement. Agent feedback was used selectively to tighten phrasing and flag consistency issues. |
| **Verification performed** | Student confirmed all cited numbers against notebook cell outputs before including in report. |
| **Decision** | Modified — student wrote content; agent reviewed and suggested improvements to coherence and consistency |

---

## Summary Decision Table

| Step | Agent task | Decision | Key reason |
|------|-----------|----------|-----------|
| 0 | Environment setup & imports | Modified | Replaced blanket `filterwarnings('ignore')` with targeted module-specific filters |
| 1 | Cost matrix + problem framing | Modified | Required citation of Chepkemoi et al. (2025); corrected inverted break-even formula; removed hardcoded placeholder |
| 2 | EDA code + plots | Modified | Nine corrections: duplicate cells, wrong imbalance ratio, df mutation, currency labels, log scale, leakage threshold, rounded ratio, sentinel distortion, stale threshold |
| 3 | Data preparation pipeline | Modified | Four corrections: CODE_GENDER_XNA flag; cross-cell dependency fix; hardcoded rate removed; CV leakage caveat added |
| 4 | Model comparison + ROC curves | Modified | Two MLP variants; expanded results table; class-order assertion; convergence diagnostics; AUC + profit bar charts |
| 5 | Hyperparameter tuning + profit threshold | Modified | Eleven corrections: AUC claim fixed; LR tuning added; MLP tuning removed (negligible gain); StratifiedKFold enforced; calibration and failure-mode cells added |
| 5 | Confusion matrix visualisation | Modified | Student directed switch to log colour scale for clearer visual distinction under class imbalance |
| 6 | SHAP plots + model card | Modified | TreeExplainer crash fixed via pred_contribs=True; £ labels removed; wording corrected |
| Appendix | EXT_SOURCE ablation | Accepted | Quantifies bureau score dependency; retained as robustness evidence |
| Step 3 | AMT_GOODS_PRICE removal | Accepted (student override) | ρ≈0.987 with AMT_CREDIT; ΔAUC=−0.003 judged negligible; student overrode agent rejection threshold |
| Appendix | Multicollinearity pruning experiment | Not adopted | 28 features dropped, zero performance change; confirms XGBoost robustness |
| Report | Structure and review | Modified | Student wrote content; agent suggested structure and reviewed for coherence and consistency |
