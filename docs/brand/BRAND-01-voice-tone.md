---
document_id: BRAND-01
version: 2.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - STRAT-01
  - BRAND-00
related_documents:
  - BRAND-02
  - BRAND-04
  - BRAND-09
  - DES-01
  - GUIDE-01
  - GUIDE-02
---

# BRAND-01: Voice & Tone Guide

## Executive Summary

This guide defines the Frictionless brand voice: clear, helpful, confident, and warm. The experience should feel like a modern utility app with a lifestyle edge—trustworthy, calm, and easy to act on.

The goal is simple: help people discover nearby deals without noise or gimmicks.

## The Frictionless Identity

**Tagline:** *Discover. Save. Enjoy.*

---

## 1. Brand Essence

### The Vibe: Modern Discovery

Frictionless is a smart shopping companion for everyday life in Morocco:

- **Clean:** Minimal, readable, and straightforward.
- **Confident:** We know the value and present it clearly.
- **Warm:** Friendly, local, and human.
- **Practical:** It gets you to savings fast, without clutter.

We are not a coupon warehouse. We are a **nearby discovery tool** that turns shopping into simple, timely wins.

---

## 2. Voice Characteristics

### Tone Pillars

| Attribute | Description | Example |
|-----------|-------------|---------|
| **Clear** | Short, direct, easy to scan | "3 deals nearby" |
| **Helpful** | Guidance without pressure | "Tap to see details" |
| **Confident** | Calm, reliable, not salesy | "Saved 45 DH today" |
| **Warm** | Friendly and local | "Enjoy your coffee" |

### Audience Modulation

| Audience | Tone Shift | Example |
| --- | --- | --- |
| Shoppers | Friendly, quick | "Nearby deal ready" |
| Merchants | Professional, respectful | "Deal published" |
| Support | Calm, empathetic | "We’re here to help" |
| System alerts | Neutral, direct | "Location required" |

### Voice Do's and Don'ts

**DO:**
- Use everyday language
- Lead with value and clarity
- Keep actions simple and obvious
- Use friendly, supportive phrases

**DON'T:**
- Use gaming jargon or insider slang
- Overhype with exaggerated urgency
- Sound cold or overly corporate
- Overload screens with copy

---

## 3. Copy Framework

### System Messages

| Context | Copy | Tone |
|---------|------|------|
| App launch | "Finding deals nearby..." | Calm |
| Location ready | "You're all set." | Reassuring |
| Deals nearby | "3 deals nearby" | Clear |
| No deals | "No deals here yet" | Honest |
| Deal claimed | "Deal claimed!" | Positive |

### Notification Language

```
Café Atlas · 25% off · 120m away
```

```
Limited-time deal: 30% off at Boutique Nora
Valid for 20 minutes
```

```
Saved 45 DH today. Nice find!
```

### Error States

| Error | Message |
|-------|---------|
| GPS disabled | "Turn on location to see nearby deals." |
| Network error | "Can't reach the server. Try again." |
| Deal expired | "This deal just ended." |
| Already claimed | "This deal is already claimed." |

---

## 4. Map Visual Language

### Base Map Configuration

**Base Style:** `mapbox://styles/mapbox/dark-v11`

**Overlay Palette:**

| Element | Color | Hex | Purpose |
|---------|-------|-----|---------|
| User location | Blue | `#3B82F6` | Your position |
| Deal marker | Amber | `#F59E0B` | Nearby deal |
| Limited-time deal | Red | `#EF4444` | Urgency |
| Expired/claimed | Gray | `#2D2D2D` | Inactive |

### Map Layers

```
Map Layers (bottom to top):
1. Dark basemap (muted streets)
2. Subtle activity density (optional)
3. Deal markers (static + subtle pulse)
4. User location (soft pulse)
5. Range ring (thin, low-contrast)
```

**Animation Specs:**
- Deal markers: subtle pulse, 3s cycle
- User dot: gentle scale (0.95 - 1.05), 3s cycle
- New deal: short fade-in + soft scale

### The Feel

The map should feel like:
- A premium ride-share app
- A modern banking dashboard
- Clean, quiet, and reliable

**NOT like:**
- A game UI
- A neon arcade display
- A noisy promo page

---

## 5. Iconography

### Core Icons

| Icon | Usage | Style |
|------|-------|-------|
| `📍` | Location | Clean and minimal |
| `🏷️` | Deal | Simple tag icon |
| `🔔` | Alerts | Subtle bell |
| `🔍` | Search | Clear magnifier |
| `⭐` | Ratings | Simple star |

### Style Notes

Icons should be:
- Simple and consistent
- Low contrast on dark background
- Used sparingly (clarity > decoration)

---

## 6. Sound Design Direction

*(For future implementation)*

| Action | Sound Character |
|--------|-----------------|
| App open | Soft chime, short |
| Deal found | Gentle tick, non-intrusive |
| Deal claimed | Warm confirmation tone |
| QR validated | Subtle success chime |
| Error | Soft buzz, brief |

---

## 7. Writing Examples

### Onboarding Screens

**Screen 1:**
> **Discover deals near you**
>
> See nearby offers in seconds. No noise, just what matters.

**Screen 2:**
> **Stay in the loop**
>
> Keep location on to get accurate distances and timing.

**Screen 3:**
> **Claim and save**
>
> Tap once to claim, then show your code in store.

### Empty States

**No deals nearby:**
> "No deals here yet. Check back soon."

**First-time user:**
> "Welcome! Deals appear when you're near stores." 

---

## 8. Localization Notes (Darija/French)

Keep language warm, short, and helpful. Avoid slang or hype.

| English | Darija Adaptation | French Adaptation |
|---------|-------------------|-------------------|
| "Deals nearby" | "عندك تخفيضات قريبة" | "Offres à proximité" |
| "Deal claimed" | "تثبّت العرض" | "Offre réservée" |
| "No deals here yet" | "ما كايناش عروض دابا" | "Pas d'offres pour l'instant" |

Additional guidance:
- Use **Arabic-Indic numerals** when the interface is fully Arabic.
- Keep CTAs under 12 characters where possible.
- Verify RTL alignment on Home, Wallet, and Profile.

---

## 9. Content Patterns Library

| Pattern | Example | Notes |
| --- | --- | --- |
| Value headline | "25% off nearby" | Lead with savings |
| Distance detail | "120m away" | Always include distance |
| Limited-time | "Ends in 8 minutes" | Keep urgency factual |
| CTA (primary) | "Claim Deal" | Verb + value |
| CTA (secondary) | "View Details" | Clear and simple |
| Empty state | "No deals here yet" | Add a next step |
| Error state | "Can't reach the server" | Offer retry |

---

## 10. Brand Voice Checklist

Before publishing any copy, ask:

- [ ] Is it clear in under 8 words?
- [ ] Is it helpful, not pushy?
- [ ] Does it sound modern and trustworthy?
- [ ] Does it guide the next action?
- [ ] Would a broad audience feel comfortable with it?

---

*"Good deals, nearby, without the noise."*

## Related Documents

**Dependencies**
- STRAT-01: Section 2
- BRAND-00: Section 5

**Related Specs**
- BRAND-02: Section 2
- BRAND-04: Section 2
- BRAND-09: Section 3
- DES-01: Section 1

**Implementation Guides**
- GUIDE-01: Section 2
- GUIDE-02: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 2.1 | 2026-01-30 | Brand Lead | Added audience modulation and content patterns |
| 2.0 | 2026-01-30 | Brand Lead | Pivoted to modern utility voice |
| 1.0 | 2026-01-30 | Brand Lead | Standardized metadata and cross-references |
