---
document_id: TECH-07
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-00
related_documents:
  - IMPL-02
  - TECH-08
  - TECH-09
---

# TECH-07: Deployment & Environments

## 1. Executive Summary

This document defines how Frictionless is deployed using SST v3 and AWS. It covers environment setup, secrets management, deployment commands, and rollback procedures.

The goal is safe, repeatable deployments with minimal operational overhead.

## 2. Environment Strategy

| Environment | Purpose | AWS Account | Notes |
| --- | --- | --- | --- |
| dev | Local and feature testing | Shared dev | Fast iteration, low cost |
| staging | Pre-release validation | Shared staging | Mirrors prod configs |
| prod | Live traffic | Production | Strict access control |

## 3. SST Deployment Flow

### 3.1 Prerequisites
- AWS credentials with least-privilege access
- SST v3 installed
- Neon database provisioned
- Secrets stored in AWS SSM or SST secrets

### 3.2 Core Commands

```bash
pnpm sst dev
pnpm sst deploy --stage staging
pnpm sst deploy --stage prod
```

### 3.3 Rollback

- SST supports redeploying the previous Git commit
- Use tagged releases for repeatable rollbacks

```bash
git checkout <previous_commit>
pnpm sst deploy --stage prod
```

## 4. Secrets Management

| Secret | Storage | Notes |
| --- | --- | --- |
| DATABASE_URL | SST Secret / SSM | Neon connection string |
| JWT_PRIVATE_KEY | SST Secret | RS256 signing |
| MAPBOX_TOKEN | SST Secret | Map tiles |
| SENTRY_DSN | SST Secret | Error monitoring |

## 5. Domain & TLS

- API domain: `api.frictionless.ma`
- Use ACM for TLS certs
- CloudFront handles termination and caching

## 6. Deployment Checklist

- [ ] Verify migrations applied (TECH-09)
- [ ] Run unit + integration tests (IMPL-02)
- [ ] Validate environment variables
- [ ] Confirm monitoring alerts active (TECH-08)
- [ ] Deploy to staging, run smoke tests
- [ ] Deploy to prod

## 7. Related Documents

**Dependencies**
- TECH-00: Section 2

**Related Specs**
- TECH-08: Section 3
- TECH-09: Section 2

**Implementation Guides**
- IMPL-02: Section 4

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Engineering Lead | Initial deployment guide |
