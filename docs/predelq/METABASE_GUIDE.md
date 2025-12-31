# PredelQ - Metabase Guide

> **Last Updated:** 2025-12-31

---

## Connection Settings

```
Host: db.rhvuyeiughnyapwykinu.supabase.co
Port: 5432
Database: postgres
Username: postgres
Password: 33SuYQUeuWyBYpHb6qSZEg
SSL: Required
```

---

## Available Views

### Portfolio & Risk Views

| View | Purpose |
|------|---------|
| `v_portfolio_overview` | High-level portfolio KPIs |
| `v_account_risk_dashboard` | Account-level risk scores and context |
| `v_collection_priority` | Prioritized collection queue |

### Early Warning Signals (EWS) Views

| View | Purpose | Mockup Component |
|------|---------|------------------|
| `v_ews_risk_summary` | Customer counts by risk category | Summary cards |
| `v_ews_customer_events` | Events grouped by customer and type | Detail table |
| `v_ews_customer_summary` | Event type counts pivoted per customer | Pivot view |

### Asset & Telemetry Views

| View | Purpose |
|------|---------|
| `v_asset_telemetry_dashboard` | Asset utilization and location |
| `v_poi_summary` | All detected/registered POIs |

### Event Views

| View | Purpose |
|------|---------|
| `v_open_events` | Active events requiring action |
| `v_event_timeline` | Full event audit trail |
| `v_indicator_trends` | Indicator score history |

---

## EWS Dashboard Setup

### 1. Summary Cards

Create 3 number cards:

**High Risk Card:**
```sql
SELECT SUM(customer_count) as count
FROM v_ews_risk_summary 
WHERE risk_score >= 4;
```

**Medium Risk Card:**
```sql
SELECT customer_count as count
FROM v_ews_risk_summary 
WHERE risk_score = 3;
```

**Low Risk Card:**
```sql
SELECT SUM(customer_count) as count
FROM v_ews_risk_summary 
WHERE risk_score <= 2;
```

### 2. Customer Events Table

Use `v_ews_customer_events` with grouping:
- Group by: `customer_name` → `event_type` → `event_description`
- Show: `event_count`
- Filter by: `current_risk_flag`

### 3. Customer Summary Pivot

Use `v_ews_customer_summary`:
- Columns: customer_name, current_risk_flag, identity_events, transactional_events, operational_events, behavioural_events, environmental_events, total_events

---

## Risk Categories

| Score | Category | Color |
|-------|----------|-------|
| 5 | Very High | Red |
| 4 | High | Orange |
| 3 | Medium | Yellow |
| 2 | Low | Light Green |
| 1 | Very Low | Green |

---

## Event Types

| Type | Description | Examples |
|------|-------------|----------|
| Identity | Location/base patterns | Base not visited, POI shifts |
| Transactional | Payment/financial events | Bounce, overdue, DPD |
| Operational | Asset usage patterns | Utilization drop, downtime |
| Behavioural | Customer behavior signals | Call patterns, field visits |
| Environmental | External factors | OEM issues, regional trends |
| Observability | System/data health | Telemetry gaps, data quality |

---

## Demo Scenarios

### 1. Portfolio Health
- Query `v_portfolio_overview`
- Shows: Total accounts, AUM, delinquency rate

### 2. Early Warning Detection
- Query `v_ews_customer_summary` 
- Filter: `risk_score >= 3`
- Shows: At-risk customers with event breakdown

### 3. Event Investigation
- Query `v_ews_customer_events`
- Filter by customer
- Shows: All events by type for deep dive

### 4. Collection Priority
- Query `v_collection_priority`
- Shows: DPD accounts ranked by risk

---

## Filters to Add

| Filter | Field | Values |
|--------|-------|--------|
| Risk State | `current_risk_flag` | Very High, High, Medium, Low, Very Low |
| Event Type | `event_type` | Identity, Transactional, Operational, Behavioural, Environmental |
| Customer | `customer_name` | Text search |

