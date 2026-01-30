# Background vs Foreground Location: Solution Architecture

## Decisions Made
- **Permission level:** "When In Use" only (no "Always" permission)
- **Heatmap strategy:** Show recent + live data (2-hour lookback with visual distinction)
- **Primary background tool:** Geofencing (works with "When In Use" permission)

---

## The Core Problem

**Current State:** Foreground-only tracking (30s intervals when app is open)
- Buyers (customers) miss deals when app is closed
- Sellers (merchants) see an incomplete heatmap (only active app users)
- Background tracking would drain battery and hurt user trust

**Goal:** Enable the marketplace to function even when apps are backgrounded, while maintaining battery efficiency and user privacy.

---

## Recommended Solution: Hybrid Geofencing + Smart Push

A multi-layered approach that uses different techniques based on context:

### Layer 1: Geofencing (Zero Battery Drain)

**How it works:**
- When buyer opens app, backend returns list of nearby stores with active deals
- App registers geofences (up to 20 on iOS, 100 on Android) around those stores
- OS handles monitoring using cell towers/WiFi (no GPS, no battery drain)
- When user enters geofence, OS wakes app briefly → local notification

**Implementation:**
```typescript
// expo-location geofencing
import * as Location from 'expo-location';
import * as TaskManager from 'expo-task-manager';

const GEOFENCE_TASK = 'deal-geofence-task';

// Register background task (runs even when app killed)
TaskManager.defineTask(GEOFENCE_TASK, ({ data, error }) => {
  if (error) return;
  const { eventType, region } = data;

  if (eventType === Location.GeofencingEventType.Enter) {
    // Show local notification: "Deal nearby at {store}!"
    showLocalNotification(region.identifier);
  }
});

// Set up geofences for nearby deals
async function registerDealGeofences(deals: Deal[]) {
  const regions = deals.slice(0, 20).map(deal => ({
    identifier: deal.storeId,
    latitude: deal.lat,
    longitude: deal.lng,
    radius: 150, // meters - trigger when within 150m
    notifyOnEnter: true,
    notifyOnExit: false,
  }));

  await Location.startGeofencingAsync(GEOFENCE_TASK, regions);
}
```

**Pros:**
- Near-zero battery impact (OS-managed, uses cell/WiFi)
- Works even when app is killed
- Precise enough for deal discovery (100-150m radius)

**Cons:**
- Limited to 20 geofences on iOS
- Doesn't contribute to seller heatmap (no location sent to server)

---

### Layer 2: Smart Push Notifications (Server-Side)

**The insight:** Even with foreground-only tracking, you have "last known location" data. Use it smartly.

**Scenario A: Deal Created Near Recent Users**
```typescript
// When seller creates a flash deal:
async function broadcastFlashDeal(deal: Deal) {
  // Find users who were nearby in the last 2 hours
  const nearbyUsers = await db.query(`
    SELECT DISTINCT u.push_token, ul.updated_at
    FROM users u
    JOIN user_locations ul ON u.id = ul.user_id
    WHERE ST_DWithin(
      ul.geom::geography,
      ST_MakePoint($1, $2)::geography,
      $3
    )
    AND ul.updated_at > NOW() - INTERVAL '2 hours'
  `, [deal.lng, deal.lat, deal.radius]);

  // Send contextual push
  for (const user of nearbyUsers) {
    const isStale = Date.now() - user.updated_at > 30 * 60 * 1000;

    await sendPush(user.push_token, {
      title: isStale
        ? `Deal near where you were!`
        : `Deal nearby now!`,
      body: `${deal.discount}% off at ${deal.storeName}`,
      data: { dealId: deal.id },
    });
  }
}
```

**Scenario B: Time-Based Re-engagement**
```typescript
// Cron job: Daily at peak shopping hours (12pm, 6pm)
async function sendDailyDealDigest() {
  // For each user, find deals near their most common locations
  const users = await db.query(`
    SELECT u.id, u.push_token,
      MODE() WITHIN GROUP (ORDER BY
        ROUND(ST_Y(ul.geom)::numeric, 2),
        ROUND(ST_X(ul.geom)::numeric, 2)
      ) as common_location
    FROM users u
    JOIN user_locations ul ON u.id = ul.user_id
    WHERE ul.updated_at > NOW() - INTERVAL '7 days'
    GROUP BY u.id
  `);

  // Match with active deals and send personalized digest
}
```

---

### Layer 3: Seller Heatmap Improvements

**Problem:** Heatmap only shows users with app open (incomplete picture)

**Solutions:**

**A. Extend TTL for Heatmap Display (Not Storage)**
```typescript
// Current: 30-min TTL for live locations
// New: Show "recent activity" layer with 2-hour old data

GET /v1/heatmap?lat=33.57&lng=-7.59&radius=500

// Response now includes:
{
  "live": {
    "cells": [...], // Last 30 mins (high confidence)
    "count": 42
  },
  "recent": {
    "cells": [...], // Last 2 hours (lower weight)
    "count": 128
  }
}

// UI: Live = solid color, Recent = semi-transparent
```

**B. Predictive Heatmap (ML-Enhanced)**
```typescript
// Use historical patterns to predict current foot traffic
// "At this location, at this time, on this day of week,
//  we typically see X people based on past data"

// Requires: S3 historical data + Athena queries
// Display: "Expected traffic: Medium-High (based on patterns)"
```

---

## Implementation Phases

### Phase 1: Geofencing (Immediate Win)
- Add geofencing to Buyer app
- Register geofences on app open/close
- Local notifications when entering deal zones
- **No backend changes required**
- **Permission: "When In Use" only** (user's choice)

### Phase 2: Smart Push Notifications
- Extend location TTL to 2 hours (for push targeting only)
- Add "last known location" push logic to flash deal flow
- Add daily deal digest based on user patterns

### Phase 3: Enhanced Seller Heatmap ✓
- **User chose: Show recent + live data**
- Add "recent activity" layer (2-hour lookback)
- Live data = solid color, Recent = semi-transparent (50% opacity)
- UI clearly labels: "Live users" vs "Recent activity"

### ~~Phase 4: Significant Location Changes~~ (Skipped)
- User chose "When In Use" permission only
- Geofencing provides sufficient background coverage

---

## Permission Strategy

**Buyer App Permission Flow (Simplified):**

1. **On First Launch**
   - Request "When In Use" permission only
   - Explain: "Frictionless uses your location to show nearby deals"
   - Full functionality including geofencing (iOS allows geofencing with "When In Use")

2. **No Upgrade Prompts**
   - Stay with "When In Use" for all users
   - Better App Store reviews, less user friction
   - Geofencing + smart push provides sufficient background coverage

3. **Settings Toggle**
   - "Deal alerts when nearby" on/off (controls geofencing)
   - Clear explanation of what each setting does

---

## Battery Impact Summary

| Feature | Battery Impact | Permission Needed | Status |
|---------|---------------|-------------------|--------|
| Foreground tracking | ~5%/hour active | When In Use | ✓ Current |
| Geofencing (20 zones) | <0.5%/day | When In Use | ✓ Adding |
| Smart Push (server-side) | 0% | N/A | ✓ Adding |

**Result:** Users get background deal notifications with negligible battery impact.

---

## Documentation Files to Update

### Update Existing Docs

1. **TECH-02-location-ingestion.md**
   - Add "Geofencing" section documenting the geofence registration flow
   - Add "Background Notification" section for local notification triggers
   - Update "Client-Side Implementation" with geofencing service code
   - Add section on TTL extension for push targeting (2 hours)

2. **TECH-05-mobile-stack.md**
   - Add `useGeofencing` hook documentation
   - Add TaskManager background task setup
   - Update app.config.ts example with geofencing configuration
   - Add new section "5. Background Geofencing"

3. **TECH-03-heatmap-generation.md**
   - Update heatmap API response to include `live` and `recent` layers
   - Document the 2-hour lookback for "recent" activity
   - Add visual distinction guidance (opacity for recent vs live)

4. **PRD-01-buyer-app.md**
   - Update feature list to include "Background deal alerts"
   - Add user story: "As a buyer, I get notified when I'm near a deal even if the app is closed"
   - Update permission flow to reflect geofencing behavior

5. **TECH-06-api-contract.md** (if exists)
   - Update `/v1/heatmap` response schema
   - Document push notification targeting logic

### Create New Docs

1. **ADR-002-background-location-strategy.md**
   - Document the decision to use geofencing over continuous background tracking
   - Capture the tradeoffs: battery vs data coverage
   - Record the "When In Use" permission decision
   - Document the smart push notification approach

---

## Verification Plan

1. **Documentation Review**
   - Cross-reference updates between TECH-02, TECH-03, TECH-05
   - Ensure API contract changes in TECH-06 match heatmap response schema
   - Verify ADR-002 captures all decision rationale

2. **Technical Accuracy**
   - Confirm geofencing code samples work with expo-location API
   - Validate heatmap response schema changes are backward-compatible
   - Ensure smart push SQL queries are valid PostGIS

3. **Consistency Check**
   - Update DOCUMENT-TRACKER.md with new ADR-002
   - Add cross-references from THREAD-01 to new background location flow
