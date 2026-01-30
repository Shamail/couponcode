---
document_id: TECH-11
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-08
  - OPS-03
related_documents:
  - TECH-07
  - TECH-01
  - TECH-03
  - TECH-04
  - THREAD-05
---

# TECH-11: Failure Modes & Resilience Playbooks

## 1. Executive Summary

This document catalogs the highest-impact failure modes for Frictionless and provides detection signals, mitigation steps, and recovery guidance. It is intended for engineers and on-call responders.

## 2. Failure Mode Index

| Scenario | Detection | Primary Mitigation | Owner |
| --- | --- | --- | --- |
| Neon cold start latency > 3s | Lambda p99 > 3000ms | Keep-warm + skeleton UI | Engineering |
| Mapbox API outage | Tile HEAD failures >2% | Lite Mode list view | Engineering + Design |
| PostGIS query timeout | Lambda timeout >8s | Failover to read replica | Engineering |
| Lambda cold start > 1s | @initDuration spikes | Provisioned concurrency | Engineering |
| Redemption verification lag | /redeem p95 >2s | Scale API + limit retries | Engineering |

## 3. Detailed Runbooks

### 3.1 Neon Cold Start Latency

- **Detection:** CloudWatch alarm on Lambda p99 > 3000ms (5-min window)
- **Mitigation:**
  1. Enable keep-warm Lambda (`rate(4 minutes)`) on critical routes
  2. Show skeleton loaders on map + redemption surfaces
- **Recovery:** Monitor p95 < 500ms, then remove keep-warm if cost spikes

### 3.2 Mapbox API Outage

- **Detection:** HEAD request to tile endpoint returns non-200, >2% error rate
- **Mitigation:**
  1. Switch to Lite Mode (list view + distance sorting)
  2. Disable heatmap layer to reduce tile fetches
- **Recovery:** Restore map rendering when error rate < 0.5%

### 3.3 PostGIS Query Timeout

- **Detection:** Lambda timeout after 8s (10s limit)
- **Mitigation:**
  1. Route heatmap reads to replica
  2. Reduce heatmap window to 10 minutes temporarily
- **Recovery:** Run `REINDEX CONCURRENTLY` off-peak and verify query plan

### 3.4 Lambda Cold Start > 1s

- **Detection:** CloudWatch Logs Insights on `@initDuration`
- **Mitigation:**
  1. Move DB connection setup to module scope
  2. Add provisioned concurrency for `/redeem`
- **Recovery:** Validate init duration < 500ms at p95

### 3.5 Redemption Verification Lag

- **Detection:** /redeem p95 > 2s, seller timeouts rising
- **Mitigation:**
  1. Prioritize redemption route with reserved concurrency
  2. Short-circuit duplicate checks when token already expired
- **Recovery:** Confirm seller app success rate > 95%

## 4. Post-Incident Checklist

- Update OPS-03 postmortem with root cause and remediation
- Add or adjust alerts in TECH-08
- Update this document with new mitigation steps

## 5. Related Documents

**Dependencies**
- TECH-08: Section 4
- OPS-03: Section 3

**Related Specs**
- TECH-07: Section 4
- TECH-01: Section 2
- TECH-03: Section 2
- TECH-04: Section 4
- THREAD-05: Section 3

**Implementation Guides**
- IMPL-02: Section 4

## 6. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Engineering Lead | Failure mode catalog and runbooks |
