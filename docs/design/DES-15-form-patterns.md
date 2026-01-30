---
document_id: DES-15
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-10
  - DES-08
related_documents:
  - DES-13
  - DES-18
  - PRD-02
---

# DES-15: Form Patterns

## Executive Summary

Forms should be short, clear, and forgiving. This document defines validation, error handling, and keyboard behavior for all form flows.

---

## 1. Structure

- Group related fields into sections
- Use clear labels above fields
- Keep one primary CTA at the bottom

---

## 2. Validation

- Validate **on blur** and **on submit**
- Use inline error text, not modal errors
- Provide a recovery hint whenever possible

---

## 3. Error Messaging

| Principle | Example |
| --- | --- |
| Specific | “Enter a valid phone number” |
| Actionable | “Use 10 digits” |
| Calm | Avoid all caps or blame |

---

## 4. Keyboard & Input Modes

- Set input type for numbers, email, phone
- Use next/previous navigation in multi-field forms
- Avoid large scroll jumps when keyboard opens

---

## 5. Multi-Step Forms

- Show progress (Step 1 of 3)
- Allow back navigation without loss of data
- Save draft state for merchant onboarding

---

## Related Documents

**Dependencies**
- DES-10: Section 2
- DES-08: Section 1

**Related Specs**
- DES-13: Section 4
- DES-18: Section 2
- PRD-02: Section 4

**Implementation Guides**
- GUIDE-02: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial form pattern guidance |
