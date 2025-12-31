# PredelQ - Current State

> Last Updated: 2025-12-31

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

---

## Project Status

| Aspect | Status |
|--------|--------|
| **Phase** | Sprint 0 - Demoware |
| **Sprint** | 0 |
| **Blocker** | None |
| **Database** | ✅ Schema deployed, mock data loaded |
| **Metabase** | ✅ Connected, views ready |

---

## What's Deployed

### Database Schema (25 tables)

| Layer | Tables | Status |
|-------|--------|--------|
| **Master Data** | oems, asset_models, customers, accounts, assets | ✅ 15 accounts |
| **Transactions** | payments, repayment_schedule, delinquency_history | ✅ 63 payments |
| **Raw Data** | raw_telemetry | ✅ Schema ready |
| **Metrics** | metric_definitions, metric_values | ✅ 14 metrics |
| **Indicators** | indicator_definitions, indicator_values | ✅ 6 indicators |
| **Events** | event_types, event_definitions, events | ✅ 8 types, 12 defs, 7 open |
| **Operational** | asset/account/customer_current_state | ✅ With risk scores |
| **POIs** | pois, poi_visit_patterns | ✅ 15 POIs |

### Event Type Taxonomy

| ID | Type | Customer-Facing | Event Count |
|----|------|-----------------|-------------|
| 8 | Identity | Yes | Location/base signals |
| 3 | Transactional | Yes | Payment/DPD signals |
| 4 | Operational | Yes | Utilization signals |
| 5 | Behavioural | Yes | Customer behavior |
| 6 | Environmental | Yes | External factors |
| 1 | Observability | Yes | System health |
| 2 | Ingestion | No | Data pipeline |

### Customer Risk Scoring

| Score | Category | Count |
|-------|----------|-------|
| 5 | Very High | 1 |
| 4 | High | 2 |
| 3 | Medium | 2 |
| 2 | Low | 0 |
| 1 | Very Low | 10 |

### Metabase Views (11)

| View | Purpose |
|------|---------|
| `v_portfolio_overview` | Portfolio KPIs |
| `v_account_risk_dashboard` | Account-level risk |
| `v_asset_telemetry_dashboard` | Asset utilization |
| `v_open_events` | Active events |
| `v_event_timeline` | Event audit trail |
| `v_indicator_trends` | Indicator history |
| `v_poi_summary` | POI locations |
| `v_collection_priority` | Collection queue |
| `v_ews_risk_summary` | **EWS summary cards** |
| `v_ews_customer_events` | **EWS detail table** |
| `v_ews_customer_summary` | **EWS pivot view** |

---

## Credentials & Endpoints

### Supabase
| Key | Value |
|-----|-------|
| Project ID | `rhvuyeiughnyapwykinu` |
| Project URL | `https://rhvuyeiughnyapwykinu.supabase.co` |
| Dashboard | https://supabase.com/dashboard/project/rhvuyeiughnyapwykinu |
| Region | us-east-1 |
| DB Host | `db.rhvuyeiughnyapwykinu.supabase.co` |
| DB Port | 5432 |
| DB Name | postgres |
| DB User | postgres |
| DB Password | `33SuYQUeuWyBYpHb6qSZEg` |
| Service Key | `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJodnV5ZWl1Z2hueWFwd3lraW51Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc2NzEzNTA5MCwiZXhwIjoyMDgyNzExMDkwfQ.aIEfiuxOXYP7i7FprPvIEWOnWuCnHxlfLuCbgmXlLjs` |
| Management API | `sbp_f69d63cf863a33ba6d9af03a599ec12f94ce7da0` |

### GitHub
| Key | Value |
|-----|-------|
| Repo | `hsheth-ape/PredelQ` |

### Modal
| Key | Value |
|-----|-------|
| Bootstrap Gateway | `https://hsheth-ape--gibbon-bootstrap-run.modal.run` |

---

## Next Steps

1. **Build EWS Metabase dashboard** using the new views
2. **Import full event catalog** (402 events from CSV)
3. **Client demo** for concept validation
4. **Sprint 1**: Real data ingestion pipeline

