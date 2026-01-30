---
document_id: MKT-01
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Research Lead
dependencies:
  - STRAT-02
related_documents:
  - MKT-02
  - MKT-03
  - BIZ-03
---

# MKT-01: Competitive Analysis

## 1. Executive Summary

Frictionless competes in a fragmented landscape that includes coupon aggregators, e-commerce marketplaces, and location discovery platforms. None deliver a real-time, on-the-street conversion loop for merchants.

Our differentiation is a live, location-first marketplace that ties discovery to immediate in-store action, backed by a spatial data moat. This creates a defensible position against both local coupon players and global mapping platforms.

## 2. Competitive Landscape Map

### 2.1 Direct Competitors (Local)
- **Daily deal aggregators** (e.g., Hmizate): web-first, pre-planned, not location-aware
- **Local loyalty apps**: single-brand value, no multi-merchant discovery
- **Retail chain apps**: locked to one merchant, no cross-store heatmap

### 2.2 Adjacent Competitors (Global)
- **Discovery platforms**: Google Maps, Yelp, Foursquare
- **Marketplaces**: Jumia, Glovo, retail chain ecommerce
- **Coupon sites**: RetailMeNot, Groupon-type models

## 3. Competitive Matrix

| Dimension | Frictionless | Daily Deals | Discovery Apps | Ecommerce Marketplaces |
| --- | --- | --- | --- | --- |
| Real-time location targeting | Strong | Weak | Medium | Weak |
| In-store conversion | Strong | Weak | Weak | None |
| Merchant ROI attribution | Strong | Weak | Weak | Medium |
| Hardware requirement | None | None | None | None |
| Heatmap intelligence | Strong | None | None | None |
| Merchant network effects | Strong | Weak | Medium | Strong |

## 4. Positioning Statement

> Frictionless is the only Moroccan platform that converts foot traffic into sales in under 10 minutes without hardware installation or POS integrations.

## 5. Differentiators & Defensibility

1. **Live Heatmap Intelligence**: unique seller-side visibility of demand
2. **SafeColor Verification Protocol**: secure, low-friction redemption
3. **PostGIS Data Moat**: spatial-temporal data accumulation
4. **City Density Strategy**: block-by-block expansion raises entry barriers

## 6. Watchlist & Threats

| Threat | Likelihood | Impact | Response |
| --- | --- | --- | --- |
| Incumbent mapping platform adds local deals | Medium | High | Build proprietary merchant network and redemption UX |
| Coupon aggregator pivots to mobile | Medium | Medium | Win on real-time relevance and heatmap value |
| Telco launches location promos | Low | High | Partner early; focus on retailer trust |
| Copycat local app | Medium | Medium | Move fast, lock in corridors, brand the category |

## 7. Related Documents

**Dependencies**
- STRAT-02: Section 2

**Related Specs**
- MKT-02: Section 3
- MKT-03: Section 2
- BIZ-03: Section 4

**Implementation Guides**
- OPS-04: Section 2

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Research Lead | Updated SafeColor terminology |
| 1.0 | 2026-01-30 | Research Lead | Initial competitive analysis |
