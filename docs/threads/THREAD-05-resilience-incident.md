---
document_id: THREAD-05
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Documentation Lead
dependencies:
  - TECH-08
  - OPS-03
  - TECH-11
related_documents:
  - TECH-07
  - IMPL-02
  - OPS-02
---

# THREAD-05: Resilience -> Incident -> Recovery

## 1. Executive Summary

This thread connects monitoring indicators to incident response actions and recovery playbooks. It ensures teams know how failures are detected, escalated, mitigated, and closed.

## 2. Thread Map (End-to-End)

```
Monitoring Indicator
  -> Alert Trigger
    -> Incident Triage
      -> Mitigation Playbook
        -> Customer Comms
          -> Postmortem + Follow-ups
```

## 3. Stage-by-Stage Trace

| Stage | Primary Source | Key Details |
| --- | --- | --- |
| Monitoring | TECH-08 | Logs, metrics, SLOs |
| Alerting | TECH-08 | PagerDuty + CloudWatch |
| Triage | OPS-03 | Severity tiers, owner assignment |
| Mitigation | TECH-11 | Failure-mode runbooks |
| Comms | OPS-03 | Status updates + timelines |
| Recovery | TECH-07 | Deploy + rollback guidance |

## 4. Key SLIs and Triggers

| Indicator | Threshold | Action |
| --- | --- | --- |
| Lambda p99 > 3s | 5 mins | Investigate cold start | 
| Mapbox tile failures | >2% | Switch to Lite Mode |
| Heatmap timeout rate | >1% | Route to replica |

## 5. Related Documents

**Dependencies**
- TECH-08: Section 3
- OPS-03: Section 2
- TECH-11: Section 2

**Related Specs**
- TECH-07: Section 4
- OPS-02: Section 5

**Implementation Guides**
- IMPL-02: Section 4

## 6. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Documentation Lead | Updated monitoring terminology |
| 1.0 | 2026-01-30 | Documentation Lead | Resilience and incident response thread |
