---
document_id: DES-11
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
  - DES-08
related_documents:
  - DES-01
  - DES-10
  - DES-13
---

# DES-11: Container Components

## Executive Summary

Container components structure content and manage elevation across screens. They provide hierarchy without visual noise.

---

## 1. Cards

- Background: `background.elevated`
- Radius: 12-16px
- Padding: 16px
- Elevation: `elevation.md`

### Card Types
| Type | Use |
| --- | --- |
| Deal card | Show savings + distance |
| Info card | Informational text |
| Wallet card | Past/active deals |

---

## 2. Bottom Sheets

- Library: `@gorhom/bottom-sheet`
- Snap points: 40%, 70%, 92%
- Handle: 32x4px, radius 2px
- Background: `background.elevated`

---

## 3. Modals

- Use for critical decisions only
- Background scrim: `rgba(0,0,0,0.6)`
- Primary CTA on the right (LTR)

---

## 4. Overlays

- Floating banners: 12-16px padding
- Limit to 2 layers above map

---

## Related Documents

**Dependencies**
- DES-04: Section 5
- DES-08: Section 4

**Related Specs**
- DES-01: Section 2
- DES-10: Section 1
- DES-13: Section 2

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial container component specs |
