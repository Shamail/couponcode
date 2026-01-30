---
document_id: DES-14
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
  - DES-11
---

# DES-14: Data Display Components

## Executive Summary

Data display components present savings, distances, and membership information with consistent hierarchy and readability.

---

## 1. Lists

- Default row height: 56px
- Leading icon/avatar: 32px
- Title: `bodyMedium`, 500 weight
- Subtitle: `bodySmall`, `text.secondary`

---

## 2. Badges & Chips

| Type | Size | Usage |
| --- | --- | --- |
| Status badge | 24-28px height | Ready / Expired |
| Value chip | 28-32px height | “25% off” |
| Tier badge | 24px | Bronze/Silver/Gold |

---

## 3. Avatars

- Sizes: 24, 32, 40
- Default background: `background.secondary`
- Initials in `text.primary`

---

## 4. Progress Indicators

- Linear: 4px height, radius 2px
- Circular: 32px default
- Use `semantic.success` for achieved progress

---

## Related Documents

**Dependencies**
- DES-04: Section 2
- DES-08: Section 1

**Related Specs**
- DES-01: Section 2
- DES-10: Section 1
- DES-11: Section 1

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial data display component specs |
