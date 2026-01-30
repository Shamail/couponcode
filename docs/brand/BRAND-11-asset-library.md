---
document_id: BRAND-11
version: 1.0
status: Final
priority: P2
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-03
  - BRAND-04
  - BRAND-05
  - BRAND-06
  - BRAND-07
related_documents:
  - BRAND-10
---

# BRAND-11: Asset Library Index

## Executive Summary

This document defines the structure, naming standards, and governance for Frictionless brand assets. The asset library is the single source of truth for logos, icons, photography, and templates.

---

## 1. Repository Structure

| Folder | Contents |
| --- | --- |
| `/logos` | Master logos, wordmarks, icon marks |
| `/icons` | SVG icon set and exported sizes |
| `/typography` | Typeface files and licenses |
| `/imagery` | Approved photography and stock |
| `/templates` | Social, print, pitch, and email templates |
| `/map-styles` | Mapbox styles and exports |

---

## 2. Naming Conventions

Format: `category_variant_color_version.ext`

Example: `logo_primary_indigo_v1.svg`

Rules:
- Use lowercase with underscores
- Include version number for every master asset
- Avoid spaces and special characters

---

## 3. Versioning & Approvals

- **Master assets** are versioned and locked.
- **Derived assets** (exports, crops) inherit the master version.
- New versions require approval from Brand Lead.

---

## 4. Licensing & Attribution

- All external imagery must include license metadata in a `LICENSES.md` file.
- Stock or partner imagery must be cleared for commercial use in Morocco.

---

## 5. Access & Requests

- Designers have read/write access.
- Non-design teams should request assets through the Brand Lead.
- Urgent requests should include usage context and deadline.

---

## Related Documents

**Dependencies**
- BRAND-03: Section 8
- BRAND-04: Section 8
- BRAND-05: Section 6
- BRAND-06: Section 6
- BRAND-07: Section 7

**Related Specs**
- BRAND-10: Section 6

**Implementation Guides**
- OPS-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial asset library structure |
