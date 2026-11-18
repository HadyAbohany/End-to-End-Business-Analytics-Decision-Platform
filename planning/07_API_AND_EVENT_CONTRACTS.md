# Proposed API and Event Contracts

Descriptive contracts only: no endpoints, schemas or services have been implemented. [Architecture](06_SYSTEM_ARCHITECTURE.md) owns runtime boundaries; [replay](05_REPLAY_AND_DATA_DESIGN.md) owns event semantics.

## Interface and Authorization

Browser → Streamlit → FastAPI is synchronous HTTP on localhost. All reads require run context. Viewer reads aggregates, analyst reads redacted order detail, operator controls replay, offline ML reviewer promotes artifacts through a restricted workflow. Scheduler has ingestion-only credentials; worker has its bounded DB role; neither UI nor worker has raw-vault access. Local deployment is single-user by default with role simulation explicitly labeled; multi-user identity is an extension, not claimed implemented security.

| Method and proposed path | Request / response semantics | Failure / permission |
|---|---|---|
| POST /runs | dataset manifest ID, cutoff, immutable model/policy versions, mode → run_id and Idle | Operator; reject unknown artifacts and source-clock policy |
| POST /runs/{id}/commands | command start/pause/resume/step/speed/reset, expected revision, idempotency key → command receipt | Operator; 202 accepted is not applied; 409 invalid transition/stale revision; reset returns a new run |
| GET /runs/{id} | state, speed, released-through, processed-through, queue count, last failure | No future scheduled payloads; state revision included |
| POST /internal/runs/{id}/events | canonical source event or timestamp batch → accepted/duplicate/quarantined receipt | Scheduler only; 422 invalid, 409 same identity/different payload, 503 transient DB failure |
| GET /runs/{id}/kpis | cutoff ≤ processed watermark, cohort interval, metric IDs, mode → values, denominators, coverage, definitions | Reject unavailable future cutoff; final-status mode explicitly retrospective |
| GET /runs/{id}/orders/{order_id} | released observations, risk results, assumptions and reason flags | No raw customer identity or future fields |
| GET /runs/{id}/alerts | bounded page, status, policy, cutoff | Stable pagination cursor bound to run/filter |
| POST /runs/{id}/alerts/{alert_id}/actions | acknowledge/dismiss, reason, expected revision → action receipt | Operator/authorized analyst; immutable audit; not a customer communication |
| GET /runs/{id}/forecasts | origin and horizon → versioned point/interval, target week and availability | Origin ≤ cutoff; no mixing future-vintage forecasts |
| POST /internal/runs/{id}/forecast-jobs | closed-week origin, model version → durable job key | Scheduler; duplicate origin/version returns same job |
| GET /runs/{id}/model-health | volumes, score status, cohort age, known-label coverage, matured metrics | No unsupported “accuracy” before labels arrive |
| GET /health/live and /health/ready | process alive; DB/artifact compatibility readiness | Readiness failure blocks scoring, does not invent zero-risk results |

MVP inference is an internal callable component, not a public endpoint. An optional expanded POST /internal/inference/{model-family} has an immutable feature snapshot reference, schema version, as_of, artifact digest and request ID. It returns probability or duration, model version and explanation status; it never builds features by querying future source records. Forecasting has a separate batch interface.

## Canonical Event Envelope

| Field | Proposed type and rule |
|---|---|
| schema_version | version string; unknown major rejected |
| event_id | deterministic opaque string from source identity; immutable |
| run_id | opaque run identifier, validated against receiving route |
| event_type | OrderPlaced / OrderApproved / CarrierHandoff / OrderDelivered / ReviewSubmitted; derived/synthetic types separately namespaced |
| aggregate_id | order_id, never customer_unique_id |
| event_time | source naive timestamp + explicit source-clock policy; not mislabeled UTC |
| recorded_at | UTC wall receipt time, set by ingestion |
| source | manifest digest, file identity, source row locator; restricted in public response |
| provenance | observed / assumed_snapshot / derived_rule / synthetic; field-level assumptions when mixed |
| payload | allowlisted fields for event type; future columns rejected |
| payload_digest | integrity fingerprint; duplicate identity with differing digest is a conflict |
| correlation_id / causation_id | link command/event → job → prediction → alert |
| quality_flags | structured reason identifiers, never silent repaired values |

Example design sketch in prose: a run R accepts OrderPlaced for order O at source time 2018-06-04 10:00:00. Its payload contains the purchase timestamp and references plus an assumed placement basket. There is **no** delivery timestamp or final status. A later OrderDelivered carries the actual delivery time and causes a separate outcome record. These symbols are illustrative, not fabricated source records.

## Prediction and Forecast Contracts

A risk result contains prediction_id, run_id, order_id, feature_snapshot_id, feature_schema_version, artifact digest, model family, target version, prediction moment, probability in [0,1], created_at, as_of, cohort limitation, policy version and explanation status. A score status is one of scored, skipped_ineligible, failed_dependency or failed_validation. Null score with a status is never interpreted as low risk.

A forecast contains run, origin, target week boundaries, integer horizon, nonnegative point forecast, optional lower/upper bounds, interval method/nominal coverage, artifact version, training cutoff and complete-week input watermark. Actuals join by target week after closure, not by forecast creation week.

## Delivery Guarantees and Failure Contract

Ingestion atomically writes event, released state and job entries in PostgreSQL. The HTTP receipt acknowledges that transaction only. Worker leases jobs, persists feature/prediction/alert effects with uniqueness keys, then completes consumption; retry cannot duplicate effects. For multi-step work, each stage has its own idempotent key. Backoff is bounded and terminal failures enter a visible dead-letter state. Pause on systematic schema incompatibility; do not conceal failed records to advance a healthy-looking watermark.

Optional Redis adds a transactional outbox relay; database commit then direct publish without an outbox is rejected because crashes can lose events. Relay duplicates are expected. Consumers acknowledge after effect commit. Remote inference timeout permits retry using the same immutable features and request identity; no model version fallback without explicit policy and metadata.

Schema changes: additive optional fields are minor versions; changed target meaning, time interpretation or required fields are major. Raw column spelling remains in lineage. Unknown optional fields can be ignored by an explicitly compatible reader, but future-outcome payload fields are denied irrespective of version.

## Assumptions / Evidence / Open Questions

Evidence: source has no native event IDs or processing logs, so the envelope is a proposed design. Assumptions: local single-user environment and one primary DB. Open questions: multi-user authentication and remote hosting are optional owner scope, not necessary for the historical demo. No real customer contact endpoint is authorized or planned.
