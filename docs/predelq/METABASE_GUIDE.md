# Metabase Connection Guide

## Connection Details

| Setting | Value |
|---------|-------|
| **Database Type** | PostgreSQL |
| **Host** | `db.rhvuyeiughnyapwykinu.supabase.co` |
| **Port** | `5432` |
| **Database** | `postgres` |
| **Username** | `postgres` |
| **Password** | `33SuYQUeuWyBYpHb6qSZEg` |
| **SSL** | Required |

---

## Pre-Built Views

These views are optimized for Metabase dashboards:

### Portfolio Level
| View | Description | Key Columns |
|------|-------------|-------------|
| `v_portfolio_overview` | Single-row KPIs | total_accounts, delinquency_rate_pct, total_outstanding |

### Account Level
| View | Description | Key Columns |
|------|-------------|-------------|
| `v_account_risk_dashboard` | Risk scores + context | risk_category, delinquency_risk_score, risk_message, recommendation |
| `v_collection_priority` | DPD accounts for action | collection_priority, days_past_due, last_lat/lon |

### Asset Level
| View | Description | Key Columns |
|------|-------------|-------------|
| `v_asset_telemetry_dashboard` | Utilization + location | utilization_pct, days_since_base_visit, utilization_message |
| `v_poi_summary` | Detected locations | poi_type, confidence_score, visit_count |

### Events
| View | Description | Key Columns |
|------|-------------|-------------|
| `v_open_events` | Active alerts | severity, title, message, recommendation, days_open |
| `v_event_timeline` | Audit trail | triggered_at, status, title |

### Trends
| View | Description | Key Columns |
|------|-------------|-------------|
| `v_indicator_trends` | Score history | score, previous_score, score_change, trend |

---

## Suggested Dashboard Layout

### Dashboard 1: Executive Overview
1. **Scorecard row**: Total AUM, Outstanding, Delinquency Rate (from v_portfolio_overview)
2. **Pie chart**: Accounts by risk_category (from v_account_risk_dashboard)
3. **Bar chart**: Open events by category (from v_open_events)
4. **Table**: Top 5 at-risk accounts (from v_account_risk_dashboard WHERE risk_category IN ('At Risk', 'Critical'))

### Dashboard 2: Collections Workbench
1. **Table**: v_collection_priority with all columns
2. **Map**: Asset locations (last_lat, last_lon from v_collection_priority)
3. **Filter**: By collection_priority, days_past_due range

### Dashboard 3: Telemetry Intelligence
1. **Scatter plot**: Utilization % vs Days Since Base Visit (from v_asset_telemetry_dashboard)
2. **Table**: Assets with utilization_score >= 4
3. **Map**: Asset last known locations with color by utilization_score

### Dashboard 4: Event Management
1. **Table**: v_open_events with expand for full_context
2. **Timeline**: v_event_timeline
3. **Filter**: By category, severity

---

## Example Queries

### High-Risk Accounts with Context
```sql
SELECT 
    account_number,
    customer_name,
    days_past_due,
    delinquency_risk_score,
    risk_message,
    recommendation
FROM v_account_risk_dashboard
WHERE risk_category IN ('At Risk', 'Critical')
ORDER BY delinquency_risk_score DESC;
```

### Assets Not Visiting Base
```sql
SELECT 
    registration_number,
    customer_name,
    days_since_base_visit,
    base_visit_score,
    base_visit_message,
    days_past_due
FROM v_asset_telemetry_dashboard
WHERE days_since_base_visit > 7
ORDER BY days_since_base_visit DESC;
```

### Event Context Drill-Down
```sql
SELECT 
    event_name,
    customer_name,
    severity,
    title,
    message,
    recommendation,
    full_context
FROM v_open_events
WHERE severity >= 4;
```

---

## Notes

- All `context` and `full_context` columns contain rich JSON with human-readable messages
- The `recommendation` field contains actionable next steps
- Views join across tables so Metabase doesn't need complex SQL
- Filters on `risk_category`, `severity`, `poi_type` work well for drill-downs
