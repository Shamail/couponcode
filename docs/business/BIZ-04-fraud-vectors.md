---
document_id: BIZ-04
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Business Lead
dependencies:
  - BIZ-02
  - TECH-04
related_documents:
  - TECH-06
  - METRICS-03
  - OPS-02
---

# BIZ-04: Fraud Vectors & Mitigation Strategy

## 1. Executive Summary

This document identifies the primary fraud vectors in Frictionless and defines mitigation tactics, thresholds, and economic impact assumptions. It is the business-facing companion to TECH-04 security controls.

## 2. GPS Spoofing Detection

| Method | Threshold | Action |
| --- | --- | --- |
| Altitude check | >50m from terrain baseline | Flag + review |
| Speed variance | >120 km/h | Block redemption |
| Location jumping | >5km in <5 minutes | Suspend user |
| Mock location API | Any detection | Warn + log |

## 3. Gamification Abuse Detection

| Abuse Pattern | Detection | Mitigation |
| --- | --- | --- |
| Streak manipulation | Timezone changes >2/day | Lock streak for 24h |
| Point farming | >10 check-ins with <10m movement | Cap daily points |
| Bot timing | Identical inter-check-in intervals | CAPTCHA challenge |
| Multi-account | Same device fingerprint | Link + merge accounts |

## 4. Merchant Collusion Detection

- Device fingerprint clustering on redemptions
- Graph analysis of merchant-user redemption networks
- Z-score on daily redemption spikes (store-level anomalies)

## 5. Economic Sensitivity Enhancements

Add the following parameters to BIZ-02 sensitivity analysis:

- `fraud_loss_rate`: 2-5% of GMV
- Seasonal multipliers: Ramadan 1.8x, Summer 1.4x
- Cohort economics: Free vs Pro vs Enterprise LTV

## 6. Reporting and Escalation

| Indicator | Owner | Action |
| --- | --- | --- |
| Fraud alerts >1% of redemptions | Ops | Escalate to Security |
| Merchant anomaly score >3 sigma | Sales Ops | Manual review |
| GPS spoofing events >50/day | Product | Prioritize hardening |

## 7. Related Documents

**Dependencies**
- BIZ-02: Section 6
- TECH-04: Section 4

**Related Specs**
- TECH-06: Section 3
- METRICS-03: Section 4

**Implementation Guides**
- OPS-02: Section 5

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Business Lead | Updated terminology for check-ins |
| 1.0 | 2026-01-30 | Business Lead | Fraud vectors and mitigations |
