---
document_id: DES-03
version: 1.1
status: Final
priority: P2
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-01
  - BRAND-02
related_documents:
  - PRD-01
  - PRD-02
---

# DES-03: Motion Guidelines

## 1. Executive Summary

Motion supports clarity and reassurance in a modern, utility-first interface. Animations should feel calm, polished, and intentional—never flashy or distracting.

This guide specifies default durations, easing, and motion patterns for key interactions.

## 2. Motion Principles

1. **Guide, don’t distract**
2. **Gentle pacing** (avoid rapid loops)
3. **Consistent direction** (gesture direction = motion direction)
4. **Accessible by default** (respect Reduce Motion)

## 3. Default Timings

| Type | Duration | Easing |
| --- | --- | --- |
| Micro interactions | 150-220ms | ease-out |
| Page transitions | 280-360ms | ease-in-out |
| Map pulses | 3000ms | ease-in-out |
| Success feedback | 450-700ms | ease-out |

## 4. Key Animations

- **Deal marker pulse:** scale 0.95 → 1.05, 3s cycle
- **User location pulse:** subtle scale, 3s cycle
- **Bottom sheets:** platform-native spring, no overshoot
- **Success feedback:** short ripple + confirmation, no full-screen flash

## 5. Depth & Shadows

- Remove neon glows
- Use neutral elevation shadows only
- Prefer opacity shifts over brightness spikes

## 6. Reduce Motion Support

If device has Reduce Motion enabled:
- Replace pulses with static highlights
- Shorten success animation to 150ms
- Disable looping motion on the map

## 7. Related Documents

**Dependencies**
- DES-01: Section 2
- BRAND-02: Section 3

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 4

**Implementation Guides**
- TECH-05: Section 4

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Design Lead | Updated motion for modern utility style |
| 1.0 | 2026-01-30 | Design Lead | Initial motion guidelines |
