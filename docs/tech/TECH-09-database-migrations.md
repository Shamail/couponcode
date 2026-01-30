---
document_id: TECH-09
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-01
related_documents:
  - TECH-07
  - IMPL-02
---

# TECH-09: Database Migrations

## 1. Executive Summary

This document defines the migration strategy for the Frictionless Postgres/PostGIS schema. It prioritizes zero-downtime changes and repeatable environments.

All schema changes must be applied through versioned migrations and validated in staging before production deployment.

## 2. Migration Strategy

- Use a migrations folder with timestamped SQL files
- Apply migrations in order using a controlled runner
- Never edit or delete applied migrations
- Prefer additive changes (add columns, avoid destructive drops)

## 3. Migration Workflow

1. Create a new migration file
2. Apply locally
3. Validate in staging
4. Deploy to production

```bash
# Example
pnpm db:migrate:create add-deal-radius
pnpm db:migrate:up --stage staging
pnpm db:migrate:up --stage prod
```

## 4. Zero-Downtime Patterns

| Pattern | Example |
| --- | --- |
| Add column + backfill | Add `merchant_tier`, backfill in batches |
| Dual write | Write to old + new columns temporarily |
| Feature flag | Roll out read paths gradually |

## 5. PostGIS Considerations

- Spatial indexes can be heavy; create during low-traffic windows
- Use `CREATE INDEX CONCURRENTLY` where supported

## 6. Rollback Strategy

- Prefer reversible migrations (create complementary down scripts)
- Keep a snapshot before complex migrations

## 7. Related Documents

**Dependencies**
- TECH-01: Section 3

**Related Specs**
- TECH-07: Section 4

**Implementation Guides**
- IMPL-02: Section 3

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Engineering Lead | Initial migration strategy |
