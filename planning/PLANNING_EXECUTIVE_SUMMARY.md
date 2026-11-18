# Olist Planning Executive Summary

**PLANNING COMPLETE — NO IMPLEMENTATION PERFORMED.**

Both supplied Markdown briefs were read as source material; the user's request controls scope. All nine CSVs were fully scanned in memory. The requested `data/raw/` directory is absent; the actual source is `/Users/hadyahmed/Downloads/archive-3/`. No source data was moved or modified. Only Markdown planning documents were created.

## Evidence and Recommendation

The snapshot contains 99,441 orders, 112,650 items, 103,886 payments, 99,224 review rows, 99,441 customer records, 32,951 products, 3,095 sellers, 1,000,163 geolocation rows and 71 translations. Exact columns, nulls, keys, timestamp ranges, hashes, FK coverage and anomalies are in the [audit](01_DATASET_AUDIT.md).

Recommend a local modular FastAPI/PostgreSQL/Streamlit design, a scoring/batch worker and a separately isolated replay scheduler. Use durable DB jobs first; Redis, MLflow, React and separate inference services are optional extensions. The scheduler can read future truth; API, scoring and UI cannot. No architecture, service or model has been implemented or performance-validated.

| Capability | Verdict |
|---|---|
| Reconciled analytics, historical replay, transparent rules and state geography | GO as a proposed educational scope |
| Placement-time late-delivery risk | CONDITIONAL GO; main supervised experiment |
| Company-wide weekly order forecasting | CONDITIONAL GO; 1-week primary, 2–4 weeks secondary |
| Delivery-time negative-review risk | CONDITIONAL GO extension; prove value beyond known lateness |
| Buyer segmentation | GO descriptive bands; defer ML clustering |
| Delivery-duration regression | Technically conditional; exclude from MVP as redundant |
| Inventory, profit, refunds, causal impact, reliable CLV, real Olist monitoring | NO-GO without appropriate additional data |

## Critical Findings

- Calendar-day versus timestamp lateness gives 6,535 versus 7,827 positives among 96,476 observed deliveries. Adopt calendar-day comparison, with explicit sensitivity reporting; missing delivery is not an on-time label.
- 4,769 first reviews were answered at/before delivery. The raw first-answer-after-delivery cohort is 91,061 orders, with 8,612 negatives. Score only delivered, not-yet-reviewed orders and report responder selection.
- Only 2,997 of 96,096 buyers repeat. Clustering and customer lifetime claims are weak additions.
- Forecast history is short: 86 complete calendar weeks in the chosen stable span; September–October 2018 has only 20 orders. Treat late collapse as uncertain extract coverage, not business demand truth.
- Geography has 261,831 exact duplicate rows and ambiguous/outlying coordinates. Child-table joins can inflate money and labels; 303 matched orders differ by more than one cent between payments and item-plus-freight totals.
- Final statuses are not transition history. Carrier chronology conflicts, 63 review answers before purchase, undated payments and static catalog/promise assumptions require explicit handling.

## Deliverable Index

1. [Dataset audit](01_DATASET_AUDIT.md)
2. [Idea-to-data feasibility matrix](02_FEASIBILITY_MATRIX.md)
3. [Business case and KPI dictionary](03_BUSINESS_CASE_AND_KPIS.md)
4. [Model cards and temporal protocol](04_ML_MODEL_DESIGN.md)
5. [Replay and data design](05_REPLAY_AND_DATA_DESIGN.md)
6. [System architecture and 11 Mermaid diagrams](06_SYSTEM_ARCHITECTURE.md)
7. [API and event contracts](07_API_AND_EVENT_CONTRACTS.md)
8. [Dashboard specification and demo](08_DASHBOARD_PRODUCT_SPEC.md)
9. [ML lifecycle and future test strategy](09_ML_OPS_AND_TEST_STRATEGY.md)
10. [Roadmap, decisions and risks](10_ROADMAP_DECISIONS_AND_RISKS.md)

Architecture includes context, MVP containers, components, raw ERD, curated ERD, data flow, delayed-label sequence, offline training/batch forecasting, deployment, replay state and optional expanded services. Mermaid source is included; rendered layout was not verified.

## Assumptions / Evidence / Open Questions

Evidence is local full-file profiling, not predictive performance. Original Kaggle page was opened but unreadable through the available web tool; exact local version and license binding remain unverified. [Creative Commons terms](https://creativecommons.org/licenses/by-nc-sa/4.0/) were checked; original source/download evidence is still required before redistribution. No raw dataset is redistributed here.

Owner choices remain: scope/rubric, static placement assumptions, promise-day convention, alert capacity and team/deadline/resources. Missing source timestamps/outcomes and extract completeness are data limitations, not owner choices. The proposed temporal protocol freezes final models before June 2018, scores through August 26, and separates later label follow-up from training.

Completed checks: full-source inventory and profiling; cross-document links, expected document presence and Mermaid fence inventory. Not completed or claimed: model training/testing, runtime verification, rendered diagram validation, deployment or implementation.
