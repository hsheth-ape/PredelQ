# PredelQ - Backlog

> Last Updated: 2025-12-30

## Priority Legend

| Priority | Meaning |
|----------|---------|
| P0 | Must have for MVP |
| P1 | Should have for MVP |
| P2 | Nice to have |
| P3 | Future consideration |

---

## Sprint 0: Foundation (Current)

| ID | Item | Priority | Status |
|----|------|----------|--------|
| S0-1 | Create GitHub repo | P0 | ✅ Done |
| S0-2 | Set up Supabase project | P0 | ✅ Done |
| S0-3 | Verify Modal gateway | P0 | ✅ Done |
| S0-4 | Design data architecture | P0 | ✅ Done |
| S0-5 | Deploy schema to Supabase | P0 | Pending |
| S0-6 | Create initial pipelines scaffold | P0 | Pending |
| S0-7 | Connect Metabase | P1 | Pending |
| S0-8 | Load sample data | P1 | Pending |

---

## Sprint 1: Data Ingestion (Upcoming)

| ID | Item | Priority | Status |
|----|------|----------|--------|
| S1-1 | Define data source connections | P0 | Not started |
| S1-2 | Build telemetry ingest pipeline | P0 | Not started |
| S1-3 | Build payments ingest pipeline | P0 | Not started |
| S1-4 | Build master data loaders | P0 | Not started |
| S1-5 | Implement data cleaning stage | P0 | Not started |
| S1-6 | Set up archival pipeline | P1 | Not started |

---

## Sprint 2: Metrics Engine (Future)

| ID | Item | Priority | Status |
|----|------|----------|--------|
| S2-1 | Define initial metric set | P0 | Not started |
| S2-2 | Build metric computation pipelines | P0 | Not started |
| S2-3 | Implement watermark processing | P0 | Not started |
| S2-4 | Create metric validation checks | P1 | Not started |
| S2-5 | Build metrics dashboard in Metabase | P1 | Not started |

---

## Sprint 3: Indicators & Events (Future)

| ID | Item | Priority | Status |
|----|------|----------|--------|
| S3-1 | Define initial indicator set | P0 | Not started |
| S3-2 | Implement scoring logic (static rules) | P0 | Not started |
| S3-3 | Build event detection engine | P0 | Not started |
| S3-4 | Create alerts/notifications | P1 | Not started |
| S3-5 | Build events dashboard | P1 | Not started |

---

## Sprint 4: ML Integration (Future)

| ID | Item | Priority | Status |
|----|------|----------|--------|
| S4-1 | Prepare training dataset | P0 | Not started |
| S4-2 | Train initial delinquency model | P0 | Not started |
| S4-3 | Integrate model scoring | P0 | Not started |
| S4-4 | Implement model registry workflow | P1 | Not started |
| S4-5 | Build model performance dashboard | P1 | Not started |

---

## Backlog (Unscheduled)

| ID | Item | Priority | Notes |
|----|------|----------|-------|
| B-1 | Action matching engine | P2 | Post-MVP |
| B-2 | Customer interaction data ingestion | P2 | Depends on data access |
| B-3 | Multi-tenancy support | P3 | If needed |
| B-4 | Real-time streaming | P3 | Not needed for MVP |
| B-5 | Base location detection | P1 | Key indicator |
| B-6 | Maturity curve computation | P1 | Per-customer baseline |

---

## Icebox (Parked Ideas)

| Item | Reason Parked |
|------|---------------|
| Mobile app alerts | Focus on Metabase first |
| Customer self-service portal | Not MVP scope |

---

## Notes

- Sprint scope subject to change based on data source discovery
- P0 items must complete before sprint closes
- P1 items can slip to next sprint if needed
