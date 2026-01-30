---
document_id: BRAND-06
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
  - BRAND-04
  - BRAND-05
related_documents:
  - DES-07
  - BRAND-10
  - BRAND-11
---

# BRAND-06: Iconography

## Executive Summary

Icons in Frictionless are minimal, friendly, and immediately readable. The icon system supports map discovery and quick actions without visual noise.

---

## 1. Style Rules

- **Stroke style:** Outline-only, 1.5px stroke on 24px grid
- **Corner radius:** Subtle rounding, 2px min radius
- **Perspective:** Flat, no 3D or shadows
- **Detail level:** Low; remove interior lines unless required for clarity

---

## 2. Grid & Sizing

| Size | Use |
| --- | --- |
| 16px | Inline labels, metadata |
| 20px | Secondary actions |
| 24px | Primary actions, tab icons |
| 32px | Empty states, onboarding |

All icons are drawn on a 24x24 grid and scaled as needed. Maintain a 2px padding from the grid edge.

---

## 3. Core Icon Set

| Category | Icons |
| --- | --- |
| Navigation | Home, Explore, Wallet, Profile, Menu |
| Actions | Search, Filter, Share, Save |
| Deals | Tag, Flash, Timer, Verified |
| Map | Pin, Target, Compass, Route |
| Feedback | Check, Close, Warning, Info |
| Commerce | Store, Receipt, QR, Scan |

---

## 4. Color Usage

- Default color: `text.secondary`
- Active color: `brand.primary`
- Alert color: `semantic.warning` or `semantic.error`

Avoid multi-color icons unless in illustrations.

---

## 5. Do / Don't

**Do**
- Use consistent stroke weight
- Keep icons simple and balanced
- Align icon direction with LTR/RTL (e.g., arrows)

**Don't**
- Mix filled and outline styles
- Add gradients or drop shadows
- Use decorative icons in core flows

---

## 6. Custom Icon Creation

1. Start from the 24px grid template.
2. Use 1.5px strokes aligned to pixel grid.
3. Test at 16px and 20px for readability.
4. Export in SVG and include in asset library.

---

## 7. Accessibility

Icons must always be paired with accessible text labels or `accessibilityLabel` in UI code. Do not rely on icon-only actions without text alternative.

---

## Related Documents

**Dependencies**
- BRAND-00: Section 4
- BRAND-04: Section 3
- BRAND-05: Section 2

**Related Specs**
- DES-07: Section 2
- BRAND-10: Section 3
- BRAND-11: Section 2

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial iconography system |
