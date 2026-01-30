---
document_id: DES-04
version: 1.0
status: Final
priority: P0
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - BRAND-04
  - BRAND-05
related_documents:
  - DES-01
  - DES-08
  - DES-09
  - DES-10
  - TECH-05
---

# DES-04: Design Tokens

## Executive Summary

Design tokens are the single source of truth for the Frictionless visual system. They normalize colors, typography, spacing, radii, elevation, and motion across platforms so that product and marketing surfaces remain consistent and predictable.

---

## 1. Token Architecture

### Naming Convention
- **Category.Object.Property** (e.g., `color.brand.primary`)
- Use **semantic aliases** for component usage (e.g., `button.primary.background`)

### Token Layers
1. **Core tokens** (raw values)
2. **Semantic tokens** (meaningful usage)
3. **Component aliases** (specific UI elements)

---

## 2. Color Tokens (Core)

```typescript
export const colors = {
  background: {
    primary: '#121212',
    secondary: '#1E1E1E',
    elevated: '#2D2D2D',
  },
  text: {
    primary: '#F9FAFB',
    secondary: '#9CA3AF',
    tertiary: '#6B7280',
    inverse: '#121212',
  },
  brand: {
    primary: '#4F46E5',
    accent: '#10B981',
  },
  semantic: {
    success: '#10B981',
    warning: '#F59E0B',
    error: '#EF4444',
    info: '#3B82F6',
  },
  map: {
    userLocation: '#3B82F6',
    dealMarker: '#F59E0B',
    flashDeal: '#EF4444',
  },
  heatmap: {
    cool: 'rgba(59, 130, 246, 0.35)',
    warm: 'rgba(245, 158, 11, 0.6)',
    hot: 'rgba(239, 68, 68, 0.9)',
  },
} as const;
```

---

## 3. Typography Tokens

```typescript
export const typography = {
  displayLarge: { fontSize: 48, fontWeight: '800', lineHeight: 56 },
  displayMedium: { fontSize: 32, fontWeight: '700', lineHeight: 40 },
  h1: { fontSize: 28, fontWeight: '700', lineHeight: 36 },
  h2: { fontSize: 24, fontWeight: '600', lineHeight: 32 },
  h3: { fontSize: 20, fontWeight: '600', lineHeight: 28 },
  bodyLarge: { fontSize: 18, fontWeight: '400', lineHeight: 28 },
  bodyMedium: { fontSize: 16, fontWeight: '400', lineHeight: 24 },
  bodySmall: { fontSize: 14, fontWeight: '400', lineHeight: 20 },
  labelLarge: { fontSize: 16, fontWeight: '600', lineHeight: 24 },
  labelMedium: { fontSize: 14, fontWeight: '500', lineHeight: 20 },
  labelSmall: { fontSize: 12, fontWeight: '500', lineHeight: 16 },
} as const;
```

---

## 4. Spacing Tokens

```typescript
export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
  xxxl: 64,
} as const;
```

---

## 5. Radius & Elevation

```typescript
export const radius = {
  sm: 6,
  md: 10,
  lg: 16,
  xl: 24,
  pill: 999,
} as const;

export const elevation = {
  sm: { shadowOpacity: 0.12, shadowRadius: 6, elevation: 2 },
  md: { shadowOpacity: 0.2, shadowRadius: 10, elevation: 4 },
  lg: { shadowOpacity: 0.28, shadowRadius: 16, elevation: 8 },
} as const;
```

---

## 6. Motion Tokens

```typescript
export const motion = {
  duration: {
    micro: 180,
    short: 220,
    medium: 320,
    long: 600,
  },
  easing: {
    standard: 'ease-in-out',
    enter: 'ease-out',
    exit: 'ease-in',
  },
} as const;
```

---

## 7. Semantic Alias Examples

| Alias Token | Maps To | Use |
| --- | --- | --- |
| `button.primary.background` | `color.brand.primary` | Primary CTA |
| `text.success` | `color.semantic.success` | Confirmation text |
| `surface.card` | `color.background.elevated` | Cards & sheets |

---

## 8. Governance

- Tokens are defined in a shared package and synced across apps.
- Any change requires Design + Engineering approval.
- Product teams must use tokens directly; no hard-coded color values.

---

## Related Documents

**Dependencies**
- BRAND-04: Section 2
- BRAND-05: Section 2

**Related Specs**
- DES-01: Section 5
- DES-08: Section 2
- DES-09: Section 3
- DES-10: Section 2

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial token architecture |
