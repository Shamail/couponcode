---
document_id: METRICS-02
version: 1.0
status: Final
priority: P2
last_updated: 2026-01-30
owner: Data Lead
dependencies:
  - METRICS-01
related_documents:
  - METRICS-03
  - TECH-08
---

# METRICS-02: Dashboards

## 1. Executive Summary

This document defines the dashboard suite for tracking the Frictionless marketplace. Dashboards are organized by stakeholder: leadership, product, growth, operations, and engineering.

Each dashboard is built from the KPI framework and includes alert thresholds for proactive response.

## 2. Dashboard Inventory

### 2.1 Executive Dashboard
**Audience:** Leadership
- MSR (Monthly Successful Redemptions)
- Active Buyers / Merchants
- GMV Influenced
- MRR and Take Rate
- Weekly trend deltas

### 2.2 Marketplace Health Dashboard
**Audience:** Product + Ops
- Deal Supply by corridor
- Claim Rate vs Redemption Rate
- Heatmap Density by hour
- Merchant Activation funnel

### 2.3 Growth Dashboard
**Audience:** Growth + Marketing
- Install → Signup → Opt-in → First View
- Referral share rate
- CAC by channel
- Activation cohorts (D1, D7)

### 2.4 Ops Dashboard
**Audience:** Field Operations
- Merchant onboarding progress
- Top merchants by redemptions
- Support ticket volume
- Merchant churn alerts

### 2.5 Engineering Reliability Dashboard
**Audience:** Engineering
- API latency (p95, p99)
- Error rates by endpoint
- Lambda cold starts
- Heatmap generation latency

## 3. Alerting Rules

| Alert | Threshold | Action |
| --- | --- | --- |
| Redemption drop | -25% WoW | Trigger merchant success outreach |
| Deal supply drop | -30% week over week | Onboard new merchants |
| API error rate | >2% over 30 min | Page engineering on-call |
| Heatmap latency | p95 > 2s | Investigate query performance |

## 4. Implementation Notes

- All dashboards source the analytics warehouse or Postgres read replicas.
- Daily snapshot tables should be generated for executive reporting.
- Use a shared metric dictionary to avoid definition drift.

## 5. Related Documents

**Dependencies**
- METRICS-01: Section 3

**Related Specs**
- METRICS-03: Section 2
- TECH-08: Section 3

**Implementation Guides**
- IMPL-02: Section 4

## 6. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Data Lead | Initial dashboard definitions |
