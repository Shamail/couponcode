---
document_id: BRAND-05
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
  - BRAND-04
related_documents:
  - BRAND-09
  - DES-04
  - DES-06
---

# BRAND-05: Typography

## Executive Summary

Typography communicates trust and clarity in Frictionless. The system is built for legibility on dark surfaces, multilingual consistency (Arabic/French/English), and fast scanning in map and deal contexts.

---

## 1. Typeface Families

### Primary Latin (EN/FR)
- **Family:** Manrope
- **Weights:** 400, 500, 600, 700, 800
- **Tone:** Modern, neutral, easy to scan

### Arabic (Darija/MSA)
- **Family:** Noto Sans Arabic
- **Weights:** 400, 500, 600, 700
- **Tone:** Clean, contemporary, highly legible

### Fallbacks
- iOS: SF Pro / SF Arabic
- Android: Roboto / Noto Sans Arabic
- Web: `system-ui, -apple-system, Segoe UI, Roboto, Noto Sans Arabic`

---

## 2. Type Scale

| Token | Size | Weight | Line Height | Usage |
| --- | --- | --- | --- | --- |
| `displayLarge` | 48 | 800 | 56 | Launch screens, hero banners |
| `displayMedium` | 32 | 700 | 40 | Major headlines |
| `h1` | 28 | 700 | 36 | Screen titles |
| `h2` | 24 | 600 | 32 | Section titles |
| `h3` | 20 | 600 | 28 | Card titles |
| `bodyLarge` | 18 | 400 | 28 | Primary body |
| `bodyMedium` | 16 | 400 | 24 | Default body |
| `bodySmall` | 14 | 400 | 20 | Secondary body |
| `labelLarge` | 16 | 600 | 24 | Primary button |
| `labelMedium` | 14 | 500 | 20 | Secondary label |
| `labelSmall` | 12 | 500 | 16 | Meta text |

---

## 3. Numerals & Currency

- Use **Latin numerals** for prices by default (e.g., 25 DH) unless the entire UI is Arabic.
- For Arabic UI, switch to **Arabic-Indic numerals** and keep currency suffix `د.م`.
- Always show currency as **DH** or **د.م**; avoid $ or €.

---

## 4. Language-Specific Guidance

| Language | Preferred Tone | Notes |
| --- | --- | --- |
| English | Short and direct | Use sentence case |
| French | Clear and polite | Avoid literal translations |
| Arabic (Darija) | Warm and simple | Keep lines short for RTL |

Keep Arabic headings one line where possible; break long lines into two short lines instead of one long line.

---

## 5. Accessibility & Legibility

- Minimum body size: **14pt**
- Increase weight to **500+** for text over complex map tiles.
- Avoid all caps for Arabic and French.
- Maintain contrast ratios from BRAND-04.

---

## 6. Implementation Notes

Typography tokens live in `DES-04`. All app styles should reference tokens, not hard-coded values.

---

## Related Documents

**Dependencies**
- BRAND-00: Section 4
- BRAND-04: Section 2

**Related Specs**
- DES-04: Section 3
- DES-06: Section 2
- BRAND-09: Section 4

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial typography system |
