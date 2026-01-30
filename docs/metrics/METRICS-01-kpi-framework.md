---
document_id: METRICS-01
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Data Lead
dependencies:
  - STRAT-01
  - BIZ-01
related_documents:
  - METRICS-02
  - METRICS-03
  - PRD-01
  - PRD-02
---

# METRICS-01: KPI Framework

## 1. Executive Summary

This document defines the North Star metric and supporting KPIs for Frictionless. Metrics are aligned to the marketplace loop: deal supply, buyer discovery, redemption, and merchant retention.

The KPI framework is designed to connect strategy (STRAT-01) to product decisions and operational reporting. It prioritizes conversion outcomes over vanity engagement metrics.

## 2. North Star Metric

**North Star:** Monthly Successful Redemptions (MSR)

**Rationale:** Redemptions are the core value delivery moment for both buyers and merchants. Every redemption indicates real-world conversion, validates deal relevance, and drives revenue.

**Formula:**
```
MSR = count(redemption_id) where status = "success" and created_at within month
```

## 3. KPI Hierarchy

### 3.1 Marketplace Health
| KPI | Definition | Target (Year 1) |
| --- | --- | --- |
| MSR | Monthly Successful Redemptions | 10,000 |
| Active Merchants | Merchants with ≥1 redemption in 30 days | 550 |
| Active Buyers | Buyers with ≥1 session in 30 days | 50,000 |
| Deal Supply | Active deals visible per day | 1,000 |

### 3.2 Buyer Funnel
| KPI | Definition | Target |
| --- | --- | --- |
| Install → Signup | % installs completing OTP | 60% |
| Signup → Opt-in | % users granting location | 70% |
| Opt-in → First View | % users viewing first deal | 65% |
| Claim Rate | Claims / Deal Views | 18% |
| Redemption Rate | Redemptions / Claims | 55% |

### 3.3 Merchant Funnel
| KPI | Definition | Target |
| --- | --- | --- |
| Lead → Signup | % merchants completing onboarding | 50% |
| Signup → First Deal | % merchants posting a deal | 60% |
| Redemption per Merchant | Monthly avg redemptions | 20 |
| Pro Conversion | % merchants upgrading to Pro | 10% |

### 3.4 Revenue
| KPI | Definition | Target |
| --- | --- | --- |
| GMV Influenced | Total spend from redeemed deals | 3M MAD / month |
| Take Rate | % of GMV captured | 10% |
| MRR | Monthly recurring revenue | 300k MAD |

## 4. Metric Definitions (YAML)

```yaml
metrics:
  msr:
    name: Monthly Successful Redemptions
    owner: data
    source: redemptions
    grain: month
  active_buyers:
    name: Monthly Active Buyers
    definition: buyers with >=1 session in 30d
    source: sessions
  claim_rate:
    name: Claim Rate
    definition: claims / deal_views
    source: events
  redemption_rate:
    name: Redemption Rate
    definition: redemptions / claims
    source: redemptions
  merchant_retention_30d:
    name: Merchant Retention 30d
    definition: merchants with redemption in last 30d
    source: redemptions
```

## 5. Data Quality Rules

- Deduplicate events by `event_id`
- Enforce server-side timestamps for redemption success
- Require merchant_id for all deal events
- Enforce location accuracy <= 50m for heatmap inputs

## 6. Related Documents

**Dependencies**
- STRAT-01: Section 7
- BIZ-01: Section 2

**Related Specs**
- METRICS-03: Section 3
- PRD-01: Section 3
- PRD-02: Section 4

**Implementation Guides**
- METRICS-02: Section 2

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Data Lead | Initial KPI framework |
