---
document_id: THREAD-03
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Documentation Lead
dependencies:
  - PRD-02
  - TECH-06
  - TECH-01
  - PRD-01
  - METRICS-03
related_documents:
  - OPS-01
  - OPS-04
  - DATA-01
---

# THREAD-03: Deal Lifecycle (Create -> Discover -> Redeem -> Expire)

## 1. Executive Summary

This thread traces a deal from seller creation through buyer discovery, redemption, and expiry. It unifies product requirements, API contracts, database constraints, and analytics tracking.

## 2. Thread Map (End-to-End)

```
Seller Creates Deal
  -> API Validation
    -> DB Persistence
      -> Buyer Discovery
        -> Redemption Flow
          -> Expiration + Analytics
```

## 3. Stage-by-Stage Trace

| Stage | Primary Source | Key Details |
| --- | --- | --- |
| Deal creation | PRD-02 | Title max 50 chars, required fields |
| API validation | TECH-06 | `discount_type` enum, radius caps |
| DB persistence | TECH-01 | `deals` table, active indexes |
| Buyer discovery | PRD-01 | Map + list surface, proximity cues |
| Redemption | THREAD-02 | SafeColor Verification flow |
| Expiry | TECH-01 | `expires_at` enforced at query time |
| Analytics | METRICS-03 | Deal view, claim, redeem events |

## 4. Edge Cases

- **Expired deal still visible**: Ensure API filters `expires_at > NOW()` (TECH-01).
- **High radius abuse**: Cap redemption radius at 500m (TECH-06).
- **Low-quality deals**: Ops review gating for new merchants (OPS-01).

## 5. Metrics and KPIs

| Metric | Source | Target |
| --- | --- | --- |
| Deal view -> claim | METRICS-01 | 20% |
| Claim -> redeem | METRICS-01 | 60% |
| Redemption latency | TECH-06 | <2s p95 |

## 6. Related Documents

**Dependencies**
- PRD-02: Section 3
- TECH-06: Section 4
- TECH-01: Section 4
- PRD-01: Section 3
- METRICS-03: Section 2

**Related Specs**
- DATA-01: Section 2
- OPS-01: Section 3
- OPS-04: Section 2

**Implementation Guides**
- GUIDE-01: Section 3
- GUIDE-02: Section 3

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Documentation Lead | Updated SafeColor terminology |
| 1.0 | 2026-01-30 | Documentation Lead | Deal lifecycle thread |
