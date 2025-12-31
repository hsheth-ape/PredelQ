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
| **Metabase** | ⏳ Ready to connect |

---

## What's Deployed

### Database Schema (Supabase)

| Layer | Tables | Status |
|-------|--------|--------|
| **Master Data** | oems, asset_models, customers, accounts, assets | ✅ 15 accounts |
| **Transactions** | payments, repayment_schedule, delinquency_history | ✅ 63 payments |
| **Raw Data** | raw_telemetry | ✅ Schema ready |
| **Metrics** | metric_definitions, metric_values | ✅ 14 metrics, 37 values |
| **Indicators** | indicator_definitions, indicator_values | ✅ 6 indicators, 15 values |
| **Events** | event_definitions, events | ✅ 12 event types, 10 events |
| **Operational** | asset/account/customer_current_state | ✅ All populated |
| **POIs** | pois, poi_visit_patterns | ✅ 15 POIs |

### Mock Data Profile

| Category | Count | Notes |
|----------|-------|-------|
| Healthy accounts | 5 | Good utilization, on-time payments |
| At-risk accounts | 5 | Declining utilization, late payments |
| Delinquent accounts | 5 | DPD 5-45, missed payments |
| Open events | 7 | Across all categories |

### Metabase Views Ready

| View | Purpose |
|------|---------|
| `v_portfolio_overview` | High-level portfolio KPIs |
| `v_account_risk_dashboard` | Account-level risk scores and context |
| `v_asset_telemetry_dashboard` | Asset utilization and location |
| `v_open_events` | Active events requiring action |
| `v_event_timeline` | Full event audit trail |
| `v_indicator_trends` | Indicator score history |
| `v_poi_summary` | All detected/registered POIs |
| `v_collection_priority` | Prioritized collection queue |

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

### GitHub
| Key | Value |
|-----|-------|
| Repo | `hsheth-ape/PredelQ` |

### Modal
| Key | Value |
|-----|-------|
| Bootstrap Gateway | `https://hsheth-ape--gibbon-bootstrap-run.modal.run` |

---

## Metabase Connection Settings

When connecting Metabase, use:

```
Host: db.rhvuyeiughnyapwykinu.supabase.co
Port: 5432
Database: postgres
Username: postgres
Password: 33SuYQUeuWyBYpHb6qSZEg
SSL: Required
```

---

## Key Demo Scenarios

### 1. Portfolio Health
- Query `v_portfolio_overview`
- Shows: 15 accounts, ₹20.18L AUM, 26.7% delinquency rate

### 2. Early Warning Detection
- Query `v_account_risk_dashboard` WHERE risk_category = 'At Risk'
- Shows: Accounts with declining utilization + payment stress
- Context explains: "Utilization dropped 35%, no base visit in 8 days"

### 3. Event Investigation
- Query `v_open_events` ORDER BY severity DESC
- Drill into event context JSON for full story
- Shows indicator movement, metric comparisons, recommendations

### 4. Collection Priority
- Query `v_collection_priority`
- Shows: DPD accounts with last known location, risk score

### 5. Asset Location Intelligence
- Query `v_poi_summary` WHERE entity_type = 'asset'
- Shows: Detected base, work, charging locations
- Confidence scores, visit patterns

---

## Next Steps

1. **Connect Metabase** to Supabase
2. **Build demo dashboards** using the views
3. **Client demo** for concept validation
4. **Sprint 1**: Real data ingestion pipeline
