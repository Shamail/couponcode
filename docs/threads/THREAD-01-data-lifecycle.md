---
document_id: THREAD-01
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Documentation Lead
dependencies:
  - PRD-01
  - TECH-02
  - TECH-06
  - TECH-01
  - TECH-03
  - DES-01
related_documents:
  - DATA-01
  - TECH-00
  - TECH-08
---

# THREAD-01: The Life of a Check-in

## 1. Executive Summary

This thread traces a single location check-in from the Buyer app to the Seller heatmap. It connects PRD behavior, API contracts, storage rules, and map rendering so teams can reason about latency, privacy, and failure modes end-to-end.

## 2. Thread Map (End-to-End)

```
Expo Location
  -> Privacy Fuzzing (3 decimals)
    -> API Gateway (POST /check-in)
      -> Lambda Validation
        -> PostGIS Ingestion
          -> TTL Cleanup
            -> Heatmap Aggregation
              -> Seller App Render
```

## 3. Stage-by-Stage Trace

| Stage | Primary Source | Key Details |
| --- | --- | --- |
| Location capture | PRD-01 | 30s interval, 50m distance threshold |
| Privacy fuzzing | TECH-02 | Truncate to 3 decimals (~111m) |
| API validation | TECH-06 | Zod schema, rate limit 60/min |
| PostGIS write | TECH-01 | SP-GiST index, insert via function |
| TTL cleanup | TECH-02 | 30-minute expiration, 5-min cron |
| Heatmap query | TECH-03 | ST_DWithin + k-anonymity (k=3) |
| Mapbox render | DES-01 | Heatmap layer, 30s polling |

## 4. Failure and Edge Cases

- **No location permission**: Buyer map shows CTA and feature gating (PRD-01).
- **Stale check-in**: Client timestamp >5 minutes flagged but recorded (TECH-06).
- **High load**: Heatmap queries should move to replica when read RPM spikes (TECH-01).
- **Mapbox outage**: Switch to Lite Mode list view (DES-01).

## 5. Observability Hooks

| Indicator | Source | Alert |
| --- | --- | --- |
| `CheckInIngested` count drop | TECH-08 | Alert if down >30% in 10 min |
| Heatmap latency p95 | TECH-03 | Alert if >500ms |
| TTL cleanup lag | TECH-02 | Alert if >2 runs missed |

## 6. Related Documents

**Dependencies**
- PRD-01: Section 3
- TECH-02: Section 2
- TECH-06: Section 3
- TECH-01: Section 2
- TECH-03: Section 2
- DES-01: Section 1

**Related Specs**
- DATA-01: Section 2
- TECH-08: Section 4

**Implementation Guides**
- IMPL-02: Section 3

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Documentation Lead | Updated terminology and check-in naming |
| 1.0 | 2026-01-30 | Documentation Lead | End-to-end data lifecycle thread |
