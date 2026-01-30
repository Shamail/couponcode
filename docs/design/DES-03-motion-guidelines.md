---
document_id: DES-03
version: 2.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-01
  - BRAND-02
related_documents:
  - DES-08
  - DES-13
  - PRD-01
  - PRD-02
---

# DES-03: Motion Guidelines

## 1. Executive Summary

Motion supports clarity and reassurance in a modern, utility-first interface. Animations should feel calm, polished, and intentional—never flashy or distracting.

This guide specifies default durations, easing, and motion patterns for key interactions.

---

## 2. Motion Principles

1. **Guide, don’t distract**
2. **Gentle pacing** (avoid rapid loops)
3. **Consistent direction** (gesture direction = motion direction)
4. **Accessible by default** (respect Reduce Motion)

---

## 3. Default Timings

| Type | Duration | Easing |
| --- | --- | --- |
| Micro interactions | 150-220ms | ease-out |
| Page transitions | 280-360ms | ease-in-out |
| Map pulses | 3000ms | ease-in-out |
| Success feedback | 450-700ms | ease-out |
| Toasts | 200-250ms | ease-in-out |

---

## 4. Component Motion Patterns

### Map
- **Deal marker pulse:** scale 0.95 → 1.05, 3s cycle
- **User location pulse:** subtle scale, 3s cycle
- **New marker:** 150ms fade-in + 8% scale

### Bottom Sheets
- Use native spring
- No overshoot
- Dismiss via swipe down

### Buttons
- Pressed state: 150ms scale to 0.98
- Disabled state: instant opacity change

### Feedback
- Success: ripple + confirmation (450-700ms)
- Error: quick shake (150-200ms) then static

---

## 5. Depth & Shadows

- Avoid neon glows
- Use neutral elevation shadows only
- Prefer opacity shifts over brightness spikes

---

## 6. Reduce Motion Support

If device has Reduce Motion enabled:
- Replace pulses with static highlights
- Shorten success animation to 150ms
- Disable looping motion on the map
- Reduce parallax in map transitions

---

## 7. Implementation Notes

Motion tokens live in `DES-04` and should be reused in UI components to avoid mismatched timing.

---

## 8. Related Documents

**Dependencies**
- DES-01: Section 4
- BRAND-02: Section 3

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 4
- DES-08: Section 6
- DES-13: Section 3

**Implementation Guides**
- TECH-05: Section 4

## 9. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 2.0 | 2026-01-30 | Design Lead | Expanded motion patterns and Reduce Motion support |
| 1.1 | 2026-01-30 | Design Lead | Updated motion for modern utility style |
| 1.0 | 2026-01-30 | Design Lead | Initial motion guidelines |
