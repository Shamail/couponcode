---
document_id: BRAND-09
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
  - BRAND-05
related_documents:
  - BRAND-01
  - DES-18
  - DES-06
---

# BRAND-09: Localization Guidelines

## Executive Summary

Frictionless is built for Morocco with Arabic (Darija/MSA) and French as primary languages. Localization should feel native, respectful, and practical, prioritizing clarity over literal translation.

---

## 1. Language Priorities

| Language | Priority | Notes |
| --- | --- | --- |
| Arabic (Darija) | Primary | Default for most users |
| French | Primary | Widely used in urban centers |
| English | Secondary | For tourists and expats |

---

## 2. Tone Adaptation

- **Arabic (Darija):** Warm, simple, conversational
- **French:** Polite and concise, avoid formal corporate tone
- **English:** Direct and clear, short sentences

Avoid slang or hype across all languages.

---

## 3. RTL & Layout

- Mirror directional icons (chevrons, arrows)
- Keep action buttons in the same physical position for muscle memory
- Avoid centering long Arabic strings; prefer left/right alignment by direction

---

## 4. Numerals, Currency, Dates

| Element | Arabic UI | French UI | English UI |
| --- | --- | --- | --- |
| Currency | `د.م` | `DH` | `DH` |
| Numerals | Arabic-Indic | Latin | Latin |
| Date | DD/MM/YYYY | DD/MM/YYYY | DD/MM/YYYY |
| Time | 24h preferred | 24h preferred | 24h preferred |

Always show distances in meters/kilometers (m, km).

---

## 5. Content Length & Expansion

- French strings can expand **20-30%** vs English
- Arabic can expand vertically; allow 2 lines for key CTAs
- Avoid truncation on primary actions

---

## 6. Cultural Guidelines

- Respect prayer times and Friday patterns in notifications
- Avoid marketing pushes during late-night hours
- Use local examples (dirhams, neighborhoods, cafés)

---

## 7. Localization Checklist

- [ ] All CTA strings tested in Arabic and French
- [ ] RTL layout verified on key screens
- [ ] Currency and numerals correct
- [ ] Text fits without truncation
- [ ] Icons mirrored where required

---

## Related Documents

**Dependencies**
- BRAND-00: Section 3
- BRAND-05: Section 4

**Related Specs**
- BRAND-01: Section 8
- DES-06: Section 5
- DES-18: Section 2

**Implementation Guides**
- GUIDE-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial localization and RTL guidance |
