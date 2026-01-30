---
document_id: DES-08
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
related_documents:
  - DES-10
  - DES-11
  - DES-13
  - DES-15
---

# DES-08: Component States

## Executive Summary

This document defines standardized UI states for components across the Frictionless apps. Consistent states reduce ambiguity and improve accessibility.

---

## 1. Standard States

| State | Definition | Visual Treatment |
| --- | --- | --- |
| Default | Idle, ready state | Base surface and text |
| Pressed | Active interaction | 6-10% darker surface, subtle scale |
| Focused | Keyboard or switch focus | 2px outline in `brand.primary` |
| Disabled | Not available | 40% opacity, no shadow |
| Loading | In-progress | Spinner + disabled action |
| Error | Validation or failure | `semantic.error` text, outline |
| Success | Completed | `semantic.success` accent |

---

## 2. Buttons

| Variant | Default | Pressed | Disabled |
| --- | --- | --- | --- |
| Primary | `brand.primary` + white text | 8% darker | 40% opacity |
| Secondary | `background.elevated` + text | Darker surface | 40% opacity |
| Ghost | Transparent + text | 6% overlay | 40% opacity |

---

## 3. Inputs

- Default: border `#2D2D2D`
- Focus: border `brand.primary`
- Error: border `semantic.error` + helper text
- Disabled: 40% opacity, no cursor

---

## 4. Cards & List Items

- Default: `background.elevated`
- Pressed: overlay `rgba(255,255,255,0.04)`
- Selected: 1px border `brand.primary`

---

## 5. Loading & Empty States

- Use skeletons for lists
- Loading indicators at 16-24px
- Empty states include CTA when possible

---

## 6. Motion Rules

- Pressed feedback: 150-200ms ease-out
- Loading spinners: 1s linear rotation
- Success state: 300-500ms fade/scale

---

## Related Documents

**Dependencies**
- DES-04: Section 2

**Related Specs**
- DES-10: Section 3
- DES-11: Section 2
- DES-13: Section 2
- DES-15: Section 4

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Standardized component states |
