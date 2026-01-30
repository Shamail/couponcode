---
document_id: TECH-02
version: 1.3
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-01
  - TECH-06
related_documents:
  - TECH-03
  - METRICS-03
  - DATA-01
  - THREAD-01
---

# TECH-02: Location Ingestion

## Executive Summary

This document describes how the Buyer app sends GPS check-ins to the backend and how the system stores and expires location data. The pipeline balances privacy, freshness, and performance for heatmap generation.

Location ingestion is a core dependency for both buyer discovery and seller heatmap visualization.

## GPS Data Pipeline Architecture

**Version:** 1.0
**Date:** January 2026
**Classification:** Technical Architecture Document

---

## Overview

This document describes how the Frictionless platform ingests, processes, and stores user location data from the Buyer app. The system is designed for:

- **Privacy:** Location fuzzing and anonymization
- **Efficiency:** Optimized PostGIS writes
- **Freshness:** 30-minute TTL for "live" map data

---

## API Endpoint

### POST /location

Receives GPS coordinates from the Buyer app.

#### Request

```http
POST /api/v1/location
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "latitude": 33.573141,
  "longitude": -7.619219,
  "accuracy": 15.5,
  "timestamp": "2026-01-30T15:32:00Z"
}
```

#### Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "next_update_in": 30
}
```

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                  LOCATION INGESTION FLOW                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐                                           │
│   │  Buyer App  │                                           │
│   │  (Expo)     │                                           │
│   └──────┬──────┘                                           │
│          │ GPS coords (lat, lng, accuracy, timestamp)       │
│          ▼                                                  │
│   ┌─────────────┐                                           │
│   │  API GW +   │                                           │
│   │  Lambda     │                                           │
│   └──────┬──────┘                                           │
│          │                                                  │
│          ├──────────────────────────────────────┐           │
│          │                                      │           │
│          ▼                                      ▼           │
│   ┌─────────────┐                        ┌─────────────┐    │
│   │  Validate   │                        │  Rate       │    │
│   │  & Sanitize │                        │  Limiter    │    │
│   └──────┬──────┘                        └─────────────┘    │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                           │
│   │  Privacy    │  Truncate to 3 decimal places             │
│   │  Fuzzing    │  (~111m precision)                        │
│   └──────┬──────┘                                           │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                           │
│   │  PostGIS    │  UPSERT with ST_MakePoint                 │
│   │  Write      │                                           │
│   └──────┬──────┘                                           │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐                                           │
│   │  Neon DB    │  user_locations table                     │
│   │  (PostGIS)  │                                           │
│   └─────────────┘                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Implementation

### Lambda Handler (SST)

```typescript
// packages/functions/src/location/ingest.ts

import { APIGatewayProxyHandlerV2 } from 'aws-lambda';
import { db } from '@frictionless/core/db';
import { sql } from 'drizzle-orm';
import { userLocations } from '@frictionless/core/schema';
import { verifyJWT } from '@frictionless/core/auth';

interface LocationPayload {
  latitude: number;
  longitude: number;
  accuracy?: number;
  timestamp?: string;
}

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  // 1. Authenticate
  const token = event.headers.authorization?.replace('Bearer ', '');
  if (!token) {
    return { statusCode: 401, body: JSON.stringify({ error: 'Unauthorized' }) };
  }

  const user = await verifyJWT(token);
  if (!user) {
    return { statusCode: 401, body: JSON.stringify({ error: 'Invalid token' }) };
  }

  // 2. Parse and validate payload
  const body: LocationPayload = JSON.parse(event.body || '{}');
  
  if (!isValidCoordinate(body.latitude, body.longitude)) {
    return { 
      statusCode: 400, 
      body: JSON.stringify({ error: 'Invalid coordinates' }) 
    };
  }

  // 3. Apply privacy fuzzing (truncate to 3 decimal places)
  const fuzzedLat = truncateCoordinate(body.latitude, 3);
  const fuzzedLng = truncateCoordinate(body.longitude, 3);

  // 4. Upsert into PostGIS
  await upsertLocation(user.id, fuzzedLat, fuzzedLng);

  return {
    statusCode: 200,
    body: JSON.stringify({
      success: true,
      next_update_in: 30 // Tell client to send next update in 30s
    })
  };
};
```

### Coordinate Validation

```typescript
// packages/core/src/location/validation.ts

export function isValidCoordinate(lat: number, lng: number): boolean {
  // Basic range check
  if (lat < -90 || lat > 90) return false;
  if (lng < -180 || lng > 180) return false;

  // Morocco bounding box (rough)
  const MOROCCO_BOUNDS = {
    minLat: 27.0,
    maxLat: 36.0,
    minLng: -13.5,
    maxLng: -1.0
  };

  // Warn but don't reject if outside Morocco (user might be traveling)
  const inMorocco = 
    lat >= MOROCCO_BOUNDS.minLat &&
    lat <= MOROCCO_BOUNDS.maxLat &&
    lng >= MOROCCO_BOUNDS.minLng &&
    lng <= MOROCCO_BOUNDS.maxLng;

  if (!inMorocco) {
    console.warn(`Location outside Morocco: ${lat}, ${lng}`);
  }

  return true;
}

export function truncateCoordinate(value: number, decimals: number): number {
  const factor = Math.pow(10, decimals);
  return Math.trunc(value * factor) / factor;
}
```

### Privacy Fuzzing

```typescript
// packages/core/src/location/privacy.ts

/**
 * Privacy fuzzing strategy:
 * 
 * Decimal places → Precision
 * 1 → ~11.1 km
 * 2 → ~1.1 km
 * 3 → ~111 m  ← We use this
 * 4 → ~11 m
 * 5 → ~1.1 m
 * 
 * 3 decimal places provides ~111m precision, which:
 * - Is precise enough for heatmap generation
 * - Prevents exact user tracking
 * - Groups nearby users naturally
 */

export function fuzzLocation(lat: number, lng: number): { lat: number; lng: number } {
  return {
    lat: truncateCoordinate(lat, 3),
    lng: truncateCoordinate(lng, 3)
  };
}

// Example:
// Input:  33.573141, -7.619219
// Output: 33.573, -7.619
```

### PostGIS Upsert

```typescript
// packages/core/src/location/repository.ts

import { db } from '../db';
import { sql } from 'drizzle-orm';

export async function upsertLocation(
  userId: string,
  latitude: number,
  longitude: number
): Promise<void> {
  // Use raw SQL for PostGIS functions
  await db.execute(sql`
    INSERT INTO user_locations (user_id, geom, updated_at)
    VALUES (
      ${userId},
      ST_SetSRID(ST_MakePoint(${longitude}, ${latitude}), 4326),
      NOW()
    )
    ON CONFLICT (user_id)
    DO UPDATE SET
      geom = ST_SetSRID(ST_MakePoint(${longitude}, ${latitude}), 4326),
      updated_at = NOW()
  `);
}
```

---

## Database Schema

### user_locations Table

```sql
-- Enable PostGIS extension (run once)
CREATE EXTENSION IF NOT EXISTS postgis;

-- Create table
CREATE TABLE user_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  geom GEOMETRY(Point, 4326) NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  CONSTRAINT user_locations_user_id_unique UNIQUE (user_id)
);

-- Create spatial index (SP-GiST for fast queries)
CREATE INDEX user_locations_geom_idx 
ON user_locations 
USING SPGIST (geom);

-- Create index for TTL cleanup
CREATE INDEX user_locations_updated_at_idx 
ON user_locations (updated_at);

-- Add comment
COMMENT ON TABLE user_locations IS 
'Live user locations with 30-minute TTL. Coordinates fuzzed to 3 decimal places.';
```

### Drizzle Schema

```typescript
// packages/core/src/schema/userLocations.ts

import { pgTable, uuid, timestamp, index } from 'drizzle-orm/pg-core';
import { users } from './users';

// Note: geometry type handled via raw SQL
export const userLocations = pgTable('user_locations', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' })
    .unique(),
  // geom column managed via PostGIS, not Drizzle
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
}, (table) => ({
  updatedAtIdx: index('user_locations_updated_at_idx').on(table.updatedAt),
}));
```

---

## TTL Cleanup

Locations older than 30 minutes are deleted to keep the map "live."

### Cron Job (SST)

```typescript
// stacks/CronStack.ts

import { Cron } from 'sst/constructs';

export function CronStack({ stack }) {
  // Run every 5 minutes
  new Cron(stack, 'LocationCleanup', {
    schedule: 'rate(5 minutes)',
    job: 'packages/functions/src/cron/cleanup-locations.handler',
  });
}
```

### Cleanup Handler

```typescript
// packages/functions/src/cron/cleanup-locations.ts

import { db } from '@frictionless/core/db';
import { sql } from 'drizzle-orm';

export const handler = async () => {
  const TTL_MINUTES = 30;

  const result = await db.execute(sql`
    DELETE FROM user_locations
    WHERE updated_at < NOW() - INTERVAL '${sql.raw(TTL_MINUTES.toString())} minutes'
    RETURNING id
  `);

  console.log(`Cleaned up ${result.rowCount} stale location records`);

  return {
    deleted: result.rowCount,
    timestamp: new Date().toISOString()
  };
};
```

---

## Client-Side Implementation

### Buyer App Location Service

```typescript
// apps/buyer/src/services/location.ts

import * as Location from 'expo-location';
import { api } from './api';

const UPDATE_INTERVAL = 30000; // 30 seconds
const SIGNIFICANT_DISTANCE = 50; // meters

let lastLocation: Location.LocationObject | null = null;
let locationSubscription: Location.LocationSubscription | null = null;

export async function startLocationTracking(): Promise<void> {
  // Request permissions
  const { status } = await Location.requestForegroundPermissionsAsync();
  if (status !== 'granted') {
    throw new Error('Location permission denied');
  }

  // Start watching location
  locationSubscription = await Location.watchPositionAsync(
    {
      accuracy: Location.Accuracy.Balanced,
      timeInterval: UPDATE_INTERVAL,
      distanceInterval: SIGNIFICANT_DISTANCE,
    },
    handleLocationUpdate
  );
}

export function stopLocationTracking(): void {
  locationSubscription?.remove();
  locationSubscription = null;
}

async function handleLocationUpdate(location: Location.LocationObject): Promise<void> {
  // Skip if location hasn't changed significantly
  if (lastLocation && !hasMovedSignificantly(lastLocation, location)) {
    return;
  }

  lastLocation = location;

  try {
    await api.post('/location', {
      latitude: location.coords.latitude,
      longitude: location.coords.longitude,
      accuracy: location.coords.accuracy,
      timestamp: new Date(location.timestamp).toISOString(),
    });
  } catch (error) {
    console.error('Failed to send location:', error);
    // Don't throw - location updates are best-effort
  }
}

function hasMovedSignificantly(
  prev: Location.LocationObject,
  curr: Location.LocationObject
): boolean {
  const distance = calculateDistance(
    prev.coords.latitude,
    prev.coords.longitude,
    curr.coords.latitude,
    curr.coords.longitude
  );
  return distance >= SIGNIFICANT_DISTANCE;
}

function calculateDistance(
  lat1: number, lon1: number,
  lat2: number, lon2: number
): number {
  // Haversine formula
  const R = 6371e3; // Earth radius in meters
  const φ1 = (lat1 * Math.PI) / 180;
  const φ2 = (lat2 * Math.PI) / 180;
  const Δφ = ((lat2 - lat1) * Math.PI) / 180;
  const Δλ = ((lon2 - lon1) * Math.PI) / 180;

  const a =
    Math.sin(Δφ / 2) * Math.sin(Δφ / 2) +
    Math.cos(φ1) * Math.cos(φ2) * Math.sin(Δλ / 2) * Math.sin(Δλ / 2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

  return R * c;
}
```

### Battery Optimization

```typescript
// apps/buyer/src/services/location.ts

import * as Battery from 'expo-battery';

// Adjust tracking based on battery level
async function getOptimalTrackingConfig(): Promise<Location.LocationOptions> {
  const batteryLevel = await Battery.getBatteryLevelAsync();
  
  if (batteryLevel < 0.15) {
    // Low battery: reduce accuracy and frequency
    return {
      accuracy: Location.Accuracy.Low,
      timeInterval: 60000, // 60 seconds
      distanceInterval: 100, // 100 meters
    };
  }
  
  if (batteryLevel < 0.30) {
    // Medium battery: balanced mode
    return {
      accuracy: Location.Accuracy.Balanced,
      timeInterval: 45000,
      distanceInterval: 75,
    };
  }
  
  // Normal battery: full accuracy
  return {
    accuracy: Location.Accuracy.Balanced,
    timeInterval: 30000,
    distanceInterval: 50,
  };
}
```

---

## Rate Limiting

### Per-User Limits

```typescript
// packages/functions/src/middleware/rateLimit.ts

import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

const LOCATION_RATE_LIMIT = {
  windowMs: 60000, // 1 minute
  maxRequests: 5,   // 5 updates per minute max
};

export async function checkLocationRateLimit(userId: string): Promise<boolean> {
  const key = `rate:location:${userId}`;
  const count = await redis.incr(key);
  
  if (count === 1) {
    await redis.expire(key, LOCATION_RATE_LIMIT.windowMs / 1000);
  }
  
  return count <= LOCATION_RATE_LIMIT.maxRequests;
}
```

---

## Monitoring

### CloudWatch Metrics

```typescript
// packages/functions/src/location/metrics.ts

import { CloudWatch } from '@aws-sdk/client-cloudwatch';

const cloudwatch = new CloudWatch({});

export async function recordLocationMetric(
  metricName: string,
  value: number
): Promise<void> {
  await cloudwatch.putMetricData({
    Namespace: 'Frictionless/Location',
    MetricData: [
      {
        MetricName: metricName,
        Value: value,
        Unit: 'Count',
        Timestamp: new Date(),
      },
    ],
  });
}

// Usage in handler:
// await recordLocationMetric('LocationUpdatesReceived', 1);
// await recordLocationMetric('LocationUpdatesOutsideMorocco', 1);
```

### Key Metrics to Track

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `LocationUpdatesReceived` | Total updates/min | Spike detection |
| `LocationUpdateLatency` | P95 processing time | > 500ms |
| `LocationValidationFailures` | Invalid coords | > 5% |
| `StaleRecordsCleaned` | TTL cleanup count | N/A (informational) |
| `UniqueUsersTracked` | Active users | N/A (business metric) |

---

## Security Considerations

1. **Authentication:** All location updates require valid JWT
2. **Privacy:** Coordinates truncated before storage
3. **Rate limiting:** Prevents GPS spoofing attacks
4. **TTL:** Old data automatically purged
5. **No PII:** Location table contains only user_id, no personal data

---

## Background Location Strategy

### Overview

The Buyer app uses a hybrid approach to enable deal discovery even when the app is backgrounded, while maintaining battery efficiency and user privacy:

1. **Geofencing (Primary)** - OS-managed monitoring of deal zones
2. **Smart Push Notifications (Secondary)** - Server-side targeting based on last known location

### Geofencing

Geofencing enables zero-battery-drain background awareness by delegating location monitoring to the OS.

#### How It Works

1. When buyer opens the app, backend returns list of nearby stores with active deals
2. App registers geofences (up to 20 on iOS, 100 on Android) around those stores
3. OS handles monitoring using cell towers/WiFi (no GPS, no battery drain)
4. When user enters a geofence, OS wakes the app briefly → local notification is triggered

#### Implementation

```typescript
// apps/buyer/src/services/geofencing.ts

import * as Location from 'expo-location';
import * as TaskManager from 'expo-task-manager';
import * as Notifications from 'expo-notifications';

const GEOFENCE_TASK = 'deal-geofence-task';

// Register background task (runs even when app killed)
TaskManager.defineTask(GEOFENCE_TASK, ({ data, error }) => {
  if (error) {
    console.error('Geofence task error:', error);
    return;
  }

  const { eventType, region } = data as {
    eventType: Location.GeofencingEventType;
    region: Location.LocationRegion;
  };

  if (eventType === Location.GeofencingEventType.Enter) {
    // Show local notification
    showDealNearbyNotification(region.identifier);
  }
});

async function showDealNearbyNotification(storeId: string): Promise<void> {
  // Fetch deal info from cache or storage
  const deal = await getDealByStoreId(storeId);
  if (!deal) return;

  await Notifications.scheduleNotificationAsync({
    content: {
      title: `Deal nearby at ${deal.storeName}!`,
      body: `${deal.discount}% off - tap to view`,
      data: { dealId: deal.id, storeId },
    },
    trigger: null, // Immediate
  });
}

// Set up geofences for nearby deals
export async function registerDealGeofences(deals: Deal[]): Promise<void> {
  // Stop existing geofences first
  await Location.stopGeofencingAsync(GEOFENCE_TASK).catch(() => {});

  // Limit to 20 for iOS compatibility
  const regions = deals.slice(0, 20).map(deal => ({
    identifier: deal.storeId,
    latitude: deal.lat,
    longitude: deal.lng,
    radius: 150, // meters - trigger when within 150m
    notifyOnEnter: true,
    notifyOnExit: false,
  }));

  if (regions.length === 0) return;

  await Location.startGeofencingAsync(GEOFENCE_TASK, regions);
}

// Call when app opens or location changes significantly
export async function refreshGeofences(): Promise<void> {
  const { status } = await Location.getForegroundPermissionsAsync();
  if (status !== 'granted') return;

  const location = await Location.getCurrentPositionAsync({
    accuracy: Location.Accuracy.Balanced,
  });

  // Fetch nearby deals with active offers
  const response = await api.get('/deals/nearby-for-geofencing', {
    lat: location.coords.latitude,
    lng: location.coords.longitude,
    radius: 2000, // 2km radius
  });

  await registerDealGeofences(response.deals);
}
```

#### Geofencing Characteristics

| Aspect | Value | Notes |
|--------|-------|-------|
| Battery impact | < 0.5%/day | OS-managed, uses cell/WiFi |
| Max geofences (iOS) | 20 | System limit |
| Max geofences (Android) | 100 | System limit |
| Trigger radius | 150m | Sufficient for deal discovery |
| Permission required | When In Use | No "Always" permission needed |
| Works when app killed | Yes | TaskManager handles wake |

---

## Background Notification (Local)

When a geofence is triggered, the app wakes briefly to display a local notification.

### Notification Flow

```
┌─────────────────────────────────────────────────────────────┐
│                 GEOFENCE NOTIFICATION FLOW                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   User approaches store                                      │
│   (150m radius)                                              │
│         │                                                    │
│         ▼                                                    │
│   OS detects geofence entry                                  │
│   (via cell tower / WiFi)                                    │
│         │                                                    │
│         ▼                                                    │
│   TaskManager wakes app                                      │
│   (background task)                                          │
│         │                                                    │
│         ▼                                                    │
│   Fetch deal details from cache                              │
│         │                                                    │
│         ▼                                                    │
│   Schedule local notification                                │
│         │                                                    │
│         ▼                                                    │
│   User sees: "Deal nearby at Café Central!"                  │
│         │                                                    │
│         ▼                                                    │
│   User taps notification                                     │
│         │                                                    │
│         ▼                                                    │
│   App opens to deal detail screen                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Notification Content

```typescript
interface GeofenceNotification {
  title: string;     // "Deal nearby at {storeName}!"
  body: string;      // "{discount}% off - tap to view"
  data: {
    dealId: string;
    storeId: string;
    type: 'geofence_entry';
  };
}
```

---

## Smart Push Notifications (Server-Side)

Even with foreground-only tracking, the server has "last known location" data that can be used for intelligent push targeting.

### TTL Extension for Push Targeting

Location data TTL is extended to **2 hours** for push notification targeting (data is still deleted after 30 minutes for heatmap purposes).

```typescript
// packages/functions/src/deals/broadcast.ts

// When seller creates a flash deal, notify nearby users
async function broadcastFlashDeal(deal: Deal): Promise<void> {
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
    AND u.push_enabled = true
  `, [deal.lng, deal.lat, deal.radius]);

  // Send contextual push
  for (const user of nearbyUsers) {
    const isStale = Date.now() - new Date(user.updated_at).getTime() > 30 * 60 * 1000;

    await sendPushNotification(user.push_token, {
      title: isStale
        ? `Deal near where you were!`
        : `Deal nearby now!`,
      body: `${deal.discount}% off at ${deal.storeName}`,
      data: { dealId: deal.id, type: 'flash_deal' },
    });
  }
}
```

### Push Notification Scenarios

| Scenario | Trigger | Message Template |
|----------|---------|------------------|
| Flash deal nearby (live) | User location < 30 min old | "Deal nearby now! {discount}% off at {store}" |
| Flash deal nearby (recent) | User location 30-120 min old | "Deal near where you were! {discount}% off at {store}" |
| Daily digest | Cron job (12pm, 6pm) | "X deals near your usual spots today" |
| Expiring claimed deal | 15 min before expiry | "Your deal at {store} expires in 15 min!" |

---

## Performance Characteristics

| Operation | Expected Latency | Notes |
|-----------|------------------|-------|
| API Gateway → Lambda | ~10ms | Cold start: ~200ms |
| JWT Verification | ~5ms | Cached |
| PostGIS Upsert | ~10ms | SP-GiST index |
| Total E2E | < 50ms | P95 target |

---

*"Every check-in builds the map. Every map builds the moat."*

— Frictionless Engineering Team

## Related Documents

**Dependencies**
- TECH-01: Section 3
- TECH-06: Section 2

**Related Specs**
- TECH-03: Section 3
- TECH-05: Section 5 (Background Geofencing)
- METRICS-03: Section 3
- DATA-01: Section 2
- THREAD-01: Section 3
- ADR-002: Background Location Strategy

**Implementation Guides**
- IMPL-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.3 | 2026-01-31 | Engineering Lead | Added geofencing, background notifications, and smart push targeting |
| 1.2 | 2026-01-30 | Engineering Lead | Updated check-in terminology |
| 1.1 | 2026-01-30 | Engineering Lead | Added data dictionary and thread references |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
