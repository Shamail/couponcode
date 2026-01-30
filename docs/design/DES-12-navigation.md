---
document_id: DES-12
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
  - DES-06
related_documents:
  - DES-01
  - DES-16
  - DES-17
---

# DES-12: Navigation Components

## Executive Summary

Navigation components ensure predictable movement across map, explore, wallet, and profile surfaces. They must be consistent across Buyer and Seller apps.

---

## 1. Tab Bar

- Tabs: Home, Explore, Wallet, Profile, Menu
- Height: 64px + safe area
- Active color: `brand.primary`
- Inactive color: `text.tertiary`
- Background: `background.secondary`

---

## 2. Top App Bar

- Height: 48-56px
- Left: Title or back button
- Right: Context actions (filter, share)
- Background: `background.secondary`

---

## 3. Search Bar

- Height: 44px
- Radius: 999px (pill)
- Leading icon: search (20px)
- Placeholder: “Search deals...”

---

## 4. Navigation Gestures

- Swipe back enabled by default
- Bottom sheets swipe to dismiss
- Long-press on map markers opens quick actions

---

## Related Documents

**Dependencies**
- DES-04: Section 2
- DES-06: Section 2

**Related Specs**
- DES-01: Section 1
- DES-16: Section 2
- DES-17: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial navigation component specs |
