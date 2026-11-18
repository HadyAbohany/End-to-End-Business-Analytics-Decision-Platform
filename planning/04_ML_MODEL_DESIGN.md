# ML Model Design and Temporal Evaluation

No training, model selection, score estimation or feature implementation was performed. Counts below establish sample support only. [Audit](01_DATASET_AUDIT.md) is the numerical source.

## Shared Point-in-Time Feature Contract

| Feature family | Exact source / aggregation | Earliest permissible availability | Treatment |
|---|---|---|---|
| Calendar, promised lead days | orders.order_purchase_timestamp, order_estimated_delivery_date | Placement, promise timing assumed | Day/week/calendar; promise-minus-purchase; no final status |
| Buyer region | orders.customer_id → customers.customer_state, customer_zip_code_prefix | Placement snapshot assumption | State primary; ZIP prefix string, no individual identity as model input |
| Basket | items.price, freight_value, product_id, seller_id grouped by order_id | Placement snapshot assumption | Counts, totals, distinct seller/category count; missing-item indicator |
| Product metadata | products.product_category_name, product_weight_g, product_length_cm, product_height_cm, product_width_cm | First referenced placement, static snapshot assumption | Sum weights/volumes, category mix; missing/zero weight flags; fitted imputation on training only |
| Geographic distance | customer/seller ZIP → deduplicated valid geolocation medians | Static reference assumption, first order reference | Optional min/max/mean straight-line distance; missing flag. Not route or transport time |
| Seller experience | prior order events linked by items.seller_id | Only earlier released placements and matured labels | Prior counts, smoothed late rate; multi-seller aggregation min/max/weighted mean; cold-start global training prior |
| Customer history | customer_unique_id linked to prior placements | Strictly earlier released orders | Prior count/recency/value, no future lifetime totals. Omit if complexity adds little |
| Delivered experience (M2 only) | actual delivery − purchase; delivery date − promised date | At valid delivery event | Elapsed days and signed delay; carrier features omitted in MVP due chronology contradictions |
| Payments and shipping deadline | payment_type/installments/value; shipping_limit_date | No reliable event availability | Excluded from MVP predictive inputs; no fabricated payment events. Deadline reaches 2020 in snapshot |
| Forbidden | final order_status, future outcomes, review score/text, future population encodings | Never as predictive inputs for these tasks | Labels separately authorized; fit vocabularies/scalers/encoders only inside training folds |

Metadata name aliases, if later desired: product_name_lenght → product_name_length and product_description_lenght → product_description_length. Preserve original names in source lineage; these optional metadata are excluded from core features.

Feature history uses event time strictly less than score time for cross-order aggregates; own placement fields may be included. Same-time other orders never enter each other's history. All transformations and smoothing priors are fitted before the relevant cutoff. No full-snapshot target encoding. New sellers/products/categories map to explicit unknowns.

## Locked Chronological Protocol

Use 2017-01-01 onward as primary modeling period; sparse 2016 orders are optional warm-up only. T0 = 2018-01-01; validation feature-time window [2018-01-01,2018-04-01); maturation/selection buffer [2018-04-01,2018-06-01); final model freeze T1 = 2018-06-01; final test score window [2018-06-01,2018-08-27). Observation follow-up ends 2018-10-30 exclusive, after the last observed review on October 29. This is an observation bound, not proof all labels have arrived.

At T0, audit upper-bound available pools include 41,202 delivery and 38,997 delivery-time review labels (including sparse 2016); at T1 these are 75,435 and 70,492. Final model pools additionally require valid feature-time cohort, all feature assumptions and mature labels. Do not report these upper bounds as final training counts.

1. Within pre-T0 data, use expanding temporal folds for preprocessing/model development, restricting each fold's labels to those available at its fit cutoff.
2. Freeze candidates at T0, score January–March validation by each model's prediction moment, and evaluate only labels observed before T1. Select hyperparameters, calibration approach and thresholds using this validation, never the final test. The two-month buffer is not a guarantee of full maturity; remaining unknown labels stay censored.
3. Refit the selected pipeline using pre-T1 eligible observations with labels available before T1. Preserve a time-separated calibration subset or cross-fitted temporal predictions; never calibrate on fitted-row probabilities. April–May can enter final fit after policy selection, but cannot serve as an untouched test.
4. Freeze artifact and policy before final replay. No retraining on test labels. Past test outcomes may enter causal rolling history when genuinely released, because that is the specified operational feature; distinguish this from fitting model parameters.
5. Report both fixed-age outcome coverage (7/30/60 days) and all-observed-label results. Compare labeled versus unlabeled populations. Changing to a fixed response-horizon target is a new model version, not silently labeling missing answers positive.

Audit retrospective test support: 18,841 purchases, 18,487 observed deliveries, 672 calendar-late; M2 delivery-time cohort has 18,691 subsequently reviewed deliveries and 1,621 negatives. M2 includes orders purchased before June because its prediction moment is delivery. These counts exclude neither all downstream quality gates nor future missingness. Calendar-late prevalence shifts: Jan–Mar 2,658/20,628 versus Jun–Aug26 672/18,487. Calibration stability is a material risk.

## M1 — Placement-Time Late Delivery

**Owner/decision:** logistics reviews a capacity-limited queue. **Verdict:** conditional GO, primary supervised model. One prediction per order at OrderPlaced. Approval-time scoring would be a separately versioned model; do not mix moments.

**Target:** 1 when delivery calendar date exceeds estimated calendar date, 0 otherwise, among valid observed deliveries with estimate and delivery ≥ purchase. Do not filter by final delivered status in an online cohort. Six delivered timestamps conflict with canceled status; retain a quality flag and report sensitivity excluding these in retrospective evaluation. Missing delivery is unknown, not an on-time negative. The modeled probability is conditional on observed delivery, not universal failure probability.

**Features:** shared placement families, first without ZIP distance/history, then evaluate incremental benefit. Promise availability is unproven; run a predeclared ablation without promise-derived features. If owner rejects that assumption, publish the reduced model and its separate evaluation.

**Candidates:** training prevalence; promised-lead-time bucket rule estimated only on training; regularized logistic pipeline; optional one tree-boosting candidate. Random Forest is an alternative, not another mandatory model. No huge hyperparameter search warranted by ~100k orders.

**Metrics:** PR-AUC compared to contemporaneous prevalence, ROC-AUC secondary, Brier and calibration bins, precision/recall at preselected K, confusion matrix at locked threshold. Temporal-block uncertainty intervals, plus state/seller-newness/lead-time slices with denominators. No invented minimum AUC. Promote only if useful top-K improvement and acceptable calibration hold on validation and survive untouched-test reporting.

**Decision policy:** K from operator daily capacity; until supplied, show a sensitivity table at 1%, 5%, 10% queues, not an optimized business threshold. Explain factors via linear contributions or validated optional SHAP; associations are not causes. Output probability, score status, model/feature versions, as-of, cohort caveat and reason summary.

## M2 — Post-Delivery Negative Review

**Owner/decision:** CX prioritizes optional follow-up. **Verdict:** conditional extension, no obligation to ship. Score valid delivery events only if no valid review answer is already visible. Equal-time reviews suppress scoring by a same-time eligibility check.

**Label:** first valid review by answer time, then review_id tie-break, score ≤2. Pre-purchase answers (63 raw rows) are quarantined; for an affected order, choose the first remaining valid answer. Raw cohort count 91,061/8,612 uses first raw answer and is diagnostic. The additional audit removing pre-purchase answers yields 98,613 valid reviewed orders and 91,064 delivery-time labeled cases, including 8,613 negatives. Other feature-quality exclusions can further reduce the eventual training set. Do not select the worst or latest eventual review. No review text, score, creation date or eventual response latency as features. No-answer orders remain unlabeled.

**Features:** M1 placement attributes plus known delivery duration and delay. No M1 probability initially: actual delivery is already known, so stacking introduces complexity and little clear value. Compare actual-lateness-only rule to logistic and optional boosting. Require incremental benefit over that simple rule.

**Evaluation:** shared temporal protocol with delivery as feature time and answer as label-availability time. PR-AUC, Brier, top-K precision/recall, coverage by delivery age/state/lateness, and selection-bias report. Conditional observed-response population only; cannot infer dissatisfaction among nonresponders. At least 4,769 first answers precede/equal delivery, so “all reviewed orders” is invalid for this prediction moment. No causal claim that contacting customers improves scores.

## M3 — Weekly Recorded Orders

**Owner/decision:** plan likely order workload, not inventory. **Verdict:** conditional GO at company grain, 1-week primary and horizons 2–4 secondary. Monday–Sunday complete weeks; origin Monday 00:00; target count distinct purchased order_id in each subsequent week. No final-status filtering. Forecast total item price separately only after count baseline succeeds.

Stable observed calendar span is weeks ending 2017-01-08 through 2018-08-26 (86 weeks). This does not prove completeness. Exclude sparse opening 2016 and truncated late 2018. No annual seasonality claim with fewer than two complete years. Exclude a final partial week from fitting and scoring rather than extrapolating it as a full week.

**Backtests:** expanding history, first candidate origins from 2017-10-02 once ~39 complete weeks exist; validation origins through March 2018 with all horizons matured before June freeze. Final origins June 4 through July 30, 2018 support all four target weeks ending by August 26. Later origins may be reported only for shorter fully observed horizons, in a separate panel. Freeze selected parameters at June 1; recompute deterministic lag inputs on released weekly observations without refitting during test.

**Features/baselines:** last week, trailing four-week mean, damped trend; calendar and past lag 1–8 and rolling statistics shifted before target. Annual seasonal naive unsupported; optional lag regularized regression or boosting only if consistent gains. SARIMA/Prophet/deep learning are not default requirements and no package is installed. Holiday covariates require documented calendars known before origin; excluded initially.

**Metrics:** MAE, WAPE per horizon, bias and naive-relative skill. sMAPE optional with zero policy. Estimate interval widths from prior rolling-origin residuals only, report empirical coverage and width; few origins limit confidence. No category model fleet; future top-category eligibility requires sufficient continuous support selected before holdout. Forecast records persist origin, horizon, target week, interval, model, training cutoff and run.

## M4 — Descriptive Customer Groups

**Verdict:** GO for rule-based summaries, defer clustering. Compute RFM only at cutoff using customer_unique_id, prior purchases and item value. Frequency is one for 93,099 of 96,096 buyers in full snapshot, so clustering will largely reflect recency/value. Compare simple bands before standardized/log-transformed K-means; fit bands/scalers on pre-cutoff data. If explored, assess stability across cutoffs, silhouette, group sizes and stakeholder interpretation, not “accuracy.” Monthly refresh; no CLV, loyalty or demographic claim. Unknown platform tenure makes recency observational.

## M5 — Delivery Duration

**Verdict:** technically conditional, excluded from MVP because it shares target information and operational use with M1. Target elapsed days delivery − purchase among valid observed deliveries; prediction at placement. Baselines training median and supplied promise lead time; ridge or quantile boosting optional. Shared temporal splits, MAE/RMSE, regional residuals and interval coverage. A separate ETA decision and clear incremental benefit are prerequisites. Undelivered selection remains unresolved; survival modeling would require defensible observation/censoring boundaries not supplied here.

## Assumptions / Evidence / Open Questions

Evidence: audited class counts, cutoff maturity and window support. Assumptions: source wall time, static references and early promise knowledge. Open questions: operational K/costs, minimum acceptable improvement and whether scope allows CX. No predictive performance or calibrated threshold is known until separately authorized implementation and evaluation.
