# Dashboard Product Specification

Proposed interface only. Preferred MVP is Streamlit backed by the API, without direct CSV or database access. [KPIs](03_BUSINESS_CASE_AND_KPIS.md) define all metric semantics; [contracts](07_API_AND_EVENT_CONTRACTS.md) define permitted reads.

## Shared Display Rules

Every page shows **Educational historical replay**, run ID, virtual time, fully processed watermark and mode. Source-naive timestamps are labeled; host Cairo time is not substituted. As-of and retrospective views use visibly separate navigation, not a subtle filter. Predictions display prediction moment, model version and availability status. No score means unavailable, not zero risk. Actuals appear only after their source event is released; labels missing at cutoff remain unknown.

Headline metrics include denominator, time basis and missingness. Forecasts distinguish history, origin, target window and optional uncertainty; future actuals are blank. Risk probabilities are conditional on the modeled cohort. A synthetic scenario has a persistent banner and separate run; scenario metrics never appear in genuine validation. Do not display raw customer identifiers or review text by default.

## Pages and Stakeholder Actions

| Page / persona | Main content | Action and empty/error state |
|---|---|---|
| Executive / analyst | Placed orders, item value, average item order value, observed delivery coverage, conditional on-time rate, first-review distribution, complete-week forecast | Filter purchase cohort and state; open metric definition. Missing item values and unknown outcomes explicitly counted |
| Logistics | Placed orders with no observed delivery, overdue rule queue, risk-ranked orders, known promise and reason summary, geography coverage | Inspect or acknowledge suggestion. Never show final canceled status as if historically known; unavailable model leaves overdue rules usable |
| Customer experience | Delivered orders with no known answer, M2 scores if approved, actual delay, response-age distribution, labeled coverage | Inspect and acknowledge internal follow-up suggestion. Already reviewed orders leave eligibility; no messaging integration |
| Forecasting | Complete weekly history, frozen forecast vintages, horizon-specific MAE/WAPE, interval coverage, naive comparison | Select origin/horizon; later actuals attach only on week closure. Partial weeks shaded/excluded, sparse categories not automatically modeled |
| Simulation operations | Run state, virtual clock, release/processing watermarks, pending jobs, quarantine reasons, pause/step/resume/speed/reset | Controls respect revision and state. Reset explains that it creates a new run; current run remains inspectable |
| ML trust and quality | Artifact/feature/target versions, cohort support, matured metrics, calibration, missing labels by age, score failures and drift | Drill into aggregate slices with counts; show “insufficient labels” rather than confident noisy performance |
| Retrospective reconciliation | Final statuses, item/payment differences, timestamp contradictions and source lineage | Explicit final-snapshot mode; not an operational cash ledger |
| Optional buyer groups | Recency/value bands and repeat-frequency distribution | Describe observed behavior only; no loyalty/CLV claims |

State geography is sufficient for MVP. ZIP map/distance is optional and must disclose missing/outlier/ambiguous prefixes. Do not use precise location markers as individual household coordinates.

## Demo Storyline

1. Open audit summary and show the dataset path, fingerprint and quality caveats. Establish the historical educational scope.
2. Select the frozen June test run. Warm-up pre-June state is present; future delivery/review fields are absent.
3. Step one timestamp batch to a placement with sufficient inputs. Inspect the as-of feature provenance, delivery score and independent rule decision. If the model was not accepted, demonstrate the baseline transparently.
4. Advance to an observed delivery for a scored order. Show unchanged saved prediction and newly available actual label; aggregate evaluation coverage increases.
5. For an eligible unreviewed delivery, show conditional CX inference only if validated; advance to its answer and show the first valid review label.
6. Cross a complete-week boundary. Show a scheduled forecast vintage, not per-order forecast requests; compare to actual only after target closure.
7. Pause and resume, inspect a quarantined event, and reset into a new run. Show prior-run isolation and reproducibility.
8. Close with limitations: delayed/missing outcomes, snapshot assumptions, forecast truncation and unproven intervention effects. No successful model metrics are scripted before actual evaluation.

Demo fixture orders must be chosen in a future authorized phase from genuine eligible cases; no fabricated order IDs or outcomes are specified here.

## Product Acceptance Criteria

A stakeholder can identify the score's moment and target population, explain why a record has no label, distinguish observed overdue from predicted late, and trace a KPI denominator. A replay operator can see processing lag and recover without double-counting. An analyst can reconcile item/payment totals without joining raw children together. A forecast reviewer can compare like-for-like horizons and origins.

Accessibility plan: text labels in addition to color, consistent date formats, table alternatives for charts, visible empty/error states, concise explanations and keyboard-accessible controls. These are future design criteria, not tested UI behavior.

## Assumptions / Evidence / Open Questions

Evidence: [audit](01_DATASET_AUDIT.md) supports the quality panels and cohort caveats. Assumptions: local single-user demo; simulated personas. Open questions: presentation time, language beyond technical English, and whether CX/optional buyer page fits the agreed scope.
