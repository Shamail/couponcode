---
document_id: ADR-001
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-00
related_documents:
  - TECH-01
  - TECH-03
  - TECH-04
---

# ADR-001: Postgres + PostGIS over DynamoDB

## 1. Executive Summary

This ADR documents the decision to use Postgres with PostGIS for spatial workloads, rather than DynamoDB with geohash-based access patterns. The decision prioritizes query expressiveness, transactional correctness, and developer velocity.

## 2. Context

Frictionless relies on spatial queries (nearby users, heatmaps, redemption proximity) and transactional integrity (single-use redemptions). The platform also needs predictable performance under frequent writes from location check-ins.

## 3. Decision

Use **Postgres + PostGIS** as the primary datastore for spatial queries and transactional flows.

## 4. Alternatives Considered

| Requirement | DynamoDB | PostGIS | Decision |
| --- | --- | --- | --- |
| Spatial queries | Geohash (manual) | ST_DWithin (native) | PostGIS |
| Aggregation | Client-side | SQL GROUP BY | PostGIS |
| ACID transactions | Limited | Full | PostGIS |
| Developer familiarity | Low | High (SQL) | PostGIS |

## 5. Consequences

**Positive**
- Single query for "users within 500m"
- Heatmap aggregation in ~50ms with indexes
- Reliable redemption transactions (unique constraint + locks)

**Tradeoffs**
- Vendor operational awareness (mitigated by standard SQL and migrations)
- Partitioning required for sustained high write rates (TECH-01)

## 6. Related Documents

**Dependencies**
- TECH-00: Section 2

**Related Specs**
- TECH-01: Section 2
- TECH-03: Section 2
- TECH-04: Section 5

**Implementation Guides**
- TECH-09: Section 2

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Engineering Lead | Updated check-in terminology |
| 1.0 | 2026-01-30 | Engineering Lead | Initial ADR for spatial datastore |
