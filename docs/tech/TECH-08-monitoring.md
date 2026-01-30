---
document_id: TECH-08
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-07
related_documents:
  - OPS-03
  - METRICS-02
  - TECH-11
  - THREAD-05
---

# TECH-08: Monitoring & Observability

## 1. Executive Summary

This document defines Frictionless observability standards across logs, metrics, tracing, and alerting. The goal is to detect issues before they impact merchants or buyers and to support fast incident response.

Monitoring is split into system health (engineering) and business health (product/ops).

## 2. Observability Stack

| Layer | Tooling | Purpose |
| --- | --- | --- |
| Logs | CloudWatch Logs | Centralized Lambda and API logs |
| Metrics | CloudWatch Metrics | Latency, errors, concurrency |
| Tracing | AWS X-Ray (optional) | Trace API calls across Lambdas |
| Frontend | Sentry | Crash/error reporting in apps |
| Database | Neon Console | Query latency, connections |

## 3. Service Level Objectives (SLOs)

| Service | SLO | Target |
| --- | --- | --- |
| API availability | 30-day availability | 99.5% |
| Heatmap endpoint | p95 latency | < 2s |
| Redemption verify | p95 latency | < 800ms |
| Location ingestion | error rate | < 1% |

## 4. Alerting Rules

- **P1**: API error rate > 5% for 10 minutes
- **P1**: Redemption verify latency p95 > 1.5s for 10 minutes
- **P2**: Heatmap latency p95 > 3s for 30 minutes
- **P2**: Location ingestion failures > 2% for 30 minutes

## 5. Log Standards

Every log entry should include:

```json
{
  "level": "info",
  "service": "api",
  "endpoint": "/locations/heatmap",
  "request_id": "req_123",
  "latency_ms": 512,
  "status": 200
}
```

## 6. Escalation Path

1. On-call engineer acknowledges
2. Identify scope and affected services
3. Trigger OPS-03 incident protocol if customer impact detected

## 7. Related Documents

**Dependencies**
- TECH-07: Section 6

**Related Specs**
- METRICS-02: Section 3
- OPS-03: Section 2
- TECH-11: Section 2
- THREAD-05: Section 3

**Implementation Guides**
- IMPL-02: Section 4

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Engineering Lead | Linked failure modes and resilience thread |
| 1.0 | 2026-01-30 | Engineering Lead | Initial monitoring standards |
