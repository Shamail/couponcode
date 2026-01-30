---
document_id: STRAT-01
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Strategy Lead
dependencies: []
related_documents:
  - STRAT-02
  - BIZ-01
  - PRD-01
  - PRD-02
  - TECH-00
  - BRAND-01
---

# STRAT-01: Executive Summary

## 1. Executive Summary

Frictionless is the real-time navigation layer between physical retail and digital intent. It connects shoppers with nearby, time-sensitive offers while giving merchants live visibility into demand.

The core insight is simple: marketing only works when it reaches people at the exact moment they are ready to buy. Frictionless collapses the distance between discovery and redemption to minutes.

| Field | Value |
| --- | --- |
| Version | 1.0 |
| Date | January 2026 |
| Classification | Strategic Foundation Document |

## 2. The Vision: "Waze for Shopping"

Frictionless is the real-time navigation layer between physical retail and digital intent.

- **For Buyers:** Navigate deals as they walk. The app surfaces relevant offers based on *where you are*, not who you are.
- **For Sellers:** See traffic jams of customers. A live heatmap reveals foot traffic density and shopping intent within your catchment area.

We don't compete with e-commerce. We complete it. Frictionless captures the retail that still happens in physical stores — but adds the data intelligence of digital.

## 3. The Problem: High Traffic, Low Conversion

Morocco's urban retail corridors (Maarif, Gauthier, Agdal, Gueliz) see thousands of daily pedestrians. Yet:

| Metric | Reality |
| --- | --- |
| SMS Marketing Open Rate | < 5% |
| Email Marketing Conversion | < 1% |
| Instagram Ad Relevance | Geographically broad, temporally delayed |
| Walk-in Conversion | 2-3% of foot traffic |

**The friction:** Marketing reaches users when they're *not* shopping. By the time intent exists, the marketing is forgotten.

**Our thesis:** The only marketing that matters is the marketing that reaches you *while you're physically walking past the store*.

## 4. The Solution: Mobile-Only, Zero Hardware

### 4.1 What We Are

| Component | Description |
| --- | --- |
| **Buyer App** | React Native (Expo). Foreground GPS tracking. Surfaces nearby deals in a map-first UI. |
| **Seller App (Shadow)** | React Native (Expo). Polls API every 30 seconds. Displays anonymized heatmap of nearby users. |
| **Redemption** | QR-based SafeColor Verification — dynamic QR + color code. No POS integration required. |

### 4.2 What We Are Not

- No Bluetooth beacons
- No NFC taps
- No Wi-Fi sniffing
- No desktop middleware
- No hardware installation

**Philosophy:** If it requires a technician visit, it doesn't scale in Morocco.

## 5. The Data Play: Postgres for Everything

### 5.1 The Strategic Asset

By standardizing on **Neon DB (Serverless PostgreSQL)** with **PostGIS**, we build something no competitor in Morocco possesses:

> A historical spatial database of Moroccan shopping behaviors.

Every location check-in, every deal view, every redemption creates a spatial-temporal record:

```sql
-- Sample data point
{
  "user_id": "anon_hash_xyz",
  "location": "POINT(-7.6192 33.5731)",  -- Maarif, Casablanca
  "timestamp": "2026-01-30T14:32:00Z",
  "event": "deal_viewed",
  "merchant_id": "merchant_abc",
  "category": "fashion"
}
```

### 5.2 What This Enables

| Capability | Value |
| --- | --- |
| Foot Traffic Analytics | "How many people walked past Store X between 2-4pm on Saturdays?" |
| Dwell Time Patterns | "Users spend 12 minutes in this zone before purchasing." |
| Cross-Shopping Behavior | "Users who visit Store A also visit Store B within 30 minutes." |
| Catchment Mapping | "Your real catchment area is 400m, not 200m." |
| Brand Intelligence | Aggregate, anonymized insights sold to CPG brands and mall operators |

### 5.3 The Long Game

- **Year 1:** Acquire users and merchants. Build the dataset.
- **Year 2:** Launch analytics dashboard for Pro merchants.
- **Year 3:** License aggregate insights to brands, malls, and urban planners.

The heatmap is the feature. **The database is the moat.**

## 6. Technical Architecture (Overview)

```
┌─────────────────────────────────────────────────────────────────┐
│                         INFRASTRUCTURE                          │
│                      AWS via SST v3 (IaC)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│   │  Buyer App  │     │ Seller App  │     │   API       │      │
│   │  (Expo RN)  │────▶│  (Expo RN)  │────▶│ (Lambda)    │      │
│   │             │     │  Heatmap    │     │             │      │
│   └─────────────┘     └─────────────┘     └──────┬──────┘      │
│                                                   │             │
│                                           ┌──────▼──────┐      │
│                                           │   Neon DB   │      │
│                                           │  (PostGIS)  │      │
│                                           └─────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Key Decisions:**

- **SST v3:** Infrastructure as code. Reproducible. Cost-efficient serverless.
- **Neon DB:** Serverless Postgres. Scales to zero. Native PostGIS support.
- **Expo:** Over-the-air updates. Single codebase for iOS/Android.
- **Mapbox GL:** Best-in-class mapping. Works offline in Morocco.
- **Polling (not WebSockets):** Simpler. More reliable on Moroccan mobile networks.

## 7. Why Morocco, Why Now

| Factor | Opportunity |
| --- | --- |
| Smartphone Penetration | 85%+ in urban areas |
| Mobile Data Costs | Dropping rapidly (4G/5G expansion) |
| Retail Density | High concentration in walkable urban corridors |
| Digital Marketing Fatigue | Users actively ignore push notifications from brands |
| No Incumbent | No "Waze for Shopping" exists locally |
| Cash + Digital | QR redemption works with both payment types |

## 8. Success Metrics (North Stars)

| Metric | Target (Year 1) |
| --- | --- |
| Monthly Active Buyers | 50,000 |
| Active Merchants (Free) | 500 |
| Active Merchants (Pro) | 50 |
| Redemptions/Month | 10,000 |
| Location Updates/Day | 1,000,000 |
| Data Points Collected | 100M+ |

## 9. The Ask

This document establishes the strategic foundation. Subsequent documents will detail:

- **STRAT-02:** Market Analysis
- **BIZ-01:** Monetization Model
- **TECH-01:** Technical Architecture
- **PRD-01/02:** Product Requirements

> "We don't sell ads. We sell presence."

## 10. Related Documents

**Dependencies**
- None

**Related Specs**
- STRAT-02: Section 2
- BIZ-01: Section 2
- PRD-01: Section 2
- PRD-02: Section 2
- TECH-00: Section 2

**Implementation Guides**
- IMPL-01: Section 2

## 11. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Strategy Lead | Updated terminology and redemption naming |
| 1.0 | 2026-01-30 | Strategy Lead | Standardized metadata and sectioning |
