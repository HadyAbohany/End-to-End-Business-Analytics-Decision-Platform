# ML Lifecycle, Reliability and Test Strategy

This is a future verification plan. **No application tests, training runs, pipelines or deployments were executed.** Completed planning checks are reported separately in the executive summary.

## Reproducibility and Promotion

Each future experiment records raw file hashes, source/licensing evidence, cohort definition, exclusions, fit/label cutoffs, fold membership definition, target/feature versions, preprocessing, library versions, random seed, parameters, artifact digest, metrics with denominators, calibration and limitations. Store dataset/split manifests rather than raw data in public version control. MLflow may organize these records but is not required for the minimum capstone.

Artifacts have a feature signature, target contract and training cutoff. A reviewer compares baselines, leakage checks and validation evidence, then freezes the artifact/policy before final holdout. The final test is reported once for acceptance; failed candidates remain documented, not repeatedly tuned on the same test. Promotion requires a reproducible comparison, meaningful validation improvement for the stated decision, adequate coverage and no unresolved leakage. Numeric business thresholds remain owner-defined; no arbitrary AUC acceptance target is invented.

Rollback selects a prior compatible approved artifact for a new policy/run revision. Do not overwrite existing predictions or silently rescore with a new model. Explanations are versioned with the pipeline; feature importance is associative. Optional SHAP is added only after basic calibration and reliability pass.

## Verification Matrix — Planned, Not Executed

| Layer | Essential check / failure example | Required result |
|---|---|---|
| Source | Hash/column/count drift; nullable types, nonnumeric money, malformed dates | Versioned manifest match or explicit new audit; no silent schema coercion |
| Keys and joins | Items × payments × reviews fan-out; raw geography many-to-many | Order count and independently summed money preserved |
| Labels | Midnight promise policy; pre-delivery/equal-time review; missing delivery | Versioned target semantics; unknown stays unknown |
| Point-in-time | Future outcome sentinel; full-history seller aggregate; future category dictionary | Future values absent from features, with fitted transforms limited to fit cutoff |
| Same-time events | Two orders and delivery/review share timestamp | Deterministic batch result; no same-time cross-order history; suppress already-known M2 target |
| Replay | Duplicate ingestion; crash after commit before ACK; expired job lease | One effective event/prediction/alert; restart resumes same state |
| Warm start | Pre-test purchase with in-test delivery | Order state present; M1/M2 cohorts assigned by their own moments |
| Run isolation | Reset and synthetic run reuse original order IDs | No cross-run features, labels or metrics |
| Source contradictions | Handoff before purchase or after delivery | Flag/quarantine specific transition, no invented timestamp |
| Forecasting | Partial last week; origin with incomplete 4-week future; unshifted rolling feature | Reject invalid target evaluation and lookahead |
| Model contract | Wrong schema/digest, unknown category, missing artifact | Explicit failure/unknown handling, no zero-risk default |
| Access | UI/scoring process attempts raw-vault read | Denied; raw files not mounted to those processes |
| Delayed evaluation | Late review arrives; no answer; invalid review arrives first | Immutable prediction; valid label matching and transparent coverage |
| API | Invalid command revision, future cutoff, duplicate identity with conflicting payload | 409/422 or other documented errors, no state corruption |
| Recovery | Restore DB without matching manifest/artifact | Refuse unsafe run continuation |
| Resource/load | Accelerated replay and weekly jobs share worker | Bounded memory and visible lag; no lost observations |

## Operational and ML Monitoring

Separate virtual-time lag (released minus fully processed event time), wall processing latency, queue backlog age, job retry/dead-letter count, source quality count and label delay. Accelerated replay changes wall throughput, not model time features. Monitor predicted probability distribution, missing-feature rates, unknown-category/new-seller rates and score coverage by month/region. Baseline windows come from training data; drift thresholds are calibrated on historical variability, not universal magic cutoffs.

Use distribution summaries/PSI or KS only as diagnostic evidence with sample-size context, not automatic proof of performance loss. Report matured PR-AUC, Brier and top-K precision with class counts and coverage. Require adequate positive-label support before subgroup judgments; pool or suppress tiny slices. Review arrival and delivery arrival are different maturity processes. A temporary fall in labeled performance may reflect selection and delay; compare age-matched cohorts before acting.

Monitor interval coverage and bias by forecast horizon on fully observed weeks. Olist ends abruptly; apparent collapse in September 2018 is a source-boundary issue, not a learned business recession. Source-data drift, synthetic perturbations and runtime failures have separate panels.

## CI/CD and Security Plan

Future CI runs formatting/static checks, small synthetic contract fixtures, deterministic replay tests, temporal leakage assertions and artifact signature tests. Raw licensed data is not uploaded to public CI; full-data checks run locally with hashes. Building an image does not authorize cloud deployment. Model promotion remains a reviewed artifact action; retraining is not automatic on every commit. No YAML, Dockerfile or test code is generated in this phase.

Future deployment review checks loopback ports, minimum DB roles, secret handling, immutable artifact loading from trusted source, parameterized queries, log redaction and backup/restore compatibility. Pickled model artifacts must come from a trusted build, not arbitrary uploaded files. No production SLA is claimed. Resource targets in [architecture](06_SYSTEM_ARCHITECTURE.md) are unmeasured allowances.

## Definition of Done for a Future MVP

Reconciled KPIs, deterministic restart/reset, demonstrable future-truth denial, complete prediction lineage, honest baseline results, delayed-label coverage, partial-week protection and a successful offline demo. Baseline-only results can satisfy scientific correctness if stronger candidates fail. Future work requires separate explicit authorization.

## Assumptions / Evidence / Open Questions

Evidence: test cases derive from actual audited anomalies and [contracts](07_API_AND_EVENT_CONTRACTS.md). Assumptions: single-machine deployment and an offline test artifact workflow. Open questions: operational alert capacity, acceptable failure latency and owner-approved promotion criteria. No performance, latency or recovery measurement exists yet.
