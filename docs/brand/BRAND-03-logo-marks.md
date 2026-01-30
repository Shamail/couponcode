---
document_id: BRAND-03
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
  - BRAND-04
  - BRAND-05
related_documents:
  - BRAND-10
  - BRAND-11
  - DES-07
---

# BRAND-03: Logo & Marks

## Executive Summary

This document defines the Frictionless logo system, approved variants, clear space, and usage rules across product, marketing, and partner contexts. Consistent logo usage protects brand recognition and trust.

---

## 1. Logo System Overview

### Primary Logo
- **Primary mark:** Wordmark + icon lockup
- **Default color:** `brand.primary` (`#4F46E5`) on dark backgrounds
- **Preferred background:** `background.primary` (`#121212`)

### Secondary Logo
- **Wordmark only** for narrow or text-only placements
- **Icon only** for app icon, favicons, and small surface marks

---

## 2. Clear Space & Minimum Size

### Clear Space
Maintain clear space equal to **1x the height of the logomark dot** around all sides. No text, icons, or edges should enter this area.

### Minimum Sizes
| Usage | Minimum Width |
| --- | --- |
| App icon (print) | 12 mm |
| App icon (digital) | 48 px |
| Wordmark | 120 px |
| Lockup (wordmark + icon) | 160 px |

---

## 3. Color Variants

| Variant | Usage | Hex |
| --- | --- | --- |
| Primary | Default on dark backgrounds | `#4F46E5` |
| Inverse | On light backgrounds | `#121212` |
| Mono White | Single-color emboss/etch | `#FFFFFF` |
| Mono Dark | Single-color print | `#0F172A` |

**Never** use gradients or multi-color effects on the logo.

---

## 4. Background Control

- Use the **primary logo** only on solid or low-contrast backgrounds.
- If background contrast is low, place the logo on a **solid container** using `background.secondary`.
- Avoid placing logo on top of photographic noise.

---

## 5. App Icon

### Icon Structure
- **Shape:** Squircle radius 22% (iOS), 20% (Android)
- **Content:** Icon mark centered, no wordmark
- **Background:** `brand.primary` or `background.primary` with primary icon in white

### Safe Area
- Keep icon mark inside **80% of the canvas** to avoid cropping on device masks.

---

## 6. Co-Branding Rules

| Scenario | Rule |
| --- | --- |
| Merchant partner | Equal height, 1x spacing between marks |
| Platform partner | Frictionless logo first, then partner |
| Sponsor badge | Place below or trailing, never above |

Do not stretch, rotate, or apply drop shadows.

---

## 7. Incorrect Usage (Do Not)

- Do not change logo colors outside approved palette
- Do not add outlines or glow effects
- Do not skew or rotate
- Do not place on busy or low-contrast imagery
- Do not use the wordmark without approved spacing

---

## 8. File Formats

| Format | Use |
| --- | --- |
| SVG | Web, UI, scalable print |
| PDF | Print, partner kits |
| PNG | Social, slides |

Master files live in the asset library (see BRAND-11).

---

## Related Documents

**Dependencies**
- BRAND-00: Section 2
- BRAND-04: Section 2
- BRAND-05: Section 3

**Related Specs**
- DES-07: Section 2
- BRAND-10: Section 2
- BRAND-11: Section 2

**Implementation Guides**
- GUIDE-01: Section 1

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial logo system specification |
