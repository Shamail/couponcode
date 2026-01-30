---
document_id: BRAND-08
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Brand Lead
dependencies:
  - BRAND-00
  - BRAND-04
related_documents:
  - DES-01
  - DES-17
  - BRAND-03
---

# BRAND-08: Map Visual Identity

## Executive Summary

The map is the signature surface of Frictionless. It must feel calm, precise, and trustworthy, with minimal visual noise and a focus on nearby savings.

---

## 1. Base Map Styling

- **Base style:** Mapbox Dark v11 (or custom Frictionless dark style)
- **Primary background:** `#121212`
- **Muted roads and labels** to reduce clutter

| Layer | Color | Opacity |
| --- | --- | --- |
| Background | `#121212` | 100% |
| Land | `#1E1E1E` | 100% |
| Roads | `#2D2D2D` | 80-100% |
| Labels | `#9CA3AF` | 60-80% |

---

## 2. Marker System

### Deal Marker
- Color: `map.dealMarker` (`#F59E0B`)
- Shape: rounded pill with value label
- Optional pulse for limited-time deals

### User Location
- Color: `map.userLocation` (`#3B82F6`)
- Soft pulse, 3s cycle
- Heading indicator enabled

### Flash Deal
- Color: `map.flashDeal` (`#EF4444`)
- Short, subtle pulse (3s)

---

## 3. Heatmap Identity (Seller)

- Gradient: blue → amber → red
- Opacity controlled by zoom (fade on high zoom)
- No high-saturation neon

---

## 4. Typography on Map

- Labels must be **secondary**, not dominant
- Use `bodySmall` or `labelSmall`
- Place overlays on solid scrims when necessary

---

## 5. Motion & Interaction

- Marker pulse: subtle scale (0.95 → 1.05)
- Map transitions: ease-in-out 280-360ms
- No looping animations outside location and deal pulses

---

## 6. Accessibility

- Provide high-contrast overlay mode
- Ensure markers have text labels for screen readers
- Avoid placing critical text directly on busy tiles

---

## Related Documents

**Dependencies**
- BRAND-00: Section 5
- BRAND-04: Section 4

**Related Specs**
- DES-01: Section 1
- DES-17: Section 2
- BRAND-03: Section 2

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Brand Lead | Initial map visual identity system |
