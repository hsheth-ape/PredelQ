# PredelQ - Current State

> Last Updated: 2025-12-30

## What is PredelQ?

**Pre-delinquency management product for vehicle asset financing.**

Detects early warning signals from telemetry and financial data to enable intervention before delinquency occurs.

### The Problem
- Current models predict delinquency at 90% confidence
- But only 5-8 days before the event — too late to act meaningfully
- Goal: Detect faint signals at 30-90-180 days with actionable lead time

### The Framework
```
Raw Data → Metrics → Indicators (1-5) → Events → Actions
```

| Layer | Description |
|-------|-------------|
| **Metrics** | Standardized measures (atomic, aggregate, composite) |
| **Indicators** | Scored 1-5 (1=healthy, 5=at risk) |
| **Events** | State changes or threshold crossings |
| **Actions** | Interventions (later scope) |

---

## Project Status

| Aspect | Status |
|--------|--------|
| **Phase** | Sprint 0 - Architecture |
| **Sprint** | 0 |
| **Blocker** | None |

---

## Scale

| Entity | Count |
|--------|-------|
| Assets | ~3,000 |
| Loan Accounts | ~2,000+ |
| Telemetry | Per-minute records |
| Historical Labels | Delinquency events exist |

---

## Tech Stack

| Layer | Tool | Status |
|-------|------|--------|
| Database | Supabase | ✓ Project created |
| Compute | Modal | ✓ Gateway working |
| Output | Metabase | Not connected |
| Repo | GitHub | ✓ Ready |

---

## Credentials & Endpoints

> **Note:** Secrets provided at session start. Do not commit to repo.

### GitHub
| Key | Value |
|-----|-------|
| Repo | `hsheth-ape/PredelQ` |
| Username | `hsheth-ape` |

### Supabase
| Key | Value |
|-----|-------|
| Project ID | `rhvuyeiughnyapwykinu` |
| Project URL | `https://rhvuyeiughnyapwykinu.supabase.co` |
| Dashboard | https://supabase.com/dashboard/project/rhvuyeiughnyapwykinu |
| Region | us-east-1 |
| Management API | *Provide at session start* |

### Modal
| Key | Value |
|-----|-------|
| Bootstrap Gateway | `https://hsheth-ape--gibbon-bootstrap-run.modal.run` |

### Metabase
| Key | Value |
|-----|-------|
| URL | *Not configured* |

---

## What Exists

### Infrastructure
- [x] GitHub repo with templates
- [x] Supabase project (predelq-test)
- [x] Modal gateway verified
- [ ] Database schema deployed
- [ ] Metabase connected
- [ ] Pipelines scaffolded

### Documentation
- [x] Architecture inventory (SPRINT_0_INVENTORY.md)
- [x] Project definition (this file)
- [ ] Execution plan (SPRINT_0_PLAN.md)

---

## Data Sources (Expected)

| Source | Data | Granularity |
|--------|------|-------------|
| Telemetry | Location, SOC, odometer, speed | Per minute |
| Payments | Amounts, dates, status, bounces | Per transaction |
| Loan Details | Amount, tenure, rate, schedule | Per account |
| Customer | Basic info | Per customer |
| Asset | Make, model, year | Per vehicle |

---

## Open Questions

1. How does raw data currently land? (API push, file drop, DB sync?)
2. What format is telemetry in? (JSON, CSV, direct DB?)
3. Are there existing tables we need to migrate from?
4. Metabase instance — existing or new?

---

## Next Steps

1. Review SPRINT_0_INVENTORY.md architecture
2. Answer open questions about data sources
3. Create SPRINT_0_PLAN.md with migration scripts
4. Deploy schema to Supabase
5. Load sample data
