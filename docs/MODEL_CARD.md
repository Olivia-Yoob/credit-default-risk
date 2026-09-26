# Model Card — Home Credit Default Risk Classifier

*Model cards follow the structure proposed by Mitchell et al. (2019). This card documents a
portfolio model and is illustrative, not a production system.*

---

## 1. Model details

| Field | Value |
|---|---|
| **Model name** | Home Credit Default Risk Classifier |
| **Version** | 1.0 |
| **Date** | 2026-06 |
| **Model type** | Gradient-boosted decision trees (XGBoost `XGBClassifier`) |
| **Owner** | AI Risk / Model Risk function |
| **Pipeline** | Notebooks `01`–`05` (data understanding → EDA → preprocessing → modeling → fair-lending audit) |
| **Key hyperparameters** | `n_estimators=400`, `max_depth=5`, `learning_rate=0.05`, `subsample=0.8`, `colsample_bytree=0.8` |
| **Imbalance handling** | None at training (natural 8% base rate retained for calibration); handled at the decision stage via a cost-based threshold |

## 2. Intended use

- **Primary use:** estimate an applicant's probability of default to support a consumer
  credit decision.
- **Intended users:** credit underwriting / risk teams, with **human oversight** of every
  adverse decision.
- **Out-of-scope uses:** fully automated loan denial without human review; any use beyond
  consumer-credit underwriting; use on populations materially different from the training
  distribution.

## 3. Regulatory classification

- **EU AI Act:** **High-Risk** — Annex III §5(b) (creditworthiness evaluation / credit
  scoring). See [EU_AI_ACT_COMPLIANCE.md](EU_AI_ACT_COMPLIANCE.md).
- **US Fair Lending:** subject to ECOA / Regulation B. See
  [FAIR_LENDING_AUDIT.md](FAIR_LENDING_AUDIT.md).

## 4. Training data

- **Source:** Kaggle *Home Credit Default Risk* (10 tables, ~2.5 GB).
- **Main table:** `application_train` — 307,511 applicants × 122 raw columns.
- **Label `TARGET`:** 1 = payment difficulties (default), 0 = repaid. **Default rate 8.07%**
  (≈11.4 : 1 imbalance).
- **Feature set:** **96 features** — 59 from the cleaned main table (incl. 4 engineered
  affordability ratios) + 37 aggregates summarizing the 6 auxiliary history tables.
- **Split:** stratified 80/20 (246,008 train / 61,503 validation), both at 8.07% default.
- **Preprocessing:** median imputation + standardization, **fit on the training split only**
  (leakage-free). Sentinel `DAYS_EMPLOYED = 365243` (18% of rows) → `NaN` + anomaly flag.

## 5. Performance (validation set)

| Metric | Value |
|---|---|
| **PR-AUC (average precision)** | **0.283** (baseline 0.081) |
| **ROC-AUC** | 0.781 |
| **Decision threshold** | **0.10** (cost-optimal, FN:FP = 10:1; ≈ Bayes optimum 0.091) |
| **Recall (defaulters) @ 0.10** | **0.63** (vs 0.04 at the default 0.50) |
| **Precision (defaulters) @ 0.10** | 0.20 |

Accuracy is **not** reported as a headline metric: a "predict everyone repays" baseline
scores 92% while catching zero defaulters. Selection is driven by PR-AUC and recall.

**Top predictive features:** `EXT_SOURCE_2`, `EXT_SOURCE_3`, `CODE_GENDER`, `NAME_INCOME_TYPE`,
`NAME_EDUCATION_TYPE`, `EXT_SOURCE_1`, `PREV_REFUSED_RATE`, `CC_UTILIZATION_mean`.

## 6. Fairness evaluation

Full audit in [FAIR_LENDING_AUDIT.md](FAIR_LENDING_AUDIT.md). Headline findings:

| Protected attribute | Finding | Status |
|---|---|---|
| **Age** (20s) | Approval Disparate-Impact ratio **0.65** | 🔴 Adverse impact (< 0.80) |
| **Education** (Lower secondary) | DI **0.76** | 🔴 Adverse impact (< 0.80) |
| **Gender** | DI 0.83 (passes 4/5ths) but **TPR gap 0.14** | 🟡 Equal-opportunity gap |

**Mitigation validated:** removing `CODE_GENDER` changes PR-AUC by only −0.4% (0.283 → 0.279)
while improving gender DI (0.83 → 0.90) and halving the TPR gap (0.14 → 0.07).

## 7. Limitations & caveats

- **Static dataset:** no temporal validation; real deployment requires out-of-time testing
  and ongoing drift monitoring (see [DRIFT_MONITORING.md](DRIFT_MONITORING.md)).
- **Label encoding:** categorical features are integer-encoded, which suits the tree model
  but imposes a false order for linear models.
- **`EXT_SOURCE_1` ~56% missing** and median-imputed — a strong but partially-observed signal.
- **Proxy bias:** even with `CODE_GENDER` removed, age- and education-correlated features can
  carry indirect bias; continuous fairness monitoring is required.
- **Small subgroups** (e.g. *Academic degree*, n=40 in validation) have unstable metrics and
  must not be over-interpreted.

## 8. Ethical considerations & recommendation

- **Recommendation:** drop `CODE_GENDER` from the production feature set (near-free fairness
  gain) and run a documented business-necessity review of age/education-correlated features
  before deployment (risks **R1**, **R2** in [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md)).
- **Human oversight** is mandatory for adverse actions; applicants must receive an explainable
  reason (supported by feature-importance / SHAP-style attribution).
