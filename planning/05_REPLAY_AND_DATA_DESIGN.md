# Historical Replay and Data Design

All objects and processes are proposed. Raw CSVs remain unchanged. The simulator is an educational reconstruction, not recovery of Olist's actual transactional history.

## Event and Field Release Rules

| Event | Source timestamp | Released facts | Excluded / caveat |
|---|---|---|---|
| OrderPlaced | orders.order_purchase_timestamp | order_id, customer reference, purchase time; promise, basket and first referenced product/seller/address snapshots under explicit assumptions | Never final order_status, future dates or entire future catalog |
| OrderApproved | orders.order_approved_at | observed approval time | Does not imply payment settled; no payment timestamp exists |
| CarrierHandoff | orders.order_delivered_carrier_date | valid observed handoff time | Quarantine impossible ordering; do not invent corrected timestamps |
| OrderDelivered | orders.order_delivered_customer_date | delivery time, known elapsed duration and lateness | May occur without observed approval/handoff; absent prior events stay unknown |
| ReviewSubmitted | reviews.review_answer_timestamp | review_id + order_id, score, optional restricted text, survey creation metadata now known | Answer is a proxy for submission; creation date alone does not reveal score |

No Cancelled, Refunded, InventoryChanged or PaymentSettled event is supported. Final order_status is available only to a retrospective audit view at end of replay, explicitly separate from simulated state. Estimated and shipping-limit dates are deadlines, not observed events; shipping_limit_date is excluded from MVP state/features because provenance and extreme 2020 values are unresolved. A timer-generated overdue alert is a derived rule event marked as such, never a source observation.

Payment rows remain in restricted retrospective reconciliation. An optional assumed snapshot release at approval is an alternative design requiring a visible assumption flag; it is not the recommended MVP and cannot support settlement timelines.

## Quality and Ambiguous Events

1. Preserve immutable row locator (file hash, data-row number) and quality reason. Exact duplicate source records would share a semantic key; conflicts require quarantine, not last-write-wins.
2. Reject review answers before purchase; keep the rest of an order usable. Review uniqueness is (review_id,order_id), verified locally; review_id alone repeats 814 times. Keep all valid review observations; choose first valid answer for the primary target.
3. Carrier times before purchase (166) or before a known approval (1,359) are suspicious; handoff after recorded delivery (23) also conflicts. Suppress the conflicting handoff from operational progression, retain a quality record. These counts overlap and must not be summed. Do not discard a valid purchase/delivery pair solely for a bad handoff.
4. Valid delivery without earlier approval/carrier remains observed delivery with incomplete history. Six canceled snapshots have delivery timestamps; expose the delivery in replay and the contradiction only in retrospective quality analysis. Eight delivered-status orders lack a delivery timestamp: no delivery event can be generated.
5. Missing delivery/review produces unknown label. A passed promise can trigger an overdue rule but must not manufacture a confirmed delivery label. No terminal-state assertion for unfinished orders.
6. For multiple sellers, the order-level delivery is not per-seller fulfillment evidence. One order risk score aggregates basket features; avoid copying the same risk into false independent shipment outcomes.

## Clock, Ordering and Recovery

Run identity includes dataset hash manifest, event mapping version, scenario flag, model/policy versions, initial cutoff, speed and source-time interpretation. Original timestamps are naive; preserve a declared `source_naive` clock, never silently convert to UTC or Cairo. Wall processing timestamps are UTC and separate.

Source events have stable semantic IDs derived from source fingerprint, table business key, event type and source timestamp; run_id scopes consumption. Event ordering is (event_time, event-type priority, order_id, source locator). At a timestamp batch, reveal all accepted events, resolve review-before/equal-delivery eligibility, then score. Cross-order histories use strictly earlier event time. Thus tie order cannot change features or cause a review already known at delivery to be predicted.

Progress is two watermarks: **released-through** and **fully-processed-through**. Dashboard's consistent as-of defaults to the latter. Release cursor advances atomically with committed ingestion; processing cursor advances after all jobs at that timestamp reach success or explicit terminal failure. Event retries are at-least-once, with unique consumption keys giving idempotent effects. Never claim global exactly-once delivery.

Start creates an immutable run configuration; pause drains the current timestamp batch; resume continues the same cursor; step processes one full timestamp batch while paused; speed changes only wall pacing; reset creates a new run, preserving the old one. No records or labels are shared across run namespaces. A crash resumes from the last committed cursor and retries pending jobs. A deterministic error pauses with diagnostic, not silent skip. Operator-approved quarantine is logged under a new policy version if it changes behavior.

## Warm Start and Holdout Boundaries

For a June 1 final replay, reconstruct all permitted pre-cutoff events into warm state without generating retrospective test predictions. Orders purchased in May can deliver in June; they are not M1 June test orders, but can be M2 June delivery-time test cases. Pending pre-cutoff orders remain visible. Reference entities appear on first historical use, not all at run start. Never preload future labels or whole-period seller/customer aggregates.

The scheduler may read the truth vault to schedule future events, but API, scoring worker, rules and UI cannot mount/query it. A separate cutoff-authorized offline trainer may read training-era truth only through a materialization policy. Future event payloads never enter operational tables or queue until release. Scheduler private cursor metadata must not expose future payloads through status APIs.

Training/validation/test dates and maturity rules are in [model design](04_ML_MODEL_DESIGN.md). Follow-up continues to October 30 for observed labels while score cohort ends August 27. Late snapshot orders are excluded from forecasting continuity, not deleted from historical audit. Label availability and evaluation cohorts remain independent of cursor completion.

## Conceptual Storage and Grain

| Logical object | Proposed key / grain | Ownership and purpose |
|---|---|---|
| source manifest / raw tables | file hash + row locator; natural keys preserved | Offline read-only vault; no application permissions |
| replay_runs / checkpoints | run_id; run_id + consumer | API controls; release and processing watermarks |
| business_events / jobs | run_id + event_id; run_id + event_id + consumer | API ingestion ledger and durable pending work |
| orders_current and entity snapshots | run_id + order_id / entity key | API-owned released state; not sufficient alone for historical as-of queries |
| released item rows | run_id + order_id + order_item_id | API-owned assumed placement snapshot |
| released reviews | run_id + review_id + order_id | API-owned observations with answer time |
| curated order facts | run_id + order_id + valid_from | Reconstructable from ledger; no raw children fan-out |
| zip reference | ZIP prefix + reference version | Optional robust median from exact-deduplicated in-box observations, ambiguous states flagged; unknown fallback |
| feature_snapshots | run_id + order_id + model family + as_of + feature version | Worker-owned immutable model inputs and availability metadata |
| model_versions | model family + artifact digest | Offline registry; declared feature/target and fit cutoffs |
| predictions | run_id + order_id + model version + scoring moment | Worker-owned immutable outputs including failed/skipped status |
| actual_outcomes | run_id + order_id + target version + source event | Label matcher; availability timestamp and quality |
| evaluation_matches | prediction_id + outcome_id | Immutable linkage, eligibility and horizon evidence |
| alerts / alert actions | run_id + order_id + policy + prediction; action_id | Rules generate, API owns operator acknowledgement audit |
| weekly_sales_mart / forecasts | run_id + week; run_id + origin + target week + model | Released complete-week facts and immutable forecast vintages |
| metric_definitions / quality_runs | version; run_id + check + cutoff | Definition lineage, aggregate quality evidence |

Physical schema, indexes and DDL are deferred. All child relations include run_id in joins; no accidental cross-run FK. Event ledger plus versioned snapshots supports past-as-of reads; querying the latest orders_current is not valid for an earlier cutoff. Payment snapshots are intentionally absent from replay operational tables in MVP.

## Geography and Financial Rules

Prefer customer state. Optional ZIP lookup removes exact repeated observations, flags 31 broad-Brazil-box outliers (screening heuristic, not a national boundary), computes per-prefix median valid coordinates and flags 8 multi-state prefixes. Unmatched addresses remain in metrics with unknown geography. Great-circle distance is a proxy only. Preserve zero-prefixed ZIP text.

Aggregate payments and items independently by order before reconciliation; use decimal cents in a future implementation. The audit used floating point with tolerance >0.010001 to avoid roundoff at one cent. Do not derive customer revenue or historical paid-state from undated payment rows.

## Scenario Isolation

A scenario clones an explicitly labeled base run into a different scenario namespace with synthetic event provenance. Its metrics, forecasts and alerts are never merged into genuine holdout evaluation. Perturbing inputs demonstrates rule/model sensitivity only; delivery dates do not causally change unless explicitly synthetic, and synthetic outcomes are not ground truth.

## Assumptions / Evidence / Open Questions

Evidence: [audit](01_DATASET_AUDIT.md) verifies missing transitions, duplicate IDs and chronology defects. Assumptions: placement snapshots and naive source clock. Open questions: independent confirmation of timestamp/timezone semantics and whether optional geography is worth complexity. Replay fidelity is limited by the source regardless of architecture.
