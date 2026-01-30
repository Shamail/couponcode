---
document_id: ADR-002
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-31
owner: Engineering Lead
dependencies:
  - TECH-02
  - TECH-05
related_documents:
  - PRD-01
  - TECH-03
  - TECH-06
---

# ADR-002: Background Location Strategy

## Architecture Decision Record

**Date:** January 2026
**Status:** Accepted
**Deciders:** Engineering Lead, Product Lead, Privacy Officer

---

## Context

The Frictionless Buyer app needs to notify users of nearby deals even when the app is backgrounded or closed. This creates a tension between:

1. **User value:** Buyers want to be notified of deals without constantly checking the app
2. **Seller value:** Merchants want accurate heatmaps showing foot traffic patterns
3. **Battery efficiency:** Users will uninstall apps that drain battery
4. **Privacy:** Continuous background tracking raises privacy concerns and hurts App Store ratings

### Problem Statement

Current state: Foreground-only tracking (30s intervals when app is open)

- Buyers miss deals when app is closed
- Sellers see an incomplete heatmap (only active app users)
- Continuous background tracking would drain battery and hurt user trust

### Options Considered

#### Option A: Continuous Background Location ("Always" Permission)

**How it works:** Request "Always" location permission, track user continuously in background.

| Pros | Cons |
|------|------|
| Complete location coverage | 5-15% battery drain per day |
| Accurate heatmap data | "Always" permission scares users |
| Real-time server-side awareness | Privacy concerns, negative reviews |
| | iOS App Store scrutiny |
| | GDPR/CNDP compliance complexity |

#### Option B: Significant Location Changes ("Always" Permission)

**How it works:** Use iOS significant location changes API, which reports location every ~500m movement.

| Pros | Cons |
|------|------|
| Lower battery than continuous | Still requires "Always" permission |
| ~500m accuracy | Irregular update intervals |
| Works when app killed | Not available on all Android devices |
| | Still triggers privacy concerns |

#### Option C: Geofencing + Smart Push ("When In Use" Permission) ✓

**How it works:** Register geofences around deal locations, use server-side push targeting based on last known location.

| Pros | Cons |
|------|------|
| Near-zero battery impact | Limited to 20 geofences (iOS) |
| Works with "When In Use" permission | Doesn't contribute to heatmap |
| OS-managed (reliable) | Requires deal data for registration |
| No privacy concerns | |
| Works even when app killed | |

---

## Decision

**We chose Option C: Geofencing + Smart Push with "When In Use" permission only.**

### Rationale

1. **Battery efficiency is non-negotiable.** Users in Morocco often have budget phones with smaller batteries. Any noticeable drain will lead to uninstalls.

2. **Permission friction hurts adoption.** "Always" location permission prompts have significantly lower acceptance rates. For a new app, this is a growth killer.

3. **Geofencing provides sufficient coverage.** For deal discovery, we don't need continuous tracking—we just need to know when users are near a deal. Geofencing does exactly this.

4. **Smart push fills the gap.** For heatmap completeness, we extend location TTL to 2 hours and use server-side push targeting. Users who were recently nearby can still receive flash deal notifications.

5. **Privacy-first approach.** Morocco's CNDP regulations and general privacy awareness mean we should minimize data collection. "When In Use" is defensible; "Always" requires justification.

---

## Implementation

### Layer 1: Geofencing (Client-Side)

```typescript
// Register geofences around nearby deals
const regions = deals.slice(0, 20).map(deal => ({
  identifier: deal.storeId,
  latitude: deal.lat,
  longitude: deal.lng,
  radius: 150, // meters
  notifyOnEnter: true,
  notifyOnExit: false,
}));

await Location.startGeofencingAsync(GEOFENCE_TASK, regions);
```

**Characteristics:**
- Maximum 20 geofences (iOS limit)
- 150m trigger radius
- Refresh on app open and significant movement (500m)
- Local notification on entry

### Layer 2: Smart Push (Server-Side)

```typescript
// Target users based on extended location TTL
const nearbyUsers = await db.query(`
  SELECT DISTINCT u.push_token, ul.updated_at
  FROM users u
  JOIN user_locations ul ON u.id = ul.user_id
  WHERE ST_DWithin(ul.geom::geography, deal_point, deal_radius)
    AND ul.updated_at > NOW() - INTERVAL '2 hours'
`);

// Contextual messaging based on location freshness
const message = isLive ? "Deal nearby now!" : "Deal near where you were!";
```

**Characteristics:**
- 2-hour location TTL for push targeting
- Contextual messaging (live vs recent)
- Server-side decision, no client battery impact

### Layer 3: Enhanced Heatmap

Since geofencing doesn't contribute location data to the server, we enhance the heatmap with a "recent activity" layer:

- **Live layer (0-30 min):** Full opacity, high confidence
- **Recent layer (30 min - 2 hours):** 50% opacity, historical context

This gives sellers a more complete picture of foot traffic without requiring continuous background tracking.

---

## Consequences

### Positive

1. **Near-zero battery impact** (< 0.5%/day for geofencing)
2. **Higher permission acceptance** (no "Always" prompt)
3. **App Store approval** (no background location justification needed)
4. **Privacy compliance** (minimal data collection)
5. **User trust** (clear value exchange for location permission)

### Negative

1. **Limited geofence count** (20 on iOS, 100 on Android)
   - Mitigation: Prioritize highest-value deals, refresh on movement
2. **Incomplete heatmap data** (only foreground users)
   - Mitigation: Recent activity layer provides historical context
3. **No continuous server-side awareness**
   - Mitigation: Smart push targeting with 2-hour window

### Neutral

1. **Code complexity:** Geofencing + TaskManager setup is more complex than simple background tracking, but well-documented in expo-location.

---

## Alternatives Not Chosen

### Beacon/BLE Approach

Using Bluetooth beacons in stores for precise indoor positioning.

**Why not:** Requires hardware deployment in stores, not scalable for MVP.

### Predictive Models

Using ML to predict where users are based on historical patterns.

**Why not:** Requires significant historical data, complex to implement, and doesn't solve the real-time notification problem.

### Periodic Background Fetch

Using iOS background fetch to periodically wake the app.

**Why not:** Unreliable timing (iOS decides when to wake), doesn't provide location.

---

## Metrics to Monitor

| Metric | Target | Purpose |
|--------|--------|---------|
| Geofence notification CTR | > 15% | Measure notification relevance |
| Permission acceptance rate | > 80% | Confirm "When In Use" is acceptable |
| Battery complaints (reviews) | < 1% | Ensure no battery impact perception |
| Heatmap data completeness | > 70% coverage | Track foreground usage patterns |
| Push notification opt-in | > 60% | Measure smart push reach |

---

## Review Schedule

This decision should be revisited if:

1. iOS/Android significantly change geofencing limits or capabilities
2. User research indicates strong demand for more proactive notifications
3. Battery technology improves to make background tracking acceptable
4. Regulatory changes require different location handling

---

## Related Documents

**Dependencies**
- TECH-02: Location Ingestion (Background Location Strategy section)
- TECH-05: Mobile Stack (Background Geofencing section)

**Related Specs**
- PRD-01: Buyer App (Background deal alerts feature)
- TECH-03: Heatmap Generation (Live/Recent layers)
- TECH-06: API Contract (Geofencing endpoint, push targeting)

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-31 | Engineering Lead | Initial ADR documenting geofencing strategy |
