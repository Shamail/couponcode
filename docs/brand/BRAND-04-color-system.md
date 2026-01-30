---
document_id: BRAND-04
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
related_documents:
  - BRAND-03
  - BRAND-05
  - BRAND-08
  - DES-04
  - DES-09
---

# BRAND-04: Color System

## Executive Summary

The Frictionless color system balances calm, dark-mode utility with clear, vibrant value cues. The palette is optimized for legibility on maps, quick savings recognition, and a premium, trustworthy feel.

---

## 1. Core Brand Palette

| Token | Name | Hex | Usage |
| --- | --- | --- | --- |
| `brand.primary` | Indigo | `#4F46E5` | Primary actions, brand accents |
| `brand.accent` | Emerald | `#10B981` | Savings confirmation, success |

---

## 2. Neutral System (Dark First)

| Token | Name | Hex | Usage |
| --- | --- | --- | --- |
| `background.primary` | Night | `#121212` | Primary surfaces |
| `background.secondary` | Charcoal | `#1E1E1E` | Elevated panels |
| `background.elevated` | Graphite | `#2D2D2D` | Cards, sheets |
| `text.primary` | Mist | `#F9FAFB` | Primary text |
| `text.secondary` | Fog | `#9CA3AF` | Secondary text |
| `text.tertiary` | Slate | `#6B7280` | Muted labels |
| `text.inverse` | Ink | `#121212` | Text on light surfaces |

---

## 3. Functional Colors

| Token | Name | Hex | Usage |
| --- | --- | --- | --- |
| `semantic.success` | Emerald | `#10B981` | Success state |
| `semantic.warning` | Amber | `#F59E0B` | Caution, deal value |
| `semantic.error` | Red | `#EF4444` | Errors, urgency |
| `semantic.info` | Blue | `#3B82F6` | Info, location |

---

## 4. Map-Specific Palette

| Element | Token | Hex |
| --- | --- | --- |
| User location | `map.userLocation` | `#3B82F6` |
| Deal marker | `map.dealMarker` | `#F59E0B` |
| Flash deal | `map.flashDeal` | `#EF4444` |
| Heatmap cool | `heatmap.cool` | `rgba(59, 130, 246, 0.35)` |
| Heatmap warm | `heatmap.warm` | `rgba(245, 158, 11, 0.6)` |
| Heatmap hot | `heatmap.hot` | `rgba(239, 68, 68, 0.9)` |

---

## 5. Light Surface Use (Marketing Only)

Use light surfaces for web, print, and merchant collateral. Pair with `text.inverse` and keep brand accents intact.

| Surface | Hex | Notes |
| --- | --- | --- |
| Light base | `#FFFFFF` | Marketing pages, print |
| Soft tint | `#F5F5F7` | Card backgrounds |
| Ink text | `#0F172A` | Long-form copy |

---

## 6. Accessibility & Contrast

- Minimum contrast ratio: **4.5:1** for body text, **3:1** for large text.
- Avoid low-contrast label text on map tiles.
- When using `brand.primary` on dark surfaces, increase font weight to 600+ for small text.

---

## 7. Usage Rules

**Do**
- Use brand colors sparingly to highlight value and action.
- Keep backgrounds neutral and calm.
- Use semantic colors consistently across components.

**Don't**
- Use red or amber for non-urgent content.
- Introduce new accent colors without approval.
- Place primary actions on low-contrast surfaces.

---

## 8. Color Implementation

Design tokens live in `DES-04` and should be the single source of truth for engineering. Any palette changes must be coordinated with brand and design leads.

---

## Related Documents

**Dependencies**
- BRAND-00: Section 5

**Related Specs**
- BRAND-03: Section 3
- BRAND-05: Section 2
- BRAND-08: Section 2
- DES-04: Section 2
- DES-09: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial color system and usage rules |
