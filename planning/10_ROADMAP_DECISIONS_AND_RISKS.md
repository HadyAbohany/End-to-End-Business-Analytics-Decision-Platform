# Scope, Decisions, Roadmap and Risks

This roadmap describes potential future work; it does not authorize implementation. Planning ends with these documents.

## Scope Options

| Option | Included | Excluded / rationale |
|---|---|---|
| Minimum credible capstone | Audited data, reconciled retrospective and as-of KPI views, deterministic replay, M1 baseline experiment, delayed-label reporting, simple total-order forecast baselines | CX model, clustering, ETA, broker, React, cloud and synthetic scenarios; prioritize correctness |
| Strong portfolio MVP — recommended | Minimum plus comparative M1 model, rolling weekly forecasts and intervals, quality/trust dashboard, frozen model/version lineage, recovery demonstration | M2 only if simple delivery-lateness rule is beaten and time permits; no mandated count of models |
| Optional production-inspired extensions | MLflow, Redis/outbox, separate inference runtime, React, richer geographic features, CX model, synthetic scenario isolation | Requires measured bottleneck or team ownership; cloud only after separate authorization/provenance review |

If DEPI requires three independent models, owner must supply the rubric. Preferred response is an explicitly evaluated conditional M2, not relabeling RFM bands or repeated algorithms as independent business problems.

## Milestones and Dependencies

| Milestone | Future outputs | Dependency and acceptance gate | Relative effort |
|---|---|---|---|
| A: owner specification review | Scope, rubric, source/license record, assumptions and KPI choices | Planning reviewed; separate implementation authorization | Small |
| B: curated temporal contracts | Source validation, independent child aggregation, event mapping and quality quarantine | A; match audit counts and explain every exclusion | Medium |
| C: replay and KPI foundation | Durable ledger, controls, warm start, as-of views and recovery | B; no future access, no duplicates, reconciled arithmetic | Large |
| D: offline baselines and models | Frozen temporal protocol, M1 and forecast comparisons, optional M2 | B; feature-time/maturity gates before fitting; no test tuning | Large |
| E: integration and dashboard | Versioned inference, rules, delayed evaluation and forecast scheduler | C and accepted artifacts from D; failed inference visible | Medium–large |
| F: reliability and presentation | Recovery/access tests, resource measurement, reproducible demo and limitations | E; all critical checks pass or scope reduced | Medium |
| G: optional extensions | Broker, services, geography, CX or scenarios | Measured value and separate scope decision after F | Variable/high |

No calendar/person-day estimate is justified without team size, experience, availability and deadline. Dependency-aware effort is relative; C and D are the largest risk areas. If time is tight, remove extensions before reducing temporal correctness. After B, data/replay and offline-model work could be separate team workstreams, but no agents or implementation workstreams were launched in this planning task.

## Architecture Decision Records

| ADR | Decision / challenged assumption | Evidence and consequence |
|---|---|---|
| 001 | Audit archive in place; do not assume data/raw exists | Workspace is empty except Git; all nine files attached under Downloads/archive-3 |
| 002 | Modular backend plus isolated scheduler | Future truth must be inaccessible to scoring; one all-powerful worker breaks that guarantee |
| 003 | DB event/jobs table before Redis | ~99k orders; no throughput evidence warrants broker/outbox overhead |
| 004 | Conditional delivery labels, calendar-date primary | 6,535 vs 7,827 late cases depending on policy; undelivered outcomes unknown |
| 005 | M2 scores delivery-time unreviewed cohort | 4,769 first answers already at/before delivery; full review set is invalid |
| 006 | Total recorded orders before sales/category demand forecasts | 86 usable calendar weeks, sparse categories and late truncation; no demand/capture ledger |
| 007 | Descriptive buyer bands before clustering | 96.88% buyers have one observed order; weak frequency discrimination |
| 008 | Defer duration regression | Same predictors/outcomes as M1 without a distinct agreed decision |
| 009 | Undated payments are retrospective only | No settlement/visibility timestamps; avoid fabricated state |
| 010 | Source-naive clock, explicit snapshot assumptions | Timezone, catalog history and promise visibility unverified |
| 011 | No final status transitions inferred | 625 canceled snapshots but no cancellation time; six have delivery evidence |
| 012 | Preserve raw data and document only | User explicitly requested planning/read-only analysis; no implementation authorization |

## Risk Register

| Risk | Likelihood / impact | Mitigation and owner | Residual limitation |
|---|---|---|---|
| Future leakage through snapshot columns or aggregates | High / critical | Data/ML lead: per-field availability, cutoff fits, raw access denial | Static promise/catalog knowledge still assumed |
| Review timing and responder selection | High / high | ML lead: delivery-time eligibility, first valid answer, response-age coverage | Cannot infer all-customer dissatisfaction |
| Forecast truncation or extract selection | High / high | Analyst: stable window, complete weeks, horizon maturity | Underlying true demand remains unknown |
| Late-class drift | High / high | ML lead: temporal validation/calibration and per-period counts | Test prevalence differs materially from early 2018 |
| As-of final-status/cancellation fiction | High / high | Data lead: observed events only; separate retrospective mode | Operational “active” orders cannot be fully recovered |
| Join inflation | High / high | Analyst: preaggregate each child and retain unknown categories/geography | Metric meanings still need stakeholder agreement |
| Replay retries/crashes | Medium / high | Backend owner: atomic jobs/state, consumer uniqueness, checkpoints | Future behavior must be tested |
| Overengineered service fleet | High / medium | Architect: modular MVP, optional expansion on evidence | Team learning goals may justify some extension |
| Source license/version not confirmed | Medium / high for redistribution | Owner: recover original download metadata and license binding | Local hashes prove identity only, not rights |
| Limited repeat buyers / category support | High / medium | Analyst: descriptive grouping and company forecast | No reliable CLV or broad category model portfolio |
| Resource/demo limits | Unknown / medium | Operator: measure on target laptop, bounded batch processing | No performance benchmark performed |

## Owner Decisions Versus Technical Blockers

**Owner decisions:** accept placement-time snapshot/promise assumptions and calendar-day target; choose scope and whether optional CX is required; provide DEPI rubric, team/deadline/machine budget, review queue capacity and intended audience. Recommended defaults are documented, so planning is complete without these answers.

**Technical/data limitations:** actual raw directory differs from request; payment/cancellation times, inventory/cost/returns/interventions and system telemetry are absent; source version/license attachment and timezone are unverified; final extract completeness is unknown. No architecture fixes missing data. These block specific claims or redistribution, not completion of the blueprint. Predictive usefulness remains untested and can only be resolved during separately authorized implementation.

## Assumptions / Evidence / Open Questions

Evidence: [audit](01_DATASET_AUDIT.md) and [feasibility matrix](02_FEASIBILITY_MATRIX.md). Assumptions and unresolved decisions above are explicit, not hidden defaults. Original briefs are planning input, not instructions to build or deploy. Stop here until separate explicit implementation authorization.
