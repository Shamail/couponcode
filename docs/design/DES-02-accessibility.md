---
document_id: DES-02
version: 1.0
status: Final
priority: P2
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-01
related_documents:
  - PRD-01
  - PRD-02
  - GUIDE-01
  - GUIDE-02
---

# DES-02: Accessibility Guidelines

## 1. Executive Summary

This document defines accessibility standards for the Frictionless Buyer and Seller apps. The goal is to ensure usable experiences across devices, lighting conditions, and user abilities.

Accessibility is critical for map-based interfaces where small UI elements and low-contrast overlays can cause friction.

## 2. Core Standards

| Area | Standard |
| --- | --- |
| Text size | Minimum 14pt body, 16pt for primary actions |
| Touch target | Minimum 44x44 px |
| Contrast | 4.5:1 for body text, 3:1 for large text |
| Focus states | Visible for keyboard and switch controls |

## 3. Map-Specific Guidelines

- Provide a **high-contrast mode** for map overlays
- Use distinct shapes and patterns, not just color
- Avoid placing critical text on busy map tiles
- Provide zoom controls for one-handed use

## 4. Motion & Accessibility

- Respect system “Reduce Motion” setting
- Avoid flashing animations > 3 times per second
- Provide alternative non-animated feedback for redemption

## 5. Localization Considerations

- Ensure Arabic and French text fits on primary CTAs
- Support right-to-left layouts in buyer onboarding

## 6. Related Documents

**Dependencies**
- DES-01: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 4

**Implementation Guides**
- GUIDE-01: Section 2

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial accessibility guidelines |
