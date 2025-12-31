# Sprint 0: Architecture Inventory

> **Purpose:** Design document for PredelQ data architecture
> **Status:** Draft for Review
> **Created:** 2025-12-30

---

## Overview

PredelQ is a pre-delinquency management product for vehicle asset financing. It detects early warning signals from telemetry and financial data to enable intervention before delinquency occurs.

### Current State
- 90% confidence at 5-8 days before delinquency
- Goal: Similar confidence at 30-90-180 days

### Scale
- 3,000 assets (EVs and vehicles)
- 2,000+ loan accounts
- Minute-level telemetry data
- Labeled historical delinquency events

### Tech Stack
| Layer | Tool |
|-------|------|
| Database | Supabase |
| Compute | Modal |
| Output | Metabase |

---

## Data Architecture

### Conceptual Model

```
Raw Data → Metrics → Indicators → Events → Actions
                         ↓
                    (1-5 score)
```

### Schema Layers

```
┌─────────────────────────────────────────────────────────────┐
│ MASTER DATA (dimensions)                                    │
├─────────────────────────────────────────────────────────────┤
│ customers, accounts, assets, oems, asset_models             │
│ entity_identifiers                                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ TRANSACTION DATA                                            │
├─────────────────────────────────────────────────────────────┤
│ payments, repayment_schedule, delinquency_events            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ RAW DATA                                                    │
├─────────────────────────────────────────────────────────────┤
│ raw_telemetry, raw_telemetry_archive                        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ METRIC/INDICATOR/EVENT SYSTEM                               │
├─────────────────────────────────────────────────────────────┤
│ metric_definitions, metric_values                           │
│ indicator_definitions, indicator_values                     │
│ event_definitions, events                                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ PIPELINE SYSTEM                                             │
├─────────────────────────────────────────────────────────────┤
│ pipeline_definitions, pipeline_runs, pipeline_watermarks    │
│ model_registry                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Schema Definitions

### Master Data

#### customers
```sql
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    status TEXT NOT NULL DEFAULT 'active',  -- active, inactive, churned
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### accounts
```sql
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL REFERENCES customers(id),
    status TEXT NOT NULL DEFAULT 'active',  -- active, delinquent, closed, written_off
    loan_amount DECIMAL(12,2),
    tenure_months INTEGER,
    interest_rate DECIMAL(5,4),
    disbursement_date DATE,
    maturity_date DATE,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### assets
```sql
CREATE TABLE assets (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id),
    asset_model_id UUID REFERENCES asset_models(id),
    status TEXT NOT NULL DEFAULT 'active',  -- active, repossessed, sold
    year_of_manufacture INTEGER,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### oems
```sql
CREATE TABLE oems (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    country TEXT,
    created_at TIMESTAMPTZ DEFAULT now()
);
```

#### asset_models
```sql
CREATE TABLE asset_models (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    oem_id UUID NOT NULL REFERENCES oems(id),
    name TEXT NOT NULL,
    vehicle_type TEXT,  -- car, scooter, bus
    battery_capacity_kwh DECIMAL(6,2),
    created_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(oem_id, name)
);
```

#### entity_identifiers
```sql
CREATE TABLE entity_identifiers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type TEXT NOT NULL,  -- customer, account, asset
    entity_id UUID NOT NULL,
    identifier_type TEXT NOT NULL,  -- customer_ref, vin, registration, device_id
    identifier_value TEXT NOT NULL,
    source TEXT,  -- oem, customer, internal, rto
    is_primary BOOLEAN DEFAULT false,
    valid_from DATE DEFAULT CURRENT_DATE,
    valid_to DATE,
    created_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(entity_type, identifier_type, identifier_value)
);

CREATE INDEX idx_entity_identifiers_lookup 
ON entity_identifiers(identifier_value);

CREATE INDEX idx_entity_identifiers_entity 
ON entity_identifiers(entity_type, entity_id);
```

---

### Transaction Data

#### payments
```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id),
    due_date DATE NOT NULL,
    paid_date DATE,
    amount_due DECIMAL(12,2) NOT NULL,
    amount_paid DECIMAL(12,2),
    payment_method TEXT,
    status TEXT NOT NULL,  -- paid, partial, missed, bounced
    bounce_reason TEXT,
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_payments_account ON payments(account_id, due_date);
```

#### repayment_schedule
```sql
CREATE TABLE repayment_schedule (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id),
    installment_number INTEGER NOT NULL,
    due_date DATE NOT NULL,
    principal_component DECIMAL(12,2),
    interest_component DECIMAL(12,2),
    total_due DECIMAL(12,2) NOT NULL,
    created_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(account_id, installment_number)
);
```

#### delinquency_events (source data, not our generated events)
```sql
CREATE TABLE delinquency_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id),
    event_type TEXT NOT NULL,  -- dpd_30, dpd_60, dpd_90, default
    triggered_at TIMESTAMPTZ NOT NULL,
    resolved_at TIMESTAMPTZ,
    days_past_due INTEGER,
    outstanding_amount DECIMAL(12,2),
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_delinquency_account ON delinquency_events(account_id);
```

---

### Raw Data

#### raw_telemetry
```sql
CREATE TABLE raw_telemetry (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL REFERENCES assets(id),
    ts TIMESTAMPTZ NOT NULL,
    lat DECIMAL(10,7),
    lon DECIMAL(10,7),
    soc DECIMAL(5,2),  -- state of charge %
    odometer DECIMAL(12,2),
    speed DECIMAL(6,2),
    is_charging BOOLEAN,
    raw_payload JSONB,  -- original data for audit
    ingested_at TIMESTAMPTZ DEFAULT now()
);

-- Partition by month for archival
CREATE INDEX idx_telemetry_asset_ts ON raw_telemetry(asset_id, ts);
CREATE INDEX idx_telemetry_ts ON raw_telemetry(ts);
```

#### raw_telemetry_archive
```sql
-- Same structure, for data > 90 days
CREATE TABLE raw_telemetry_archive (
    LIKE raw_telemetry INCLUDING ALL
);
```

---

### Metric/Indicator/Event System

#### metric_definitions
```sql
CREATE TABLE metric_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    display_name TEXT,
    description TEXT,
    entity_type TEXT NOT NULL,  -- asset, account, customer
    value_type TEXT NOT NULL,  -- float, int, pct, bool
    aggregation_type TEXT NOT NULL,  -- atomic, aggregate, composite
    time_granularity TEXT,  -- hourly, daily, weekly
    computation_logic TEXT,  -- description or reference
    depends_on UUID[],  -- array of metric_definition ids
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### metric_values
```sql
CREATE TABLE metric_values (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    metric_id UUID NOT NULL REFERENCES metric_definitions(id),
    entity_type TEXT NOT NULL,
    entity_id UUID NOT NULL,
    value DECIMAL(18,6),
    computed_at TIMESTAMPTZ DEFAULT now(),
    period_start TIMESTAMPTZ,
    period_end TIMESTAMPTZ,
    pipeline_run_id UUID  -- which run created this
);

CREATE INDEX idx_metric_values_lookup 
ON metric_values(metric_id, entity_type, entity_id, period_end DESC);

CREATE INDEX idx_metric_values_entity 
ON metric_values(entity_type, entity_id);
```

#### indicator_definitions
```sql
CREATE TABLE indicator_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    display_name TEXT,
    description TEXT,
    metric_id UUID REFERENCES metric_definitions(id),
    aggregation_type TEXT NOT NULL,  -- atomic, aggregate, composite
    scoring_method TEXT NOT NULL,  -- static_rules, std_dev, ml_model
    scoring_config JSONB,  -- thresholds, model reference, etc.
    depends_on UUID[],  -- for composite indicators
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### indicator_values
```sql
CREATE TABLE indicator_values (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    indicator_id UUID NOT NULL REFERENCES indicator_definitions(id),
    entity_type TEXT NOT NULL,
    entity_id UUID NOT NULL,
    score INTEGER NOT NULL CHECK (score >= 1 AND score <= 5),
    underlying_value DECIMAL(18,6),
    computed_at TIMESTAMPTZ DEFAULT now(),
    period_start TIMESTAMPTZ,
    period_end TIMESTAMPTZ,
    pipeline_run_id UUID
);

CREATE INDEX idx_indicator_values_lookup 
ON indicator_values(indicator_id, entity_type, entity_id, period_end DESC);
```

#### event_definitions
```sql
CREATE TABLE event_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    display_name TEXT,
    description TEXT,
    indicator_id UUID REFERENCES indicator_definitions(id),
    trigger_type TEXT NOT NULL,  -- threshold, delta, trend
    trigger_config JSONB NOT NULL,  -- threshold=4, delta=+2, etc.
    severity INTEGER CHECK (severity >= 1 AND severity <= 5),
    is_actionable BOOLEAN DEFAULT false,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### events
```sql
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_definition_id UUID NOT NULL REFERENCES event_definitions(id),
    entity_type TEXT NOT NULL,
    entity_id UUID NOT NULL,
    indicator_value_id UUID REFERENCES indicator_values(id),
    triggered_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    acknowledged_at TIMESTAMPTZ,
    resolved_at TIMESTAMPTZ,
    notes TEXT,
    pipeline_run_id UUID
);

CREATE INDEX idx_events_entity ON events(entity_type, entity_id);
CREATE INDEX idx_events_triggered ON events(triggered_at DESC);
CREATE INDEX idx_events_unresolved ON events(resolved_at) WHERE resolved_at IS NULL;
```

---

### Pipeline System

#### pipeline_definitions
```sql
CREATE TABLE pipeline_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    display_name TEXT,
    description TEXT,
    type TEXT NOT NULL,  -- ingest, transform, metric, indicator, event
    schedule_type TEXT NOT NULL,  -- cron, watermark, triggered, manual
    schedule_config TEXT,  -- cron expression or null
    source_table TEXT,
    target_table TEXT,
    entity_type TEXT,
    depends_on UUID[],  -- pipeline_definition ids
    script_path TEXT,  -- pipelines/metrics/daily_km.py
    config JSONB,  -- batch_size, lookback_days, etc.
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

#### pipeline_runs
```sql
CREATE TABLE pipeline_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    pipeline_id UUID NOT NULL REFERENCES pipeline_definitions(id),
    status TEXT NOT NULL,  -- pending, running, success, failed, partial
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    
    -- Performance metrics
    duration_ms INTEGER,
    records_in INTEGER,
    records_out INTEGER,
    records_failed INTEGER,
    bytes_processed BIGINT,
    
    -- Computed
    throughput_per_sec DECIMAL(12,2),
    success_rate DECIMAL(5,4),
    
    error_message TEXT,
    run_metadata JSONB,
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_pipeline_runs_pipeline ON pipeline_runs(pipeline_id, started_at DESC);
CREATE INDEX idx_pipeline_runs_status ON pipeline_runs(status) WHERE status = 'running';
```

#### pipeline_watermarks
```sql
CREATE TABLE pipeline_watermarks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    pipeline_id UUID NOT NULL REFERENCES pipeline_definitions(id),
    entity_type TEXT,
    entity_id UUID,
    last_processed_ts TIMESTAMPTZ,
    last_run_id UUID REFERENCES pipeline_runs(id),
    updated_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(pipeline_id, entity_type, entity_id)
);
```

#### model_registry
```sql
CREATE TABLE model_registry (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    version TEXT NOT NULL,
    description TEXT,
    storage_path TEXT NOT NULL,  -- models/delinquency_30d_v1.pkl
    trained_at TIMESTAMPTZ,
    training_data_start DATE,
    training_data_end DATE,
    metrics JSONB,  -- accuracy, precision, recall, f1
    is_active BOOLEAN DEFAULT false,
    created_by TEXT,
    created_at TIMESTAMPTZ DEFAULT now(),
    UNIQUE(name, version)
);
```

---

## Pipeline Architecture

### Execution Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────┐
│ Raw Ingest  │ ──▶ │ Metric Calc │ ──▶ │ Indicator   │ ──▶ │ Event   │
│ (cron)      │     │ (watermark) │     │ Scoring     │     │ Detect  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────┘
      │                   │                   │                  │
      ▼                   ▼                   ▼                  ▼
   Supabase           Supabase            Supabase           Supabase
```

### Pipeline Types

| Type | Trigger | Example |
|------|---------|---------|
| Cron | Fixed schedule | `raw_ingest` — pull from source every 15 min |
| Watermark | Records to process | `metric_daily_km` — run until caught up |
| Triggered | Upstream completes | `indicator_scoring` — after metrics done |
| Manual | Human initiates | `backfill_historical` — one-time jobs |

### Watermark Processing Pattern

```python
def run_watermark_pipeline(pipeline_name):
    pipeline = get_pipeline(pipeline_name)
    
    for entity in get_entities(pipeline.entity_type):
        watermark = get_watermark(pipeline.id, entity.id)
        last_ts = watermark.last_processed_ts or MIN_DATE
        
        records = query_source(
            pipeline.source_table,
            entity_id=entity.id,
            after=last_ts
        )
        
        if not records:
            continue
            
        results = process(records, pipeline.config)
        write_results(pipeline.target_table, results)
        
        update_watermark(
            pipeline.id,
            entity.id,
            last_ts=records[-1].ts
        )
```

---

## Data Retention

| Data Type | Hot Storage | Archive | Rationale |
|-----------|-------------|---------|-----------|
| Raw telemetry | 90 days | Forever | Audit needs, ML retraining |
| Metrics | Forever | N/A | Small, needed for trends |
| Indicators | Forever | N/A | Small, needed for analysis |
| Events | Forever | N/A | Core product data |
| Pipeline runs | 1 year | Archive | Performance analysis |

### Archive Pipeline

Weekly job:
1. Move `raw_telemetry` rows older than 90 days to `raw_telemetry_archive`
2. Vacuum `raw_telemetry` table
3. Log archive stats

---

## Open Decisions

| Decision | Options | Recommendation |
|----------|---------|----------------|
| Telemetry partitioning | Monthly tables vs single + archive | Single + archive (simpler) |
| Metric storage | Wide table vs registry | Registry (flexible) |
| Pipeline orchestration | Modal cron vs external scheduler | Modal cron (simpler) |

---

## Next Steps

1. Review this inventory with stakeholder
2. Create Sprint 0 execution plan with:
   - Supabase migration scripts
   - Initial pipeline scaffolding
   - Metabase connection
3. Load sample data for validation
