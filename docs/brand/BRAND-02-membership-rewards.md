---
document_id: BRAND-02
version: 2.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-01
  - BRAND-06
related_documents:
  - PRD-01
  - GUIDE-01
  - DES-03
  - DES-07
  - DES-14
---

# BRAND-02: Membership & Rewards

## Executive Summary

This document defines Frictionless membership tiers and rewards. The system is designed to feel fair, transparent, and useful—focused on savings and convenience rather than competition.

The intent is steady, repeat usage that feels like a smart habit, not a game.

---

## 1. Program Principles

1. **Clarity first** — users should always know why they earned a benefit.
2. **Value over hype** — rewards must map to real savings or time advantage.
3. **No pressure** — no shaming, no fear loops, no artificial urgency.
4. **Privacy-respecting** — rewards never require extra data sharing.

---

## 2. Points & Earning

Points reflect activity and unlock membership tiers. They are a simple accounting tool, not a score.

### Earning Actions

| Action | Points | Frequency Cap |
|--------|--------|---------------|
| Daily app open | 10 | 1x/day |
| Additional app opens | 2 | 5x/day |
| Enter a deal area | 5 | Unlimited |
| Claim a deal | 25 | Unlimited |
| Redeem a deal | 50 | Unlimited |
| Refer a friend | 100 | 3x/month |

### Principles

- Points are visible but not emphasized in core discovery flows.
- Points never expire while the account is active.
- Points cannot be purchased.

---

## 3. Membership Tiers

| Tier | Points | Perks |
|------|--------|-------|
| **Bronze** | 0 | Basic access |
| **Silver** | 500 | See stock levels |
| **Gold** | 2,000 | 5-minute early access |
| **Platinum** | 10,000 | 10-minute early access + exclusive deals |

### Tier Behavior

- Tier is shown on Profile and Wallet.
- Tier changes are celebratory but subtle.
- Early access applies only to limited-time deals.

---

## 4. Visual Tier Badges

### Badge Style

- Shape: rounded pill or compact shield
- Size: 24px height
- Icon: single simple mark (see BRAND-06)
- Text: `labelSmall` (12pt, 500)

### Tier Colors

| Tier | Color |
| --- | --- |
| Bronze | `#CD7F32` |
| Silver | `#A1A1AA` |
| Gold | `#F59E0B` |
| Platinum | `#94A3B8` |

### Placement

- Profile header (primary)
- Wallet deal cards (secondary)
- Avoid placement on map markers

---

## 5. Member Benefits in Product

### Stock Visibility (Silver+)

- Shows low/medium/high availability on deal cards.
- Helps users decide if a deal is worth walking to.

### Early Access (Gold+)

- Limited-time deals appear earlier for Gold and Platinum.
- The timer communicates the window clearly: "Early access ends in 4 minutes".

### Exclusive Deals (Platinum)

- Visible only in the Explore and Wallet tabs.
- Marked with a discrete "Platinum" tag, not a flashy badge.

---

## 6. Consistency Bonuses (Optional)

To encourage regular use without pressure, a light-touch consistency bonus can be tested:

- **3-day activity streak:** +10 bonus points
- **7-day activity streak:** +25 bonus points

Streaks are not required for core benefits and are never shown as a warning or countdown.

---

## 7. Anti-Patterns (What We Avoid)

- No public leaderboards
- No point loss or penalties
- No pay-to-win tiers
- No manipulative notifications

---

## 8. Example Member Journey

**Morning:** Opens the app on the commute and sees "2 deals nearby".

**Lunch:** Claims a café deal, redeems, and saves 18 DH.

**Evening:** Receives a limited-time deal notification and sees early access as a Gold member.

The experience feels useful and rewarding, not competitive.

---

## Related Documents

**Dependencies**
- BRAND-01: Section 2
- BRAND-06: Section 3

**Related Specs**
- PRD-01: Section 4
- DES-03: Section 2
- DES-07: Section 4
- DES-14: Section 2

**Implementation Guides**
- GUIDE-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 2.1 | 2026-01-30 | Brand Lead | Added tier badge requirements and placement |
| 2.0 | 2026-01-30 | Brand Lead | Shifted to membership & rewards model |
| 1.0 | 2026-01-30 | Brand Lead | Standardized metadata and cross-references |
