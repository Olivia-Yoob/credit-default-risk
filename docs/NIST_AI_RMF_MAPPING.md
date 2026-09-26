# NIST AI Risk Management Framework (AI RMF 1.0) Mapping

*Maps this project to the four NIST AI RMF functions — **GOVERN, MAP, MEASURE, MANAGE** — and
their key categories. Evidence references notebooks `01`–`05` and the other `docs/`.*

---

## GOVERN — culture, accountability, policies

| Category | How addressed | Evidence |
|---|---|---|
| GOVERN 1 (policies, legal/reg awareness) | Model classified High-Risk (EU AI Act) and ECOA-relevant; obligations enumerated | [EU_AI_ACT_COMPLIANCE.md](EU_AI_ACT_COMPLIANCE.md) |
| GOVERN 2 (accountability, roles) | Three-lines-of-defense ownership; per-risk owners | [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md) §5 |
| GOVERN 4 (risk culture / documentation) | Full reproducible documentation set + model card | this `docs/` set, `MODEL_CARD.md` |
| GOVERN 5 (monitoring processes) | Defined monitoring cadence & triggers | [DRIFT_MONITORING.md](DRIFT_MONITORING.md) |

## MAP — context & risk identification

| Category | How addressed | Evidence |
|---|---|---|
| MAP 1 (context, intended purpose) | Intended use / out-of-scope use defined | `MODEL_CARD.md` §2 |
| MAP 2 (classification, capabilities) | High-Risk classification; XGBoost on 96 features | `MODEL_CARD.md` §1, §3 |
| MAP 3 (benefits & costs) | Cost-based decision threshold (FN:FP = 10:1) makes the benefit/cost trade explicit | notebook `04` |
| MAP 5 (impacts to individuals/groups) | Protected attributes registered; impacts identified | notebooks `01`, `05` |

## MEASURE — analyze, assess, benchmark

| Category | How addressed | Evidence |
|---|---|---|
| MEASURE 1 (appropriate methods/metrics) | PR-AUC/ROC-AUC/recall chosen for imbalance; accuracy rejected | notebook `04` |
| MEASURE 2.1 (performance) | PR-AUC 0.283, ROC-AUC 0.781, recall 0.63 @ 0.10 | `MODEL_CARD.md` §5 |
| MEASURE 2.11 (fairness / bias) | DI (four-fifths), equal-opportunity, predictive-equality | [FAIR_LENDING_AUDIT.md](FAIR_LENDING_AUDIT.md) |
| MEASURE 2.9 (explainability) | Feature importance; per-decision attribution for adverse action | notebook `04` §6 |
| MEASURE 4 (feedback / validity over time) | Drift & fairness monitoring with control limits | [DRIFT_MONITORING.md](DRIFT_MONITORING.md) |

## MANAGE — prioritize and act

| Category | How addressed | Evidence |
|---|---|---|
| MANAGE 1 (prioritize risks) | Severity-ranked risk register R1–R8; deployment gated on High risks | [AI_RISK_FRAMEWORK.md](AI_RISK_FRAMEWORK.md) |
| MANAGE 2 (mitigation strategies) | Validated mitigation: remove `CODE_GENDER` (~free); necessity review for proxies | `FAIR_LENDING_AUDIT.md` §5–6 |
| MANAGE 3 (third-party/data risks) | `EXT_SOURCE` vendor-data dependency & missingness tracked (R6) | `AI_RISK_FRAMEWORK.md` |
| MANAGE 4 (monitoring & response) | Retraining/rollback triggers; escalation on 99% control breach | [DRIFT_MONITORING.md](DRIFT_MONITORING.md) §5 |

---

## Trustworthy-AI characteristics coverage

| NIST characteristic | Status |
|---|---|
| Valid & reliable | 🟢 Held-out metrics; 🟡 out-of-time validation pending (R7) |
| Safe | 🟡 Human oversight design-level (R8) |
| Fair — harmful bias managed | 🟢 Audited + mitigation validated; 🟡 proxy monitoring ongoing |
| Explainable & interpretable | 🟢 Feature attribution available |
| Accountable & transparent | 🟢 Model card + documentation set |
| Privacy-enhanced | 🟡 Not in scope for this dataset |
| Secure & resilient | 🔴 Robustness/security testing pending (R-future) |

**Summary:** strongest coverage in **MEASURE** (performance + fairness) and **GOVERN/MAP**
(documentation + classification); remaining gaps are operational (out-of-time validation,
human-oversight procedure, robustness testing) and tracked in the risk register.
