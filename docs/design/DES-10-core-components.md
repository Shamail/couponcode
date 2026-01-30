---
document_id: DES-10
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
  - DES-08
related_documents:
  - DES-01
  - DES-11
  - DES-13
  - DES-15
---

# DES-10: Core Components

## Executive Summary

Core components include buttons, inputs, and selection controls used across all primary flows. These components must follow token and state standards for consistency.

---

## 1. Buttons

### Sizes
| Size | Height | Padding | Usage |
| --- | --- | --- | --- |
| Large | 52 | 16px | Primary CTAs |
| Medium | 44 | 14px | Standard actions |
| Small | 36 | 12px | Secondary or inline |

### Variants
| Variant | Background | Text | Notes |
| --- | --- | --- | --- |
| Primary | `brand.primary` | White | Single primary per screen |
| Secondary | `background.elevated` | `text.primary` | Optional secondary |
| Ghost | Transparent | `text.primary` | Low emphasis |

---

## 2. Text Inputs

- Height: 44px minimum
- Radius: 10px
- Padding: 12-16px
- Placeholder uses `text.tertiary`
- Helper/error text uses `bodySmall`

### Input States
- Focus: border `brand.primary`
- Error: border `semantic.error` + helper text

---

## 3. Selection Controls

| Control | Size | Notes |
| --- | --- | --- |
| Checkbox | 20px | Square with 4px radius |
| Radio | 20px | Circular |
| Switch | 44x26px | Use platform-native where possible |
| Segmented control | 32-40px height | 2-4 segments |

---

## 4. Pickers

- Date/time pickers use native controls where possible.
- Location pickers should default to map view with search.

---

## 5. Accessibility

- Minimum touch target: 44x44px
- All inputs require labels (visible or programmatic)

---

## Related Documents

**Dependencies**
- DES-04: Section 2
- DES-08: Section 2

**Related Specs**
- DES-01: Section 2
- DES-11: Section 2
- DES-13: Section 2
- DES-15: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial core component specifications |
