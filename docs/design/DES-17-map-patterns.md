---
document_id: DES-17
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-01
  - BRAND-08
related_documents:
  - DES-12
  - DES-16
  - DES-18
---

# DES-17: Map Patterns

## Executive Summary

Map patterns define how users discover and interact with deals on the map. These patterns emphasize speed, clarity, and safety.

---

## 1. Marker Interaction

- Tap marker to open deal sheet
- Long-press to pin and show quick actions
- Cluster markers at low zoom; expand on zoom in

---

## 2. Overlay Behavior

- Floating search bar stays above map at all times
- Deal toasts appear above tab bar
- Range ring shows active discovery radius

---

## 3. Map Search

- Search results appear as list + highlight on map
- Prioritize nearby deals over distant ones

---

## 4. Navigation to Merchant

- Provide route options: walk and drive
- Distance shown in meters up to 1km, then in km

---

## 5. Safety & Accuracy

- Warn users if GPS is low accuracy
- Use passive location when app is in background

---

## Related Documents

**Dependencies**
- DES-01: Section 1
- BRAND-08: Section 2

**Related Specs**
- DES-12: Section 1
- DES-16: Section 1
- DES-18: Section 3

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial map interaction patterns |
