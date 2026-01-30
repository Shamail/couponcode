---
document_id: DES-09
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-04
  - BRAND-04
related_documents:
  - DES-01
  - DES-06
  - DES-08
---

# DES-09: Theming Architecture

## Executive Summary

Frictionless uses a dark-first theme with optional light and high-contrast variants. Theming is token-driven and designed to preserve usability on map-heavy screens.

---

## 1. Theme Structure

Themes are defined as overrides of core tokens.

```typescript
export const themes = {
  dark: {
    background: {
      primary: '#121212',
      secondary: '#1E1E1E',
      elevated: '#2D2D2D',
    },
    text: {
      primary: '#F9FAFB',
      secondary: '#9CA3AF',
      tertiary: '#6B7280',
    },
  },
  light: {
    background: {
      primary: '#FFFFFF',
      secondary: '#F5F5F7',
      elevated: '#FFFFFF',
    },
    text: {
      primary: '#0F172A',
      secondary: '#334155',
      tertiary: '#64748B',
    },
  },
  highContrast: {
    background: {
      primary: '#000000',
      secondary: '#111111',
      elevated: '#1A1A1A',
    },
    text: {
      primary: '#FFFFFF',
      secondary: '#E5E7EB',
      tertiary: '#D1D5DB',
    },
  },
} as const;
```

---

## 2. Default Theme Rules

- **Dark** is the default theme for both apps.
- Light mode is allowed for marketing or specific user preferences.
- High-contrast mode can be toggled via accessibility settings.

---

## 3. Map Styling by Theme

| Theme | Map Style | Notes |
| --- | --- | --- |
| Dark | Mapbox Dark v11 | Default |
| Light | Light/Neutral style | Reduce glare |
| High Contrast | Custom high-contrast | Increased label brightness |

---

## 4. Component Overrides

Component tokens should rely on theme values rather than custom colors. Exceptions require design approval.

---

## Related Documents

**Dependencies**
- DES-04: Section 2
- BRAND-04: Section 2

**Related Specs**
- DES-01: Section 1
- DES-06: Section 4
- DES-08: Section 1

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial theming architecture |
