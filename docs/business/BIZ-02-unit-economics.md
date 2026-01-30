---
document_id: BIZ-02
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Finance Lead
dependencies:
  - BIZ-01
related_documents:
  - METRICS-01
  - BIZ-03
  - BIZ-04
---

# BIZ-02: Unit Economics

## 1. Executive Summary

This document outlines the unit economics model for Frictionless at the merchant and marketplace level. It defines assumptions, contribution margin, and LTV:CAC targets.

The model should be updated quarterly as real data replaces assumptions.

## 2. Core Assumptions (Sample)

```yaml
assumptions:
  avg_deal_value_mad: 300
  take_rate: 0.10
  redemptions_per_merchant_month: 20
  merchant_subscription_mad: 499
  monthly_churn: 0.05
  gross_margin: 0.80
  merchant_cac_mad: 1200
  fraud_loss_rate: 0.03
```

## 3. Revenue Per Merchant

| Component | Formula | Example |
| --- | --- | --- |
| Transaction Revenue | redemptions * deal_value * take_rate | 20 * 300 * 10% = 600 MAD |
| Subscription Revenue | monthly fee | 499 MAD |
| Total Revenue | transaction + subscription | 1,099 MAD |

## 4. Contribution Margin

```
Contribution Margin = Total Revenue * Gross Margin
```

Example:
- 1,099 MAD * 80% = 879 MAD

## 5. LTV and CAC

```
LTV = (Contribution Margin) / (Monthly Churn)
LTV:CAC target = 3:1
```

Example:
- LTV = 879 / 0.05 = 17,580 MAD
- LTV:CAC = 17,580 / 1,200 = 14.6x (target exceeded)

## 6. Sensitivity Analysis

| Variable | Downside | Base | Upside |
| --- | --- | --- | --- |
| Redemptions / month | 8 | 20 | 35 |
| Take rate | 8% | 10% | 12% |
| Churn | 8% | 5% | 3% |
| Fraud loss rate | 5% | 3% | 2% |

Seasonality multipliers should be applied at planning time:
- Ramadan: 1.8x demand
- Summer: 1.4x demand

## 7. Related Documents

**Dependencies**
- BIZ-01: Section 2

**Related Specs**
- METRICS-01: Section 4
- BIZ-03: Section 3
- BIZ-04: Section 2

**Implementation Guides**
- OPS-01: Section 5

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Finance Lead | Added fraud loss and seasonality sensitivity |
| 1.0 | 2026-01-30 | Finance Lead | Initial unit economics model |
