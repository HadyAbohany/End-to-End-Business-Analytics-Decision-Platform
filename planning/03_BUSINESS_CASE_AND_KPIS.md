# Business Case and KPI Dictionary

## Purpose and Stakeholders

Demonstrate a reproducible educational historical decision-support system. Executives inspect order/item value trends; logistics staff prioritize risky orders; CX staff inspect unreviewed deliveries; analysts reconcile metrics; operators control replay; ML reviewers assess performance and coverage. No live company integration or measured intervention uplift is claimed.

Success means reproducible arithmetic, enforced as-of access, deterministic replay and honest baseline comparisons. A complicated model that fails the temporal test must not be promoted. Business thresholds require an owner-defined review capacity; no invented ROI target.

## Shared Metric Semantics

Every response includes run, virtual cutoff, metric-definition version, time basis, cohort and coverage. A purchase cohort is a half-open interval [start,end) in source wall time. As-of results use events visible by the cutoff; retrospective results are a separately labeled mode. Final status is usable only in retrospective reporting. Currency is provisionally BRL per dataset context, not independently documented by a currency column.

| KPI | Exact numerator / denominator | Grain, date and exclusions |
|---|---|---|
| Placed orders | count distinct order_id | Purchase cohort, all final statuses; source count 99,441. Do not infer paid or successful orders |
| Item sales value | sum items.price | Item → order aggregation, then purchase cohort. All recorded items; disclose 775 orders without items, not zero-value assumed orders |
| Freight charges | sum items.freight_value | Same item cohort; charges, not transport cost |
| Average item order value | total item price / distinct orders with ≥1 item | Purchase cohort; explicitly differs from dividing by all placed orders |
| Payment snapshot total | sum payments.payment_value | Payment-sequence grain, joined once per order for cohort. Undated: retrospective or explicitly assumed visibility only, never cash-flow-by-day |
| Reconciliation difference | payment total − item price − freight | Only orders with both children; match 98,665 orders. 303 differ by >0.01, max 182.81. Separately count missing sides; do not silently force equality |
| Observed delivery coverage | orders with valid delivery ≥ purchase / placed orders | Purchase cohort as of cutoff; complements censored/missing, not failure rate |
| On-time observed delivery rate | deliveries whose calendar date ≤ promise date / valid observed deliveries with promise | Source-wide 89,941 / 96,476 = 93.23%; all statuses, with six status contradictions flagged. Observed-delivery conditional metric |
| Late observed delivery rate | deliveries whose calendar date > promise date / same denominator | 6,535 / 96,476 = 6.77%. Timestamp-midnight alternative is 7,827 / 96,476 = 8.11%, separately versioned sensitivity |
| Unresolved overdue | placed orders past promise-day end without observed delivery | As-of rule; source lacks cancellation times, so label “no observed delivery,” not active shipment failure |
| First-review negative rate | first valid observed review score ≤2 / orders with a first valid observed answer | Answer selected chronologically; same-time review_id tie break. Reject pre-purchase answers. Raw first-review diagnostic 14,476/98,673 is before invalid-answer exclusions |
| Delivery-time CX negative rate | negative first answers after delivery / labeled delivered, previously unreviewed cohort | Audit diagnostic 8,612/91,061 = 9.46%; explicit conditional scope, different from all-review rate |
| Review response coverage | first-answer-observed orders / eligible placed or delivered cohort | Denominator selected in UI and never mixed; age since purchase/delivery displayed |
| Delivery duration | median and p90 of delivery − purchase in elapsed days | Valid observed deliveries; no censoring-adjusted claim |
| Seller historical late rate | prior late observed orders linked to seller / prior labeled orders linked to seller | Each order once per seller, label arrival strictly before score. Multi-seller order outcome is shared attribution, not seller causation |
| Forecast error | MAE = mean absolute error; WAPE = sum absolute error / sum actuals | Complete target weeks only, horizon-specific. Undefined WAPE if total actual is zero; display NA |
| Alert burden | unique active alerts and top-K precision among matured labels | One model/policy/order/run alert key; unreviewed alerts are not successful outcomes |

Category item-value totals reconcile if unknown category is retained. Category order counts overlap for multi-category baskets and are non-additive. State breakdowns use customer state for demand; seller state is a different metric. No raw geography join is allowed.

## CRISP-DM and Assessment Coverage

| Phase | Planned evidence | Review gate |
|---|---|---|
| Business understanding | This KPI dictionary, stakeholder decisions and exclusions | Owner agrees labels, capacity and rubric |
| Data understanding | [Audit](01_DATASET_AUDIT.md), quality and feasibility matrix | Full-source inventory and anomaly accounting |
| Data preparation | [Replay/data specification](05_REPLAY_AND_DATA_DESIGN.md), as-of joins | Future implementation proves no fan-out or leakage |
| Modeling | [Model cards](04_ML_MODEL_DESIGN.md), baseline comparison plan | Locked temporal protocol before fitting |
| Evaluation | [Test strategy](09_ML_OPS_AND_TEST_STRATEGY.md), coverage and limitations | Honest acceptance or rejection per model |
| Deployment planning | [Architecture](06_SYSTEM_ARCHITECTURE.md), contracts, local resource plan | Proposed only; separate implementation authorization |
| Communication | [Dashboard](08_DASHBOARD_PRODUCT_SPEC.md), demo and executive summary | Stakeholder distinguishes prediction, observation and assumption |

## Assumptions / Evidence / Open Questions

Evidence: financial totals and cohort denominators are from the local audit. Assumptions: end-of-calendar-day promise interpretation and BRL display convention. Open questions: official DEPI rubric has not been supplied; coverage above is a proposed mapping, not certification. Owner must choose whether an unsuccessful ML comparison is acceptable (recommended: yes).
