# Proposed System Architecture

Every diagram is a **proposed design**, not a deployed or validated system. Dataset scale (~99k orders, ~1M geography rows) does not justify a distributed platform. The main challenge is temporal correctness, not throughput. See [audit](01_DATASET_AUDIT.md), [replay](05_REPLAY_AND_DATA_DESIGN.md) and [contracts](07_API_AND_EVENT_CONTRACTS.md).

## Architecture Decision

Recommend a modular backend with PostgreSQL, FastAPI, Streamlit, one scoring/batch worker, and a **separate replay scheduler**. Separating the scheduler is necessary because it reads future truth; scoring must not share its raw-data access. Delivery/CX inference and rules are distinct modules in the worker, not separate services. Forecasting is a scheduled job. PostgreSQL event/jobs tables replace Redis in MVP. Offline fitting is a separately authorized lifecycle, never part of request serving.

scikit-learn pipelines are sufficient for baselines and initial candidates; XGBoost is optional after evidence of benefit. Versioned artifact directories plus a manifest suffice initially; MLflow is an optional experiment UI. Streamlit avoids React build/auth complexity; React is justified only for richer controls or team expertise. Compose is future local packaging, not evidence of production readiness. GitHub Actions is a future code/contract-check runner, not automatic model promotion or cloud deployment.

## 1. System Context — C4 Level 1

```mermaid
flowchart LR
    E[Executive and analyst] -->|Inspect metrics and forecasts| P[Historical decision-support platform]
    L[Logistics and CX] -->|Inspect risks and acknowledge suggestions| P
    O[Replay operator] -->|Control virtual time| P
    M[ML reviewer] -->|Review baseline evidence and versions| P
    S[Olist historical CSV snapshot] -->|Offline source with provenance caveats| P
    P -->|Educational observations and predictions| E
    P -->|Suggested triage only| L
```

No external business system or customer messaging integration exists. Publisher provenance and local dataset version remain unverified; the local hashes identify the analyzed bytes.

## 2. Containers and Service Boundaries — C4 Level 2, MVP

```mermaid
flowchart TB
    B[Local browser] -->|HTTP| UI[Streamlit dashboard and controls]
    UI -->|Synchronous read and command HTTP| API[FastAPI public boundary]
    API -->|Control state and transaction writes| DB[(PostgreSQL released state and jobs)]
    VAULT[(Read-only future truth vault)] -->|Private read| SIM[Replay scheduler process]
    SIM -->|Poll authorized run commands| DB
    SIM -->|Timestamp-qualified events via internal HTTP| API
    DB -->|Lease committed jobs asynchronously| W[Scoring and batch worker]
    W -->|Features predictions rules outcomes| DB
    ART[(Versioned model artifacts)] -->|Read frozen model| W
    DB -->|As-of reads| API
    API -->|Versioned view results| UI
    TRAIN[Offline cutoff-authorized training process] -->|Approved artifact only| ART
    VAULT -->|Restricted historical cutoff policy| TRAIN
```

The API owns ingestion/state and read interfaces; worker owns ML/policy outputs. Shared PostgreSQL is acceptable for a modular MVP, with logical schema/role ownership. Scheduler's DB permission is limited to control/checkpoint metadata; event application must pass ingestion. Raw vault is mounted only to scheduler and separately authorized offline training, never API/UI/worker. One process per risk model adds no demonstrated value.

## 3. Backend Components — C4 Level 3

```mermaid
flowchart LR
    CLIENT[UI or scheduler] --> ROUTES[Route and role validation]
    subgraph API[FastAPI process]
        ROUTES --> READ[As-of analytics query module]
        ROUTES --> CONTROL[Replay command state machine]
        ROUTES --> VALID[Event allowlist and provenance validator]
        VALID --> TX[Idempotent state and job transaction]
    end
    READ --> DB[(Owned PostgreSQL schemas)]
    CONTROL --> DB
    TX --> DB
    subgraph WORKER[Separate scoring worker process]
        LEASE[Job lease and retry manager] --> FX[Point-in-time feature builder]
        FX --> SCORE[Frozen delivery or CX pipeline]
        SCORE --> RULE[Versioned business rules]
        RULE --> SAVE[Idempotent result commit]
        LEASE --> LABEL[Outcome matcher and delayed evaluation]
        LEASE --> FORECAST[Closed-week forecast handler]
    end
    DB --> LEASE
    DB --> FX
    SAVE --> DB
    LABEL --> DB
    FORECAST --> DB
    MODELS[(Read-only artifacts)] --> SCORE
    MODELS --> FORECAST
```

In-process calls inside API/worker are synchronous functions. API ingestion and worker execution are asynchronous across committed jobs. M2 eligibility is resolved before features; all events at a timestamp are visible while cross-order feature history remains strictly earlier. Label matching never changes the saved input snapshot or probability.

## 4a. Source Relational ERD — Observed Conceptual Cardinalities

```mermaid
erDiagram
    CUSTOMERS ||--|| ORDERS : customer_id
    ORDERS ||--o{ ORDER_ITEMS : order_id
    ORDERS ||--o{ ORDER_PAYMENTS : order_id
    ORDERS ||--o{ ORDER_REVIEWS : order_id
    PRODUCTS ||--o{ ORDER_ITEMS : product_id
    SELLERS ||--o{ ORDER_ITEMS : seller_id
    CATEGORY_TRANSLATION o|--o{ PRODUCTS : optional_category_mapping
    CUSTOMERS {
        string customer_id PK
        string customer_unique_id
        string customer_zip_code_prefix
    }
    ORDERS {
        string order_id PK
        string customer_id FK
        datetime order_purchase_timestamp
        datetime order_delivered_customer_date "nullable"
        string order_status "snapshot only"
    }
    ORDER_ITEMS {
        string order_id PK,FK
        int order_item_id PK
        string product_id FK
        string seller_id FK
    }
    ORDER_PAYMENTS {
        string order_id PK,FK
        int payment_sequential PK
        decimal payment_value
    }
    ORDER_REVIEWS {
        string review_id PK
        string order_id PK,FK
        datetime review_answer_timestamp
        int review_score
    }
    GEOLOCATION_RAW {
        string geolocation_zip_code_prefix "not unique"
        decimal geolocation_lat
        decimal geolocation_lng
    }
    PRODUCTS {
        string product_id PK
        string product_category_name "nullable or unmapped"
    }
    SELLERS {
        string seller_id PK
        string seller_zip_code_prefix
    }
    CATEGORY_TRANSLATION {
        string product_category_name PK
        string product_category_name_english
    }
```

Customer/order is exactly 1:1 in this local snapshot; real-person repeats use customer_unique_id, not that FK. Review PK is composite. Raw geography is deliberately unconnected: a direct join fans out. Translation has 610 null product categories and 13 nonnull unmatched products. All raw declarations describe observed candidate keys, not existing physical constraints.

## 4b. Proposed Curated and Operational ERD

```mermaid
erDiagram
    REPLAY_RUNS ||--o{ BUSINESS_EVENTS : scopes
    REPLAY_RUNS ||--o{ ORDERS_CURRENT : isolates
    BUSINESS_EVENTS ||--o{ JOBS : triggers
    ORDERS_CURRENT ||--o{ ORDER_FACT_VERSIONS : history
    ORDERS_CURRENT ||--o{ RELEASED_ITEMS : contains
    ORDERS_CURRENT ||--o{ RELEASED_REVIEWS : observes
    ORDERS_CURRENT ||--o{ FEATURE_SNAPSHOTS : scores_at_time
    FEATURE_SNAPSHOTS ||--o{ PREDICTIONS : produces
    MODEL_VERSIONS ||--o{ PREDICTIONS : governs
    PREDICTIONS ||--o{ ALERTS : suggests
    PREDICTIONS ||--o{ EVALUATION_MATCHES : evaluated
    ACTUAL_OUTCOMES ||--o{ EVALUATION_MATCHES : labels
    ORDERS_CURRENT ||--o{ ACTUAL_OUTCOMES : matures
    ZIP_REFERENCE o|--o{ RELEASED_CUSTOMERS : optional_zip
    ZIP_REFERENCE o|--o{ RELEASED_SELLERS : optional_zip
    RELEASED_CUSTOMERS ||--o{ ORDERS_CURRENT : identifies
    RELEASED_SELLERS ||--o{ RELEASED_ITEMS : sells
    REPLAY_RUNS ||--o{ WEEKLY_MART : aggregates
    REPLAY_RUNS ||--o{ FORECASTS : isolates
    MODEL_VERSIONS ||--o{ FORECASTS : generates
    REPLAY_RUNS {
        string run_id PK
        string source_manifest
        datetime processed_through
    }
    BUSINESS_EVENTS {
        string run_id PK,FK
        string event_id PK
        datetime event_time
        string provenance
    }
    ORDERS_CURRENT {
        string run_id PK,FK
        string order_id PK
        string observed_state
    }
    FEATURE_SNAPSHOTS {
        string snapshot_id PK
        string run_id FK
        datetime as_of
        string schema_version
    }
    PREDICTIONS {
        string prediction_id PK
        string snapshot_id FK
        string model_version FK
        decimal probability "nullable on failure"
    }
    FORECASTS {
        string run_id PK,FK
        datetime origin PK
        date target_week PK
        string model_version PK,FK
    }
```

This is a logical design, not migration DDL. Diagram keys abbreviate details: all order-scoped joins must include run_id; artifact versions are global immutable references. Weekly marts derive from event-qualified order facts; forecast actual matching uses target week and run, not a direct per-order FK. Source-to-curated lineage retains all nine raw tables even when payments and geography remain offline reference/reconciliation inputs.

## 5. Data Flow and Future-Truth Isolation

```mermaid
flowchart LR
    RAW[(Nine immutable CSVs)] --> AUD[Read-only audit and manifest]
    RAW --> V[(Private truth vault)]
    V --> S[Scheduler and field-release gate]
    S --> EV[Released observed or assumed events]
    EV --> OP[(Operational event ledger and state)]
    OP --> CUR[Quality-controlled order facts]
    CUR --> MART[As-of KPI and weekly marts]
    CUR --> FX[As-of feature snapshots]
    FX --> PRED[Model predictions]
    EV --> LAB[Arrived outcome labels]
    PRED --> EVAL[Delayed eligibility-aware evaluation]
    LAB --> EVAL
    MART --> UI[Dashboard via API]
    PRED --> RULE[Suggestions and alerts]
    RULE --> UI
    EVAL --> UI
    SYN[Isolated synthetic scenario run] --> SUI[Separate scenario dashboard]
```

Layers are logical, not separate database products. “Bronze” truth must not be an application-readable schema. Synthetic experiments have a separate run and never feed genuine evaluation.

## 6. Event-to-Decision Sequence and Delayed Labels

```mermaid
sequenceDiagram
    participant S as Replay scheduler
    participant A as Ingestion API
    participant D as PostgreSQL
    participant W as Worker
    participant M as In-process risk pipeline
    participant R as Rule module
    participant U as Dashboard via API
    S->>A: OrderPlaced with permitted fields only
    A->>D: Atomic event + state + job commit
    A-->>S: Durable receipt and cursor acknowledgement
    W->>D: Lease committed job at timestamp barrier
    W->>D: Read point-in-time inputs and earlier history
    W->>M: Frozen feature snapshot and model version
    M-->>W: Probability or explicit failure
    W->>R: Apply locked capacity and threshold policy
    R-->>W: Suggestion or no alert
    W->>D: Idempotent prediction + alert + completion
    U->>A: Read at processed watermark
    A->>D: As-of query
    D-->>A: Released results
    A-->>U: Versioned dashboard view
    S->>A: OrderDelivered later
    A->>D: Commit observed delivery and jobs
    W->>D: Match M1 prediction to arrived label
    opt No review known by delivery
        W->>M: Score M2 using now-known delivery experience
        M-->>W: Conditional negative-review probability
        W->>D: Save M2 prediction and policy result
    end
    S->>A: ReviewSubmitted at answer timestamp
    A->>D: Commit valid review and label job
    W->>D: Match first valid answer to eligible M2 prediction
    W->>D: Update label coverage and delayed metrics
```

Scoring is asynchronous; durable ingestion does not imply completed prediction. No-review cases never acquire a fabricated negative label.

## 7. Offline Training, Registry and Separate Forecast Batch

```mermaid
flowchart TB
    HIST[(Cutoff-authorized historical data)] --> SPLIT[Versioned chronological folds and label maturity]
    SPLIT --> FIT[Fit preprocessing and candidate baselines offline]
    FIT --> VAL[Validation and calibration before test]
    VAL --> REVIEW[Reviewer promotion gate]
    REVIEW --> REG[(Immutable artifacts and metadata)]
    REG --> RISK[Event-scoring worker]
    REG --> BATCH[Weekly forecast handler]
    MART[(Released complete-week mart)] --> BATCH
    CLOCK[Virtual Monday boundary] -->|Enqueue unique origin job| BATCH
    BATCH --> FORE[(Forecast vintages and intervals)]
    FORE --> DASH[Forecast dashboard via API]
    MART -->|After target week closes| ERR[Horizon-specific error and coverage]
    FORE --> ERR
```

No automatic fitting on holdout data. Weekly lag refresh is inference; retraining is a separate protocol. MLflow, if added, records runs/artifacts; it does not establish temporal validity by itself.

## 8. Proposed Local Deployment

```mermaid
flowchart TB
    USER[Browser on host] -->|localhost 8501 proposed| UI[Streamlit container]
    subgraph NET[Private Compose network - proposed only]
        UI -->|HTTP internal 8000| API[FastAPI container]
        SIM[Replay scheduler container] -->|Internal ingestion HTTP| API
        SIM -->|Control metadata role| PG[(PostgreSQL internal 5432)]
        API --> PG
        WORK[Scoring and batch worker container] --> PG
    end
    RAW[(Host source read-only mount)] -->|Scheduler only| SIM
    MODELS[(Read-only approved artifact mount)] --> WORK
    PG --> DATA[(Persistent DB volume)]
    OFF[Offline training process outside online network] --> MODELS
    RAW -->|Explicit cutoff policy| OFF
```

Ports are proposals, not listeners created here. Bind UI to loopback; API and DB need no host exposure. Scheduler and worker are separate even if they share a code image. No Redis container in MVP. Suggested planning allowance: 8 GB RAM minimum target, 16 GB comfortable, several GB disk for DB/indexes and artifacts; **not measured**. Source files total roughly 126 MB. Avoid loading million-row geography in each process; precompute a versioned reference only in a later authorized preparation stage. Measure actual memory before committing limits. No GPU requirement is justified.

## 9. Replay State and Checkpoint Diagram

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Warming : start and validate manifest
    Warming --> Running : earlier events reconstructed
    Running --> Draining : pause request
    Draining --> Paused : current timestamp processed
    Paused --> Running : resume
    Paused --> Stepping : step timestamp batch
    Stepping --> Paused : effects and checkpoint committed
    Running --> Finished : released schedule drained
    Running --> Failed : unrecoverable error
    Warming --> Failed : invalid source or version
    Stepping --> Failed : batch failure
    Failed --> Paused : diagnosed resumable recovery
    Paused --> NewRun : reset with new run ID
    Finished --> NewRun : reset with new run ID
    Failed --> NewRun : explicit clean restart
    NewRun --> Idle : preserve prior run
```

Speed changes are self-transitions within Running or Paused; they do not alter event order. Commit-before-cursor and per-consumer idempotency permit retries. Finished means all available scheduled observations consumed, not all orders delivered or reviewed.

## 10. Optional Expanded Services

```mermaid
flowchart LR
    UI[React or Streamlit] --> API[Gateway and read API]
    S[Isolated truth scheduler] --> ING[Ingestion service]
    ING -->|Atomic state plus outbox| DB[(PostgreSQL)]
    DB --> RELAY[Outbox relay]
    RELAY --> REDIS[(Redis Streams)]
    REDIS --> FX[Feature and orchestration worker]
    FX -->|Synchronous versioned request| ML[Risk inference service]
    ML -->|Read only| REG[(Artifact registry)]
    ML -->|Response| FX
    FX -->|Persist predictions| DB
    FX --> RULE[Rule module or service]
    RULE --> DB
    REDIS --> LABEL[Outcome evaluation consumer]
    LABEL --> DB
    DB --> API
    FORE[Separate weekly forecast worker] --> DB
    REG --> FORE
```

Only adopt after a measured need for independent release/scaling or team ownership. Inference owns no operational DB writes; orchestration persists outputs. A rule service is optional, not mandatory. Shared DB may remain initially with schema ownership; separate databases multiply consistency and deployment work without present evidence. Redis adds relay/consumer recovery, not better data fidelity.

## Reliability, Security and Verification

Database transactions eliminate the MVP dual-write gap. Worker leases expire safely after crashes; unique keys prevent repeated predictions/alerts. Source/schema errors enter quarantine with counts; missing model artifacts make risk unavailable while analytics continues. Metrics distinguish virtual event lag, wall processing latency and label delay. Log run/event/model/correlation IDs, not review text or raw identifiers in public logs.

Restrict raw mounts, DB roles and model promotion. Use parameterized queries, bounded filters/pagination, localhost bindings and secrets outside source control in any future implementation. Backup manifests, artifacts and DB together; resetting replay is not a backup. Test future-data denial, same-time barriers, duplicate delivery, crash recovery, run isolation, metric reconciliation and delayed labels before a demo. Details: [test strategy](09_ML_OPS_AND_TEST_STRATEGY.md).

## Assumptions / Evidence / Open Questions

Evidence: local data is modest and relational; time anomalies are real. Assumptions: one laptop, a small capstone team and no external integration. Open questions: actual machine budget and team expertise. Diagrams have been authored and structurally inspected as Mermaid source; rendered layout and runtime behavior are not validated in this planning-only phase.
