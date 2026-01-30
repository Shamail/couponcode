---
document_id: BIZ-01
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Business Lead
dependencies:
  - STRAT-01
  - STRAT-02
related_documents:
  - BIZ-02
  - BIZ-03
  - METRICS-01
---

# BIZ-01: Monetization Model

## Executive Summary

This document describes the Frictionless revenue model, including subscription tiers, transaction fees, and future data licensing. It is designed to align merchant value with platform revenue through pay-per-result mechanics.

Pricing should be reviewed quarterly as the marketplace matures.

## Revenue Streams for Frictionless

**Version:** 1.0
**Date:** January 2026
**Classification:** Business Model Document

---

## Overview

Frictionless employs a **freemium B2B model** with three revenue streams:

1. **Subscription Tiers** — Monthly merchant plans
2. **Transaction Fees** — Limited-Time Deal broadcasts
3. **Data Licensing** — Aggregate insights (Year 2+)

---

## Revenue Stream 1: Subscription Tiers

### Free Tier (الأساسي)

**Price:** 0 MAD/month

| Feature | Included |
|---------|----------|
| Store listing on map | ✅ |
| Basic activity indicator (Low/Med/High) | ✅ |
| QR redemption validation | ✅ |
| Deal creation (1 active at a time) | ✅ |
| Monthly redemption report | ✅ |
| Heatmap visualization | ❌ |
| Push broadcast capability | ❌ |
| Historical analytics | ❌ |

**Purpose:** Land merchants. Prove value. Create upgrade path.

---

### Pro Tier (المحترف)

**Price:** 299 MAD/month (~$30 USD)

| Feature | Included |
|---------|----------|
| Everything in Free | ✅ |
| **Live Heatmap visualization** | ✅ |
| Multiple active deals (up to 5) | ✅ |
| Weekly analytics dashboard | ✅ |
| Competitor activity alerts | ✅ |
| 2 free Limited-Time broadcasts/month | ✅ |
| Priority customer support | ✅ |

**Target merchant:** Established boutiques doing 50K+ MAD/month

**Value proposition:** "See your customers before they see you"

---

### Enterprise Tier (المؤسسات)

**Price:** 999 MAD/month (~$100 USD)

| Feature | Included |
|---------|----------|
| Everything in Pro | ✅ |
| Multi-location management | ✅ |
| API access for POS integration | ✅ |
| Custom analytics reports | ✅ |
| Dedicated account manager | ✅ |
| 10 free Limited-Time broadcasts/month | ✅ |
| White-label QR validation | ✅ |

**Target merchant:** Chains, franchises, mall operators

---

### Subscription Revenue Projection (Year 1)

| Quarter | Free | Pro | Enterprise | MRR |
|---------|------|-----|------------|-----|
| Q1 | 100 | 10 | 0 | 2,990 MAD |
| Q2 | 250 | 30 | 2 | 10,968 MAD |
| Q3 | 400 | 50 | 5 | 19,945 MAD |
| Q4 | 500 | 80 | 10 | 33,920 MAD |

**Year 1 Subscription Revenue:** ~200,000 MAD

---

## Revenue Stream 2: Limited-Time Deal Fee

### The Limited-Time Broadcast

A pay-per-use feature allowing merchants to push instant notifications to nearby users.

**Mechanic:**
1. Merchant creates a time-limited deal (30 min - 2 hours)
2. Merchant selects broadcast radius (100m / 300m / 500m)
3. System counts users currently visible on heatmap
4. Merchant pays fee, notification sent immediately

---

### Pricing Structure

| Radius | Base Fee | Per-User Fee | Example |
|--------|----------|--------------|---------|
| 100m | 10 MAD | +0.50 MAD/user | 20 users = 20 MAD |
| 300m | 20 MAD | +0.30 MAD/user | 50 users = 35 MAD |
| 500m | 30 MAD | +0.20 MAD/user | 100 users = 50 MAD |

**Maximum charge:** 100 MAD per limited-time broadcast (protects merchants from overspending)

---

### Limited-Time Economics (Merchant Perspective)

```
Scenario: Karim's Boutique, Saturday 3 PM

Heatmap shows: 50 users within 300m
Limited-Time deal: "30% off all items, next 2 hours"
Broadcast cost: 20 + (50 × 0.30) = 35 MAD

Results:
├── 50 users notified
├── 15 users view deal (30% open rate)
├── 5 users redeem (33% conversion)
├── Average basket: 400 MAD
├── Revenue generated: 2,000 MAD
└── ROI: 5,614% (35 MAD → 2,000 MAD)
```

---

### Limited-Time Revenue Projection (Year 1)

| Quarter | Active Pro+ Merchants | Avg Limited-Time Broadcasts/Merchant/Month | Avg Broadcast Value | Quarterly Revenue |
|---------|----------------------|---------------------------|-----------------|-------------------|
| Q1 | 10 | 4 | 30 MAD | 3,600 MAD |
| Q2 | 32 | 6 | 35 MAD | 20,160 MAD |
| Q3 | 55 | 8 | 40 MAD | 52,800 MAD |
| Q4 | 90 | 10 | 45 MAD | 121,500 MAD |

**Year 1 Limited-Time Revenue:** ~200,000 MAD

---

## Revenue Stream 3: Data Licensing (Year 2+)

### The Strategic Asset

By Year 2, Frictionless will possess:
- 100M+ GPS data points
- Foot traffic patterns for major corridors
- Cross-shopping behavior graphs
- Dwell time analytics by zone

### Potential Customers

| Customer Type | Use Case | Pricing Model |
|---------------|----------|---------------|
| **Mall Operators** | Optimize tenant mix, measure traffic flow | 5,000 MAD/month subscription |
| **CPG Brands** | Understand where their customers shop | Per-report pricing (2,000-10,000 MAD) |
| **Real Estate Developers** | Location intelligence for new projects | Project-based (10,000-50,000 MAD) |
| **Urban Planners** | Pedestrian flow analysis | Government contracts |
| **Ad Agencies** | OOH placement optimization | Per-campaign insights |

### Data Products

1. **Catchment Reports**
   - "What is the true catchment area of Location X?"
   - Includes: visitor origins, dwell times, cross-shopping

2. **Corridor Analytics**
   - "How does foot traffic in Maarif compare to Gauthier?"
   - Includes: hourly patterns, day-of-week trends, seasonal variations

3. **Brand Affinity Maps**
   - "Where do Nike shoppers also shop?"
   - Includes: anonymized cross-visit patterns

### Data Revenue Projection (Year 2)

| Product | Customers | Annual Value | Revenue |
|---------|-----------|--------------|---------|
| Catchment Reports | 20 | 24,000 MAD | 480,000 MAD |
| Corridor Subscriptions | 5 | 60,000 MAD | 300,000 MAD |
| Custom Projects | 10 | 25,000 MAD | 250,000 MAD |

**Year 2 Data Revenue Target:** 1,000,000 MAD

---

## Combined Revenue Model

### Year 1 Projection

```
Revenue Breakdown:
├── Subscriptions: 200,000 MAD (50%)
├── Limited-Time Fees: 200,000 MAD (50%)
├── Data Licensing: 0 MAD (0%)
└── Total Year 1: 400,000 MAD (~$40,000 USD)
```

### Year 2 Projection

```
Revenue Breakdown:
├── Subscriptions: 800,000 MAD (35%)
├── Limited-Time Fees: 700,000 MAD (30%)
├── Data Licensing: 800,000 MAD (35%)
└── Total Year 2: 2,300,000 MAD (~$230,000 USD)
```

### Year 3 Projection

```
Revenue Breakdown:
├── Subscriptions: 2,000,000 MAD (30%)
├── Limited-Time Fees: 1,500,000 MAD (23%)
├── Data Licensing: 3,000,000 MAD (47%)
└── Total Year 3: 6,500,000 MAD (~$650,000 USD)
```

---

## Unit Economics

### Customer Acquisition Cost (CAC)

| Channel | Cost | Merchants Acquired | CAC |
|---------|------|-------------------|-----|
| Street team | 10,000 MAD/month | 30 merchants | 333 MAD |
| Referral program | 50 MAD per referral | 20 merchants | 50 MAD |
| Organic (app discovery) | 0 | 10 merchants | 0 |

**Blended CAC:** ~200 MAD per merchant

### Lifetime Value (LTV)

```
Pro Merchant LTV Calculation:
├── Monthly subscription: 299 MAD
├── Monthly broadcast spend: 150 MAD
├── Average lifespan: 18 months
├── Gross margin: 80%
└── LTV: (299 + 150) × 18 × 0.80 = 6,465 MAD
```

**LTV:CAC Ratio:** 32:1 (extremely healthy)

---

## Pricing Philosophy

### Principles

1. **No cost until value delivered**
   - Free tier proves concept
   - Broadcast fee only charged on send
   - Subscriptions require demonstrated ROI

2. **Moroccan market pricing**
   - Pro at 299 MAD (~10 MAD/day)
   - Affordable for small merchants
   - Significant margin vs. Instagram ad spend

3. **Land and expand**
   - Single-location free → Pro upgrade
   - Multi-location → Enterprise
   - Enterprise → Data licensing customer

---

## Risk Mitigation

### Price Sensitivity

**Risk:** Merchants resist paying for digital tools

**Mitigation:**
- 30-day free Pro trial
- "Satisfaction guarantee" — refund if no redemptions
- Show clear ROI metrics in dashboard

### Competition Undercutting

**Risk:** Telco or tech giant offers similar product for free

**Mitigation:**
- Build data moat quickly
- Focus on merchant relationships
- Price is not our only differentiator (heatmap quality is)

### Low Limited-Time Adoption

**Risk:** Merchants don't use pay-per-broadcast

**Mitigation:**
- Include free limited-time broadcast credits in Pro tier
- Train merchants on optimal broadcast timing
- Show case studies of successful limited-time campaigns

---

## Financial Summary

| Metric | Year 1 | Year 2 | Year 3 |
|--------|--------|--------|--------|
| Revenue | 400K MAD | 2.3M MAD | 6.5M MAD |
| Active Merchants (Paid) | 90 | 400 | 1,200 |
| Gross Margin | 80% | 82% | 85% |
| CAC | 200 MAD | 180 MAD | 150 MAD |
| LTV | 6,465 MAD | 7,500 MAD | 9,000 MAD |
| LTV:CAC | 32:1 | 42:1 | 60:1 |

---

## Conclusion

Frictionless monetizes through three complementary streams:

1. **Subscriptions** — Recurring, predictable revenue from Pro/Enterprise merchants
2. **Limited-Time Fees** — Transactional revenue tied to real-time demand
3. **Data Licensing** — High-margin strategic asset (Year 2+)

The model is designed for:
- Low barrier to entry (free tier)
- Clear value demonstration (heatmap unlocks at Pro)
- Scalable unit economics (LTV:CAC > 30:1)

**The heatmap is the hook. The data is the gold mine.**

---

*"Free to see the signal. Pay to broadcast your own."*

— Frictionless Team

## Related Documents

**Dependencies**
- STRAT-01: Section 5
- STRAT-02: Section 5

**Related Specs**
- BIZ-02: Section 2
- METRICS-01: Section 3

**Implementation Guides**
- OPS-01: Section 6

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Business Lead | Updated limited-time broadcast terminology |
| 1.0 | 2026-01-30 | Business Lead | Standardized metadata and cross-references |
