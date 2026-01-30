---
document_id: DES-02
version: 2.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-01
  - DES-04
related_documents:
  - PRD-01
  - PRD-02
  - DES-03
  - DES-08
  - DES-18
  - GUIDE-01
  - GUIDE-02
---

# DES-02: Accessibility Guidelines

## 1. Executive Summary

This document defines accessibility standards for the Frictionless Buyer and Seller apps. The goal is to ensure usable experiences across devices, lighting conditions, and user abilities, especially in map-heavy contexts.

---

## 2. Core Standards (WCAG AA)

| Area | Standard |
| --- | --- |
| Text size | Minimum 14pt body, 16pt for primary actions |
| Touch target | Minimum 44x44 px |
| Contrast | 4.5:1 for body text, 3:1 for large text |
| Focus states | Visible for keyboard and switch controls |
| Motion | Respect Reduce Motion setting |

---

## 3. Color & Contrast

- Avoid placing primary text on busy map tiles without a scrim.
- Maintain contrast for text over brand colors.
- Provide **high-contrast mode** for map overlays.

---

## 4. Typography & Readability

- Use `bodyMedium` or larger for primary content.
- Avoid all caps for Arabic and French.
- Maintain line lengths between 30-45 characters on mobile.

---

## 5. Focus Order & Navigation

- Logical focus order top-to-bottom, left-to-right (RTL mirrored).
- Provide visible focus ring in `brand.primary`.
- Avoid focus traps in bottom sheets and modals.

---

## 6. Screen Reader Support

- Every interactive element must have an accessible label.
- Provide accessibility hints for non-obvious actions.
- Announce dynamic updates (e.g., “Deal claimed”).

---

## 7. Motion & Animation

- Respect system **Reduce Motion** setting.
- Avoid flashing content more than 3 times per second.
- Provide non-animated feedback alternatives for redemption.

---

## 8. Map-Specific Guidelines

- Provide zoom controls for one-handed use.
- Use distinct shapes and labels, not only color.
- Ensure markers are discoverable by screen readers.

---

## 9. Forms & Inputs

- Pair labels with inputs (visible or programmatic).
- Inline errors must be announced to screen readers.
- Avoid placeholder-only labels.

---

## 10. Testing Procedures

### Manual Testing
- Screen reader flows (VoiceOver, TalkBack)
- Reduce Motion on/off
- High contrast mode
- One-handed reachability

### Automated Checks
- Contrast ratio scanning
- Accessibility linting in CI
- Snapshot testing for focus rings

---

## 11. Accessibility Checklist

- [ ] All CTAs meet 44x44px minimum
- [ ] Contrast ratios pass AA
- [ ] Screen reader labels on all actions
- [ ] Reduced motion supported
- [ ] RTL layouts verified

---

## 12. Related Documents

**Dependencies**
- DES-01: Section 7
- DES-04: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 4
- DES-03: Section 6
- DES-08: Section 1
- DES-18: Section 5

**Implementation Guides**
- GUIDE-01: Section 2
- GUIDE-02: Section 2

## 13. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 2.0 | 2026-01-30 | Design Lead | Expanded WCAG coverage and testing |
| 1.0 | 2026-01-30 | Design Lead | Initial accessibility guidelines |
