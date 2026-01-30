---
document_id: STRAT-02
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Strategy Lead
dependencies:
  - STRAT-01
related_documents:
  - MKT-01
  - MKT-02
  - MKT-03
  - BIZ-01
  - BIZ-03
---

# STRAT-02: Market Analysis

## 1. Executive Summary

This document outlines the market opportunity for Frictionless in Morocco using narrative simulation, buyer and seller psychology, and corridor-based sizing. It explains why a real-time, location-driven platform can unlock conversion in dense urban retail corridors.

The core finding: existing marketing channels fail because they are temporally and geographically misaligned with buying intent. Frictionless collapses the time between discovery and purchase to minutes.

| Field | Value |
| --- | --- |
| Version | 1.0 |
| Date | January 2026 |
| Classification | Strategic Foundation Document |

## 2. The "Maarif" Simulation

### 2.1 A Buyer Journey in Casablanca

**Setting:** Saturday, 3:00 PM. Boulevard Massira Al Khadra, Maarif District.

**The Buyer: Yasmine (28)**

Yasmine lives in Hay Hassani. She takes the tramway to Maarif for her weekend shopping ritual. She has no specific purchase intent — just browsing.

```
Timeline:

15:00 — Exits tram at Maarif station. Opens Frictionless app.
        → App shows 3 deal markers within 200m
        → One is a 30% limited-time deal at a boutique she's never noticed

15:05 — Walks toward the boutique. App shows "Getting closer..."
        → The store's pin pulses as she approaches

15:08 — Enters store. Browses. Finds item she likes.
        → Taps "Claim Deal" → QR code appears (green border)
        → Timer: 60 seconds

15:10 — Shows QR to cashier. Cashier scans with Seller App.
        → Both screens flash GREEN simultaneously
        → Discount applied. Transaction complete.

15:12 — Receives "Deal redeemed" animation
        → +50 Activity Points credited
        → Prompted: "More deals nearby. Keep exploring?"
```

**What just happened:**
- A store she'd never visited converted a casual browser into a buyer
- The merchant paid nothing until redemption occurred
- The journey from discovery to purchase: 12 minutes, zero friction

### 2.2 The Seller: Karim (45)

Karim owns a men's clothing boutique on Rue Sebou. He's been in business 15 years. His Instagram has 2,000 followers, but engagement dropped 80% since algorithm changes.

```
Timeline:

14:00 — Opens Seller App (Shadow). Sees his store location.
        → Heatmap shows LOW activity (blue zone)
        → Thought: "Slow Saturday so far"

14:30 — Heatmap shifts to MEDIUM (yellow zone)
        → 12 anonymous users detected within 300m
        → Thought: "People are out. They're just not coming in."

14:45 — Creates a limited-time deal: "30% off all polos. 2 hours only."
        → Pays 20 MAD to broadcast to nearby users
        → 8 users receive push notification

15:08 — First customer enters, QR in hand
        → Karim scans. GREEN match.
        → First Frictionless conversion.

15:30 — Checks stats: 3 redemptions, 2 more QRs pending
        → ROI on 20 MAD broadcast fee: 3 sales @ avg 350 MAD = 1,050 MAD
```

**What just happened:**
- Karim saw real-time demand for the first time ever
- He made a data-driven decision to trigger a limited-time deal
- His cost per acquisition: 6.67 MAD (vs. 50+ MAD for Instagram ads)

## 3. The Friction Problem

### 3.1 Why Traditional Marketing Fails in Morocco

| Channel | The Friction |
| --- | --- |
| SMS Marketing | Perceived as spam. 95% delete without reading. Carriers increasingly block bulk SMS. |
| Email Marketing | Low penetration. Most Moroccans use email only for work. Personal email rarely checked. |
| Instagram Ads | Algorithm-dependent. Reach declining. No geographic precision. No temporal relevance. |
| WhatsApp Broadcast | Requires opt-in. Quickly blocked by users. Cannot scale legally. |
| Loyalty Cards | Physical clutter. Users forget them. No discovery mechanism. |
| Window Displays | Only works if user is already looking. Invisible to someone 100m away. |

### 3.2 The Fundamental Mismatch

```
┌─────────────────────────────────────────────────────────────┐
│                    MARKETING TIMING                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   When Marketing Reaches You:    When You're Shopping:      │
│   ───────────────────────────    ───────────────────────    │
│   • At home on Instagram         • Walking down the street  │
│   • At work checking email       • Already in the zone      │
│   • Random SMS at 11 PM          • Intent already exists    │
│                                                             │
│              ZERO OVERLAP = ZERO CONVERSION                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Frictionless thesis:** Collapse the time between marketing impression and purchase opportunity to < 5 minutes.

## 4. Competitive Landscape

### 4.1 Direct Competitors (Morocco)

| Competitor | What They Do | Why They Fail |
| --- | --- | --- |
| Hmizate | Daily deals website | Desktop-first. No location. Requires pre-planning. |
| Vente-Privee.ma | Limited-time sales for brands | E-commerce only. No physical retail integration. |
| Loyalty apps | Points accumulation | Single-brand. No discovery. No heatmap. |

**Verdict:** No one is solving the "walking past the store" moment.

### 4.2 Adjacent Competitors (Global)

| Competitor | What They Do | Gap for Morocco |
| --- | --- | --- |
| Google Maps | Location + business info | No live intent data. No deals. No heatmap for merchants. |
| Yelp | Reviews + discovery | Review-driven, not deal-driven. Weak in Morocco. |
| Foursquare | Check-ins | Dead product. No merchant value prop. |
| Shopkick | In-store rewards | Requires beacon hardware. US-only. |
| RetailMeNot | Coupon aggregator | Desktop-era UX. No spatial component. |

**Our differentiator:** We are the only product that shows merchants a live heatmap of potential customers.

## 5. Market Sizing

### 5.1 Total Addressable Market (TAM)

```
Morocco Urban Retail Market
├── Annual retail spend (urban): ~200B MAD
├── Percentage influenced by promotions: ~30%
└── Promotion-influenced spend: ~60B MAD
```

### 5.2 Serviceable Addressable Market (SAM)

```
Primary Corridors (Year 1-2)
├── Casablanca: Maarif, Gauthier, Ain Diab, Anfa
├── Rabat: Agdal, Hassan, Hay Riad
├── Marrakech: Gueliz, Hivernage
├── Tangier: City Center, Malabata
└── Estimated merchants: 15,000
```

### 5.3 Serviceable Obtainable Market (SOM)

```
Year 1 Target
├── Active merchants: 500 (free) + 50 (pro)
├── Monthly redemptions: 10,000
├── Average deal value: 300 MAD
├── GMV influenced: 3M MAD/month
└── Revenue potential: 300K MAD/month (at 10% take rate)
```

## 6. User Psychology

### 6.1 The Buyer Mindset

**Saturday afternoon in Maarif:**
- "I want to discover something"
- "I want a reason to enter a store"
- "I want to feel like I got a deal"
- "I don't want to be sold to — I want to hunt"

**Frictionless response:**
- Guided discovery ("Deal nearby")
- Visual reward ("Deal found")
- Social proof (see activity density)
- Urgency (time-limited deals)

### 6.2 The Seller Mindset

**Karim's daily stress:**
- "I can see people walking past. Why don't they come in?"
- "I spend money on Instagram but can't measure foot traffic"
- "My competitor next door seems busier. What are they doing?"
- "I need customers NOW, not next week"

**Frictionless response:**
- Visibility into real-time foot traffic
- Ability to trigger instant demand
- Pay-per-result model
- Competitive intelligence (see your zone's activity)

## 7. Barriers to Entry (Our Moat)

| Barrier | Description |
| --- | --- |
| Data network effect | More users = better heatmap = more merchant value = more deals = more users |
| Geographic density | First mover in Maarif owns Maarif. Chicken-and-egg problem for followers. |
| PostGIS expertise | Spatial SQL is rare skill. Our schema is optimized for Morocco. |
| Brand association | "Frictionless" becomes verb: "Did you Frictionless that deal?" |
| Merchant relationships | Sales team builds trust. Hard to displace once established. |

## 8. Go-to-Market: The "Block by Block" Strategy

### 8.1 Phase 1: Maarif Domination (Months 1-3)

```
Focus: 10-block radius around Boulevard Massira Al Khadra

Actions:
├── Onboard 50 merchants (free tier)
├── Street team distributes flyers to shoppers
├── "Savings Weekend" launch event (Saturday, 2 PM)
├── Target: 1,000 app downloads in Week 1
└── Success metric: 100 redemptions in Month 1
```

### 8.2 Phase 2: Casablanca Expansion (Months 4-6)

```
Corridors: Gauthier, Ain Diab, Maarif extension

Actions:
├── Launch Pro tier for proven merchants
├── Referral program: "Invite a store, get 1 free broadcast"
├── Partnership with Casablanca Mall Association
└── Target: 5,000 MAU, 200 merchants
```

### 8.3 Phase 3: Multi-City (Months 7-12)

```
Cities: Rabat, Marrakech, Tangier

Actions:
├── Localized street teams in each city
├── City-specific launch events
├── Regional merchant onboarding
└── Target: 50,000 MAU, 500 merchants nationally
```

## 9. Risk Analysis

| Risk | Probability | Impact | Mitigation |
| --- | --- | --- | --- |
| Low app adoption | Medium | High | Gamification, referral rewards, street team activation |
| Merchant churn | Medium | Medium | Prove ROI in first 30 days. Offer success guarantee. |
| Battery drain complaints | High | Medium | Optimize GPS polling. Clear user education. |
| Copycat from telco | Low | High | Move fast. Own the brand. Build data moat. |
| Privacy concerns | Medium | High | Anonymization. Transparent data policy. CNDP compliance. |

## 10. Conclusion

Morocco's urban retail corridors represent a uniquely underserved market:
- High smartphone penetration
- Dense, walkable commercial zones
- Digital marketing fatigue
- No incumbent solution

Frictionless is positioned to become the default layer between physical presence and digital discovery.

> "In Maarif, opportunity walks past your door every minute. Frictionless makes sure you see it."

## 11. Related Documents

**Dependencies**
- STRAT-01: Section 2

**Related Specs**
- MKT-01: Section 2
- MKT-02: Section 3
- MKT-03: Section 2
- BIZ-01: Section 2

**Implementation Guides**
- OPS-04: Section 2

## 12. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Strategy Lead | Updated terminology and examples |
| 1.0 | 2026-01-30 | Strategy Lead | Standardized metadata and sectioning |
