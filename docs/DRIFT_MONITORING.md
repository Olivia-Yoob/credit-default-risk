# Drift & Fairness Monitoring Plan — Home Credit Default Risk Classifier

*Post-deployment monitoring plan (EU AI Act Art. 72 post-market monitoring). The statistical
control-limit design follows a quantitative KRI approach: a moving-average baseline with
95% / 99% confidence bands.*

---

## 1. What we monitor

| Layer | Metric | Why |
|---|---|---|
| **Input drift** | **PSI** (Population Stability Index) per feature | Detect shift in applicant population vs training |
| **Prediction drift** | Mean predicted P(default); approval rate | Detect score inflation/deflation |
| **Performance** | PR-AUC, ROC-AUC, recall @ threshold 0.10 | Detect predictive decay (needs label maturation) |
| **Fairness** | DI ratio + TPR/FPR gaps by protected group | Detect emerging disparate impact |
| **Data quality** | Missing-rate per feature (esp. `EXT_SOURCE_1`) | Detect pipeline/vendor breakage |

## 2. PSI thresholds (input drift)

PSI = Σ (actual% − expected%) · ln(actual% / expected%), computed per feature against the
training distribution (10 bins).

| PSI | Interpretation | Action |
|---|---|---|
| < 0.10 | No significant shift | None |
| 0.10 – 0.25 | Moderate shift | Investigate; flag feature |
| > 0.25 | Major shift | Alert; consider retraining |

## 3. Statistical control limits (KRI-style)

For each monitored metric we maintain a **rolling baseline** (e.g. 12–24 monthly observations)
and alert when the current value breaches the band:

```
Upper/Lower limit = moving_average ± z · σ_moving
  z = 1.96  → 95% band  (warning)
  z = 2.576 → 99% band  (breach / escalation)
```

- **Warning (95% breach):** log, notify model owner, begin investigation.
- **Breach (99% breach):** escalate to Model Risk; evaluate suspension / retraining.

This converts ad-hoc "the metric looks off" into an auditable, pre-agreed trigger — the same
control logic used for operational-risk KRIs.

## 4. Fairness monitoring specifics

- Recompute **DI** (four-fifths) and **TPR/FPR gaps** for gender, age, education, family on a
  fixed cadence (e.g. monthly) using matured labels.
- **Minimum-sample guard:** suppress/annotate any subgroup with n below a set floor (rationale:
  *Academic degree* n=40 produced unstable metrics in the audit — R5).
- **Hard fairness trigger:** any protected group's DI dropping below **0.80** → immediate
  review, regardless of control-band status.

## 5. Retraining triggers

Retrain (or roll back to a prior versioned model) when **any** of:
1. PSI > 0.25 on a top-10 feature, **or** several features in 0.10–0.25 simultaneously;
2. PR-AUC breaches the 99% lower control limit;
3. A protected group's DI falls below 0.80 or a TPR gap exceeds its control band;
4. `EXT_SOURCE_*` missing rate changes materially (vendor data issue).

## 6. Governance & cadence

| Activity | Cadence | Owner |
|---|---|---|
| Input/prediction drift (PSI) | Weekly | Data Science (1st line) |
| Performance & fairness review | Monthly | Model Risk (2nd line) |
| Full audit refresh + risk-register update | Quarterly / on trigger | Model Risk |
| Framework assurance | Annual | Internal Audit (3rd line) |

All monitoring outputs are logged to satisfy record-keeping (EU AI Act Art. 12) and feed back
into [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md).

*Reference implementation note:* PSI and the moving-average control limits are intended to live
in `src/governance_metrics.py` as reusable functions.
