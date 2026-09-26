# EU AI Act Compliance Mapping — Home Credit Default Risk Classifier

*Illustrative conformity mapping for a portfolio project. References are to Regulation (EU)
2024/1689 (the AI Act). This is not legal advice.*

---

## 1. Risk classification

| Question | Determination |
|---|---|
| Is the system AI under Art. 3(1)? | Yes — a trained ML classifier. |
| Is it listed in **Annex III**? | **Yes — §5(b):** AI intended to **evaluate creditworthiness / establish credit scores** of natural persons. |
| Resulting class | **High-Risk** |

High-Risk classification triggers the Chapter III obligations mapped below.

## 2. Obligations mapping (Arts. 8–17)

| Art. | Obligation | How this project addresses it | Status |
|---|---|---|---|
| **9** | **Risk management system** — iterative, lifecycle | Risk register R1–R5 with severity, controls, owners — [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md) | 🟡 Established; needs lifecycle cadence |
| **10** | **Data & data governance** — relevant, representative, examined for bias | Data inventory + protected-attribute registration (`01`); leakage-free split & imputation (`03`); missingness/bias examined (`02`/`03`) | 🟢 Addressed |
| **11** | **Technical documentation** (Annex IV) | This `docs/` set + reproducible notebooks `01`–`05` | 🟢 Addressed |
| **12** | **Record-keeping / logging** | Saved artifacts: `model_metadata.csv`, `val_predictions.csv`, versioned figures | 🟡 Partial (no runtime event logs) |
| **13** | **Transparency & information to users** | Model card; documented intended use & out-of-scope uses | 🟢 Addressed |
| **14** | **Human oversight** | Human-in-the-loop required for adverse actions; model is decision-support, not autonomous | 🟡 Design-level; needs operational procedure |
| **15** | **Accuracy, robustness, cybersecurity** | PR-AUC 0.283 / ROC-AUC 0.781 reported on a held-out split; cost-based threshold documented | 🟡 Accuracy yes; robustness/security pending |

## 3. Bias monitoring & non-discrimination (Art. 10(2)(f)–(g), Recital 67)

- Protected attributes identified up front (`01`) and audited (`05`).
- **Findings:** adverse impact on **age** (DI 0.65) and **education** (DI 0.76); gender
  equal-opportunity gap (TPR 0.14). See [FAIR_LENDING_AUDIT.md](FAIR_LENDING_AUDIT.md).
- **Action:** validated mitigation (remove `CODE_GENDER`, ~free); ongoing fairness monitoring
  specified in [DRIFT_MONITORING.md](DRIFT_MONITORING.md).

## 4. Transparency to affected persons (Art. 86 — right to explanation)

Declined applicants are entitled to a **clear explanation** of the decision. The model
supports this via feature-importance / per-applicant attribution; the adverse-action process
must surface the principal reasons in plain language.

## 5. Post-market monitoring (Art. 72)

A continuous monitoring plan covers **feature drift (PSI)**, **performance decay**, and
**fairness-metric drift** with statistical control limits — see
[DRIFT_MONITORING.md](DRIFT_MONITORING.md). This mirrors a quantitative KRI approach
(moving-average control limits at 95% / 99% confidence).

## 6. Outstanding items before deployment

1. Operational **human-oversight** procedure (Art. 14) and adverse-action workflow.
2. **Robustness & security** testing (Art. 15) — adversarial / stability testing.
3. **Runtime logging** to satisfy record-keeping (Art. 12) in production.
4. **Out-of-time validation** (the dataset is static) and a documented business-necessity
   review for age/education-correlated features.
5. Fundamental Rights Impact Assessment (Art. 27) where applicable to the deployer.

> **Compliance posture:** documentation, data governance, and bias assessment are in place;
> operational controls (oversight procedure, logging, robustness, out-of-time validation)
> remain before a real High-Risk deployment.
