# PredelQ - Ways of Working

> Last Updated: 2025-12-30

---

## Project Overview

PredelQ is a pre-delinquency management system that detects early warning signals from vehicle telemetry and financial data.

**Goal:** Move delinquency prediction from 5-8 days (current) to 30-90-180 days.

---

## Core Framework

### The Pipeline

```
Raw Data → Metrics → Indicators → Events → Actions
```

### Definitions

#### Metric
A standardized computed measure from raw data.

| Type | Definition | Example |
|------|------------|---------|
| **Atomic** | Single, simple dataset | Days since last payment |
| **Aggregate** | Rollups, running averages | 30-day avg daily km |
| **Composite** | Multiple datasets or metrics combined | Liquidity ratio |

Metrics are arbitrary numbers — they have no inherent scale.

#### Indicator
A scored signal derived from metrics. **Always on a 1-5 scale.**

| Score | Meaning |
|-------|---------|
| 1 | Most favorable (healthy) |
| 2 | Good |
| 3 | Neutral / Watch |
| 4 | Concerning |
| 5 | Least favorable (at risk) |

Scoring methods:
- Static rules (thresholds)
- Standard deviation splits
- ML model output

#### Event
A state change or threshold crossing on an indicator.

Triggers:
- Indicator crosses a threshold (e.g., reaches 4)
- Indicator changes significantly (e.g., jumps +2)
- Trend detected (e.g., 3 consecutive increases)

Events have:
- Severity (1-5)
- Actionable flag (yes/no)

#### Action
An intervention matched to an event. **(Post-MVP scope)**

---

## Entity Hierarchy

```
Customer
    └── Account (loan)
            └── Asset (vehicle)
```

- Metrics can attach to any level
- Indicators roll up through the hierarchy
- Events surface at the appropriate level

---

## Data Layers

| Layer | Contents | Retention |
|-------|----------|-----------|
| **Master Data** | Customers, accounts, assets, OEMs, models | Forever |
| **Transaction Data** | Payments, schedules, delinquency events | Forever |
| **Raw Data** | Telemetry (location, SOC, odometer) | 90 days hot, archive forever |
| **Computed Data** | Metrics, indicators, events | Forever |
| **Pipeline Data** | Definitions, runs, watermarks | 1 year hot |

---

## Pipeline Patterns

### Processing Types

| Type | Trigger | Use Case |
|------|---------|----------|
| **Cron** | Fixed schedule | Raw data ingestion |
| **Watermark** | Records to process | Metric computation |
| **Triggered** | Upstream completes | Indicator scoring |
| **Manual** | Human initiates | Backfills, one-time jobs |

### Watermark Processing
Pipelines track where they left off per entity. This handles:
- Gaps in data (3 days missing, then data arrives)
- Variable processing rates
- Failure recovery

---

## Naming Conventions

### Tables
- Snake_case: `metric_values`, `pipeline_runs`
- Plural for collections: `customers`, `assets`
- Singular for definitions: `metric_definitions` (not metrics_definition)

### Metrics
- Format: `{measure}_{aggregation}_{window}`
- Examples: `daily_km`, `soc_avg_7d`, `payment_bounce_rate_30d`

### Indicators
- Format: `{domain}_{signal}_score`
- Examples: `utilization_health_score`, `payment_regularity_score`

### Pipelines
- Format: `{stage}_{entity}_{metric}`
- Examples: `ingest_telemetry`, `metric_daily_km`, `indicator_utilization`

---

## Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| **Database** | Supabase (Postgres) | Storage, views, functions |
| **Compute** | Modal | Pipeline execution, ML training |
| **Output** | Metabase | Dashboards, queries |
| **Code** | GitHub | Version control, pipeline scripts |

---

## Session Protocol

### Starting a Session
1. Read CURRENT.md for project state
2. Read BACKLOG.md for priorities
3. Check recent session logs
4. State session goal
5. Ask "Where did we leave off?"

### Ending a Session
1. Summarize what was done
2. Update relevant docs
3. Create session log
4. State next steps
5. Ask "Any lessons learned?"

---

## Claude Roles

| Role | Responsibilities | Does NOT Do |
|------|-----------------|-------------|
| **Planning** | Architecture, sprint planning, backlog, review | Write production code |
| **Execution** | Build features, deploy, fix bugs | Make architecture decisions |
| **UX** | Design interfaces, user flows | Write backend code |

### Handoffs
- Planning creates INVENTORY.md → Review → Creates PLAN.md → Execution implements
- Execution completes → Planning reviews → Merge/iterate

---

## Quality Gates

### Before Merging
- [ ] Code executes without error
- [ ] Tests pass (when applicable)
- [ ] Matches execution plan
- [ ] No hardcoded credentials
- [ ] Pipeline runs tracked in DB

### Before Sprint Close
- [ ] All P0 items complete
- [ ] Documentation updated
- [ ] Session logs written
- [ ] Lessons captured

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why Bad | Do Instead |
|--------------|---------|------------|
| Hardcoding credentials | Security risk | Use environment/config |
| Skipping watermarks | Lose progress on failure | Always track state |
| One table per metric | Schema bloat, hard to query | Use registry pattern |
| Ignoring gaps in data | Silent data loss | Watermark-based processing |
| Skipping session logs | Lose context | Always document |
