# Fair Lending Audit — Home Credit Default Risk Classifier

**Audit date:** 2026-06 · **Model version:** 1.0 · **Evidence:** notebook
`05_evaluation_fairlending` and figures `results/05_disparate_impact.png`,
`results/05_error_rate_gaps.png`.

---

## 1. Purpose & scope

This audit assesses whether the credit-default model produces **discriminatory outcomes**
against legally protected groups. It is required because the model is **High-Risk under the
EU AI Act** and **subject to US Fair Lending law (ECOA / Regulation B)**.

Scope: the **validation set** (61,503 applicants) scored by the production candidate
(XGBoost) at the cost-optimal threshold **0.10**. The model output `1 = likely default` maps
to a **declined** application; therefore the **favorable outcome is "approved" (prediction 0)**.

## 2. Protected attributes assessed

Registered in notebook `01` and persisted unencoded in `data/processed/protected_val.csv`:

| Attribute | Prohibited basis (ECOA / Reg B) |
|---|---|
| `CODE_GENDER` | Sex |
| Age band (from `DAYS_BIRTH`) | Age |
| `NAME_FAMILY_STATUS` | Marital status |
| `CNT_CHILDREN` | Familial status (proxy) |
| `NAME_EDUCATION_TYPE` | Disparate-impact proxy |

## 3. Methodology

- **Disparate Impact (DI) — "four-fifths rule":** for each group, DI = (group approval rate) /
  (most-favored group's approval rate). **DI < 0.80 → adverse impact.** (EEOC Uniform
  Guidelines; standard lending screen.)
- **Equal Opportunity:** equality of **TPR** (recall on true defaulters) across groups.
- **Predictive Equality:** equality of **FPR** (good applicants falsely declined) across groups.

We report all three because a model can pass one while failing another.

## 4. Findings

### 4.1 Disparate Impact (approval rates)

| Attribute | Most-favored | Least-favored | DI ratio | Adverse impact? |
|---|---|---|---|---|
| **Age band** | 60+ (90.0% approve) | **20s (58.7%)** | **0.65** | 🔴 **Yes** |
| **Education** | Higher education (86.0%) | **Lower secondary (65.4%)** | **0.76** | 🔴 **Yes** |
| **Family status** | Widow (86.1%) | Single (68.8%) / Civil marriage (68.7%) | ~0.80 | 🟡 Borderline |
| **Gender** | Female (79.8%) | Male (66.3%) | 0.83 | 🟢 Passes |

### 4.2 Error-rate gaps (equal opportunity / predictive equality)

| Attribute | TPR gap | FPR gap | Note |
|---|---|---|---|
| **Gender** | **0.14** (M 0.71 vs F 0.57) | 0.12 (M 0.30 vs F 0.18) | Model both catches and falsely flags men more |
| **Age band** | ~0.42 (20s 0.75 vs 60+ 0.33) | ~0.28 | Error rates rise steeply for younger applicants |

### 4.3 Summary

- **Age** is the most serious issue: it **fails the four-fifths rule** *and* shows the
  largest error-rate gaps. Age is a **prohibited basis** under ECOA.
- **Education** fails the four-fifths rule (proxy effect).
- **Gender** passes DI but has a meaningful **equal-opportunity gap**, partly because
  `CODE_GENDER` was a **top-3 model feature**.
- Small subgroups (e.g. *Academic degree*, n=40) are statistically unstable and are flagged
  but not concluded upon.

## 5. Mitigation experiment — remove `CODE_GENDER`

Retrained XGBoost without `CODE_GENDER`, evaluated at the same threshold:

| Variant | PR-AUC | Gender DI | Gender TPR gap |
|---|---|---|---|
| With `CODE_GENDER` | 0.2826 | 0.831 | 0.139 |
| **Without `CODE_GENDER`** | **0.2788** | **0.896** | **0.067** |

**Performance cost ≈ −0.4% PR-AUC; fairness clearly improves.** This is a strong,
low-cost mitigation.

## 6. Recommendations

1. **Remove `CODE_GENDER`** from the production feature set (risk **R4**). Validated as
   near-free.
2. **Business-necessity review** of age- and education-correlated features; if retained,
   document the justification and consider reweighting/calibration by subgroup (risks **R1**,
   **R2**).
3. **Adverse-action explainability:** every declined applicant receives the principal reasons
   (feature attributions), per ECOA notice requirements.
4. **Ongoing monitoring** of DI and TPR/FPR gaps post-deployment with minimum-sample guards
   (see [DRIFT_MONITORING.md](DRIFT_MONITORING.md)); proxy bias can persist after removing the
   direct attribute.

Findings feed the risk register in [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md) (R1–R5).
