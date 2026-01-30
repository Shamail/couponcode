---
document_id: MKT-02
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Research Lead
dependencies:
  - STRAT-02
related_documents:
  - PRD-01
  - PRD-02
  - GUIDE-01
  - GUIDE-02
---

# MKT-02: User Research

## 1. Executive Summary

This document defines buyer and seller personas, Jobs-to-be-Done (JTBD), and the research plan for validating Frictionless' value proposition in Moroccan urban corridors. It combines early assumptions with a structured approach to field validation.

The primary research goal is to confirm that real-time, location-driven deals can change in-store behavior within minutes and that merchants will pay for visible conversion outcomes.

## 2. Core Personas

### 2.1 Buyer Persona: Yasmine (Urban Deal Shopper)
- Age: 22-32
- Behavior: Walkable shopping corridors on weekends
- Motivation: Discovery + savings + fun
- Friction: Too many generic promotions, not enough immediate relevance

### 2.2 Seller Persona: Karim (Independent Merchant)
- Age: 35-55
- Behavior: Runs a boutique, relies on word of mouth + Instagram
- Motivation: Foot traffic visibility, immediate conversions
- Friction: Low ROI on ads, no visibility into passing demand

### 2.3 Secondary Persona: Amina (Chain Manager)
- Age: 28-40
- Behavior: Manages 3-10 locations
- Motivation: Compare store performance, standardize promotions
- Friction: No centralized data on physical foot traffic

## 3. Jobs-to-be-Done (JTBD)

| Persona | Job | Current Workaround | Opportunity |
| --- | --- | --- | --- |
| Buyer | Discover nearby deals while walking | Instagram browsing, window shopping | Real-time map + nearby deal alerts |
| Buyer | Feel rewarded for exploring | No structure | Reward points + membership tiers |
| Seller | Convert passersby quickly | Flyers, signage | Limited-time deals + push broadcast |
| Seller | Understand demand in real time | Guesswork | Heatmap + activity indicators |

## 4. Research Objectives

1. Validate buyer willingness to share location in exchange for nearby deals
2. Measure conversion impact of time-limited, proximity-based offers
3. Test merchant willingness to pay for redemption-based pricing
4. Evaluate trust in QR-based redemption with no hardware

## 5. Research Plan

### 5.1 Methods
- **Street Intercepts**: 30 buyer interviews in Maarif and Gauthier
- **Merchant Interviews**: 20 store owners across fashion, food, beauty
- **Prototype Testing**: 10 buyers with clickable map prototype
- **Pilot Redemption Tests**: 5 merchants running limited-time deals

### 5.2 Sample Questions

**Buyer**
- “When you walk through Maarif, how do you decide which stores to enter?”
- “Would you allow location tracking if deals were relevant and instant?”
- “What makes a deal feel worth claiming?”

**Seller**
- “How do you measure whether a promotion worked?”
- “What would make you pay for a conversion-based promo?”
- “What worries you about a QR-based redemption?”

## 6. Hypotheses to Validate

| Hypothesis | Success Indicator |
| --- | --- |
| Buyers will opt into location for immediate deals | 60%+ opt-in rate in onboarding |
| Limited-time deals increase store visits | 20% uplift vs baseline foot traffic |
| Merchants will pay for redemption-based pricing | 50% of pilot merchants opt into paid tier |
| QR redemption is trusted and fast | 90% successful scans in pilot |

## 7. Insights Capture Template

```yaml
session_id: R-0001
persona: buyer
location: maarif
insight_summary: "Buyers want to see deals only when within 200m"
quote: "If I'm already on the street, send me something now."
implication: reduce default radius to 300m for dense corridors
confidence: medium
```

## 8. Related Documents

**Dependencies**
- STRAT-02: Section 2

**Related Specs**
- PRD-01: Section 2
- PRD-02: Section 2
- GUIDE-01: Section 2

**Implementation Guides**
- OPS-04: Section 2

## 9. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Research Lead | Updated terminology |
| 1.0 | 2026-01-30 | Research Lead | Initial research plan |
