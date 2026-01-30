---
document_id: DES-18
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-06
  - BRAND-09
related_documents:
  - DES-15
  - PRD-01
  - PRD-02
---

# DES-18: Localization Patterns

## Executive Summary

Localization patterns ensure UI components and layouts behave correctly across Arabic, French, and English. This includes RTL mirroring, text expansion, and numeric formats.

---

## 1. RTL Mirroring

- Mirror navigation arrows and chevrons
- Keep tap targets in consistent positions
- Reverse list item layout where required

---

## 2. Text Expansion

- Allow 20-30% width growth for French
- For Arabic, allow 2 lines where English is 1
- Avoid fixed-width buttons

---

## 3. Numbers & Currency

- Use localized numerals based on language
- Place currency after value (25 DH / 25 د.م)
- Distance uses metric units only

---

## 4. Inputs

- Phone numbers: enforce local formats
- Names: allow Arabic, French, and Latin characters
- Address fields: support street + neighborhood

---

## 5. QA Checklist

- [ ] RTL layout verified on Home, Wallet, Profile
- [ ] Map labels do not conflict with UI
- [ ] All CTAs visible without truncation

---

## Related Documents

**Dependencies**
- DES-06: Section 5
- BRAND-09: Section 2

**Related Specs**
- DES-15: Section 4
- PRD-01: Section 3
- PRD-02: Section 3

**Implementation Guides**
- GUIDE-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial localization patterns |
