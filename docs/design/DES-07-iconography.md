---
document_id: DES-07
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - BRAND-06
  - DES-04
related_documents:
  - DES-10
  - DES-12
  - BRAND-06
---

# DES-07: Iconography

## Executive Summary

This document translates the brand icon style into UI specifications for product teams. It defines sizing, alignment, and usage rules for app icons.

---

## 1. Standard Sizes

| Size | Use |
| --- | --- |
| 16px | Inline meta and small labels |
| 20px | Secondary actions |
| 24px | Primary actions and tabs |
| 32px | Empty states and onboarding |

---

## 2. Grid & Stroke

- Base grid: **24x24**
- Stroke weight: **1.5px** at 24px
- Round caps and joins for friendliness

---

## 3. Alignment Rules

- Align to pixel grid at export sizes.
- Keep optical balance (center of mass, not geometric center).
- Mirrored icons in RTL (arrows, navigation, reply).

---

## 4. Icon Usage by Component

| Component | Size | Color |
| --- | --- | --- |
| Tab bar | 24px | `text.tertiary` / `brand.primary` |
| Search bar | 20px | `text.secondary` |
| Buttons | 16-20px | Inherit text color |
| Toasts | 20px | `semantic.*` |

---

## 5. Accessibility

Icons must include label text or accessibility labels. Icon-only buttons require a visible or screen-reader label.

---

## Related Documents

**Dependencies**
- BRAND-06: Section 2
- DES-04: Section 2

**Related Specs**
- DES-10: Section 2
- DES-12: Section 2
- BRAND-06: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial UI iconography specifications |
