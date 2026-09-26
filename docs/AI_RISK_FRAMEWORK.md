# AI Risk Framework & Risk Register — Home Credit Default Risk Classifier

*Operational-risk-style framework applied to an ML model: identify, assess (severity ×
likelihood), control, assign ownership, and monitor. Findings trace to notebooks `01`–`05`.*

---

## 1. Risk taxonomy

| Category | Description |
|---|---|
| **Fairness / discrimination** | Disparate outcomes across protected groups |
| **Data quality** | Missingness, sentinels, leakage, representativeness |
| **Performance / robustness** | Accuracy decay, drift, instability on subgroups |
| **Transparency / explainability** | Ability to explain individual decisions |
| **Operational / governance** | Oversight, logging, change control |

## 2. Severity scale

| Level | Meaning |
|---|---|
| **High** | Legal/regulatory breach or material consumer harm plausible |
| **Medium** | Material fairness/performance concern; mitigation required pre-deployment |
| **Low** | Limited impact; monitor |

## 3. Risk register

| ID | Category | Finding (evidence) | Severity | Control / mitigation | Owner |
|---|---|---|---|---|---|
| **R1** | Fairness | **Age 20s approval DI = 0.65** (< 0.80) — adverse impact (`05`) | **High** | Business-necessity review of age-correlated features; subgroup calibration; monitor | Model Risk + Legal |
| **R2** | Fairness | **Education (Lower secondary) DI = 0.76** (< 0.80) — proxy effect (`05`) | Medium | Test removal/encoding; document necessity | Model Risk |
| **R3** | Fairness | **Gender TPR gap = 0.14** (equal-opportunity) (`05`) | Medium | Remove `CODE_GENDER` (validated ~free); monitor gap | Data Science |
| **R4** | Fairness | **`CODE_GENDER` is a top-3 model feature** (`04`) — direct use of protected attribute | **High** | **Drop from production model** (PR-AUC −0.4% only) | Data Science |
| **R5** | Transparency | **Small subgroups unstable** (e.g. Academic degree n=40) (`05`) | Low | Minimum-sample threshold in monitoring & reporting | Model Risk |
| **R6** | Data quality | **`EXT_SOURCE_1` ~56% missing**, median-imputed (`02`/`03`) — strong but partial signal | Medium | Missing-indicator flag; sensitivity analysis; vendor data review | Data Science |
| **R7** | Performance | **No out-of-time validation** (static dataset) | Medium | Out-of-time test + post-deployment drift monitoring | Model Risk |
| **R8** | Operational | **Human-oversight & adverse-action procedure not operationalized** | Medium | Define HITL workflow + ECOA reason codes | Governance |

## 4. Controls summary

- **Preventive:** leakage-free preprocessing (`03`); protected-attribute removal (R4);
  documented cost-based threshold (`04`).
- **Detective:** Fair Lending audit (`05`); drift & fairness monitoring
  ([DRIFT_MONITORING.md](DRIFT_MONITORING.md)).
- **Corrective:** retraining triggers on drift/fairness breach; model rollback via versioned
  artifacts.

## 5. Three-lines-of-defense alignment

| Line | Role here |
|---|---|
| **1st — Data Science** | Builds model, applies preventive controls, owns R3/R4/R6 mitigations |
| **2nd — Model Risk / Compliance** | Independent validation, owns the audit & risk register, signs off pre-deployment |
| **3rd — Internal Audit** | Periodic assurance over the framework's operation |

## 6. Residual risk & sign-off gate

Deployment should be gated until **R1, R4** are resolved (High severity) and **R7, R8** have
documented plans. Residual fairness risk after mitigation must be re-measured and recorded in
an updated [FAIR_LENDING_AUDIT.md](FAIR_LENDING_AUDIT.md) before go-live.
