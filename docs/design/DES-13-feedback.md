---
document_id: DES-13
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
  - DES-10
  - DES-11
  - DES-15
---

# DES-13: Feedback Components

## Executive Summary

Feedback components communicate system status, success, and error states without interrupting the flow. They must be clear, brief, and unobtrusive.

---

## 1. Toasts & Snackbars

- Placement: bottom, above tab bar
- Duration: 2.5s (auto-dismiss)
- Height: 40-48px
- Use for transient info

---

## 2. Alerts & Dialogs

- Use for critical actions only
- Title + one sentence max
- Primary CTA on right in LTR

---

## 3. Loading States

| Type | Use | Spec |
| --- | --- | --- |
| Spinner | Short actions | 16-24px, 1s rotation |
| Skeleton | Lists | 3-5 rows, shimmer 1.2s |
| Progress bar | Long tasks | Linear, 4px height |

---

## 4. Empty & Error States

- Empty: short message + CTA
- Error: clear recovery action
- Avoid large illustrations in map contexts

---

## 5. Haptics

- Success: light haptic on redeem
- Error: medium haptic on failure
- Avoid repeated vibration loops

---

## Related Documents

**Dependencies**
- DES-04: Section 2
- DES-08: Section 1

**Related Specs**
- DES-01: Section 4
- DES-10: Section 1
- DES-11: Section 2
- DES-15: Section 4

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial feedback component system |
