---
document_id: DATA-01
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-01
  - TECH-06
related_documents:
  - PRD-01
  - PRD-02
  - TECH-02
  - TECH-03
  - TECH-04
---

# DATA-01: Schema Dictionary

## 1. Executive Summary

This document consolidates the canonical data constraints for Frictionless. It is the single source of truth for validation rules that appear across PRDs, API contracts, and the Neon schema.

Engineering should treat this as the authoritative reference when implementing Zod schemas, database constraints, and client-side form validation.

## 2. Canonical Field Dictionary

| Field | Constraint | Source | Enforced At |
| --- | --- | --- | --- |
| `lat` | -90 to 90 | TECH-06 | Zod + CHECK |
| `lng` | -180 to 180 | TECH-06 | Zod + CHECK |
| `location_fuzzing_decimals` | 3 decimals (~111m precision) | TECH-02 | Client + API |
| `location_ttl_minutes` | 30 minutes | TECH-02 | Cron + Query filters |
| `deal.title` | max 50 chars | PRD-02 | Client + API + DB |
| `deal.discount_type` | enum: `percentage` / `fixed` / `bogo` | TECH-06 | Zod + DB |
| `deal.discount_value` | > 0 (percentage must be 1-100) | DATA-01 | Client + API |
| `deal.radius_meters` | max 500 for redemption radius | TECH-06 | API + DB |
| `heatmap.radius` | max 2000 meters | TECH-06 | API |
| `heatmap.resolution` | min 25m, max 200m | TECH-06 | API |
| `heatmap.window` | max 60 minutes | TECH-06 | API |
| `qr_code` | max 64 chars | TECH-01 | DB |
| `redemption.status` | `initiated`  `verified`  `expired`  `cancelled` | TECH-01 | DB |
| `(user_id, deal_id)` | unique | TECH-01 | DB |
| `deal.categories` | max 3 | TECH-06 | API |
| `deal.images` | max 5 URLs | TECH-06 | API |

## 3. Time Constants

| Constant | Value | Rationale | References |
| --- | --- | --- | --- |
| Location check-in interval | 30 seconds | Balanced freshness + battery | PRD-01, TECH-02 |
| Location TTL | 30 minutes | Live map relevance | TECH-02 |
| QR token TTL | 60 seconds | Prevent replay | TECH-04 |
| Redemption expiry | 5 minutes | In-store validity | TECH-01 |
| Heatmap cache | 25 seconds | Avoid redundant fetch | TECH-03 |
| Heatmap polling | 30 seconds | UX freshness | DES-01 |

## 4. Validation Ownership Matrix

| Layer | Primary Responsibility | Tooling | Notes |
| --- | --- | --- | --- |
| Client | Basic input limits | React Native forms | Fail fast for UX |
| API | Full schema validation | Zod | Reject invalid payloads |
| Database | Integrity constraints | CHECK + UNIQUE + FK | Last line of defense |

## 5. Usage Guidance

- Any new field constraint must be added here first, then propagated to API schemas and DB migrations.
- When a constraint changes, update TECH-06 (API) and TECH-01 (schema) in the same PR.
- Treat this document as the canonical truth when conflicts appear elsewhere.

## 6. Related Documents

**Dependencies**
- TECH-01: Section 2
- TECH-06: Section 3

**Related Specs**
- TECH-02: Section 2
- TECH-03: Section 2
- TECH-04: Section 2
- PRD-01: Section 3
- PRD-02: Section 3

**Implementation Guides**
- IMPL-02: Section 4

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Engineering Lead | Updated check-in terminology |
| 1.0 | 2026-01-30 | Engineering Lead | Canonical schema dictionary |
