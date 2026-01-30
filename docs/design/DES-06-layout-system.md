---
document_id: DES-06
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
  - BRAND-05
related_documents:
  - DES-01
  - DES-18
  - BRAND-09
---

# DES-06: Layout System

## Executive Summary

The layout system ensures predictable spacing, hierarchy, and safe-area behavior across the Buyer and Seller apps. It is optimized for map overlays, bottom sheets, and mobile-first layouts.

---

## 1. Grid & Spacing

- **Base grid:** 8pt
- Use spacing tokens from `DES-04`.
- Avoid custom spacing values.

| Token | Value | Typical Use |
| --- | --- | --- |
| `sm` | 8 | Tight stacks |
| `md` | 16 | Standard padding |
| `lg` | 24 | Section breaks |
| `xl` | 32 | Major section spacing |

---

## 2. Screen Margins

| Surface | Margin |
| --- | --- |
| Mobile (phone) | 16px |
| Large phone | 20px |
| Tablet | 24px |

Map overlays should align to these margins for visual consistency.

---

## 3. Safe Areas

- Always respect top and bottom safe areas.
- Floating controls should avoid the bottom 44px on iOS.
- Bottom sheets must account for the gesture bar.

---

## 4. Layout Patterns

### Map + Overlay
- Map is full bleed.
- Overlays should be **elevated** surfaces with 16px padding.
- Overlays should not exceed 30% of vertical height unless expanded.

### List + Detail
- Primary list at full width
- Detail sheets use 40/70/92% snap points

---

## 5. RTL Layout Behavior

- Mirror horizontal padding and alignment.
- Keep **primary action position** consistent for muscle memory.
- Tabs remain in the same order; labels switch direction only.

---

## Related Documents

**Dependencies**
- DES-04: Section 4
- BRAND-05: Section 2

**Related Specs**
- DES-01: Section 1
- DES-18: Section 3
- BRAND-09: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial layout system |
