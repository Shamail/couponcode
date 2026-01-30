---
document_id: TECH-03
version: 1.2
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-01
  - TECH-02
related_documents:
  - TECH-05
  - DES-01
  - DATA-01
  - THREAD-01
---

# TECH-03: Heatmap Generation

## Executive Summary

This document defines the heatmap generation pipeline that powers the Seller app's real-time activity visualization. It describes the API flow, spatial aggregation logic, and GeoJSON output format.

The system is optimized for low latency and privacy-safe aggregation.

## Backend Spatial Aggregation Pipeline

**Version:** 1.0
**Date:** January 2026
**Classification:** Technical Specification

---

## Overview

This document details the API endpoint and PostGIS queries for generating heatmap data that powers the Seller App's real-time foot traffic visualization. The system aggregates anonymized user locations into density cells, returning GeoJSON for direct rendering in Mapbox.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        HEATMAP GENERATION PIPELINE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐                                                          │
│   │  Seller App  │                                                          │
│   │   (Expo)     │                                                          │
│   └──────┬───────┘                                                          │
│          │                                                                   │
│          │  GET /locations/heatmap?lat=33.57&lng=-7.59&radius=500          │
│          │  (Polled every 30 seconds)                                       │
│          │                                                                   │
│          ▼                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                        API GATEWAY                                    │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│          │                                                                   │
│          ▼                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                    LAMBDA: location/heatmap                           │  │
│   │  ┌────────────────────────────────────────────────────────────────┐  │  │
│   │  │  1. Authenticate JWT (must be seller)                          │  │  │
│   │  │  2. Validate query params                                      │  │  │
│   │  │  3. Execute PostGIS spatial aggregation query                  │  │  │
│   │  │  4. Format response as GeoJSON                                 │  │  │
│   │  │  5. Return 200 OK with heatmap data                           │  │  │
│   │  └────────────────────────────────────────────────────────────────┘  │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│          │                                                                   │
│          │  PostGIS Query                                                   │
│          ▼                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                          NEON DB                                      │  │
│   │                                                                       │  │
│   │    SELECT                                                            │  │
│   │      ROUND(ST_Y(geom), 4) as lat,                                   │  │
│   │      ROUND(ST_X(geom), 4) as lng,                                   │  │
│   │      COUNT(DISTINCT user_id) as weight                              │  │
│   │    FROM user_locations                                               │  │
│   │    WHERE expires_at > NOW()                                          │  │
│   │      AND ST_DWithin(geom::geography, center, radius)                │  │
│   │    GROUP BY lat, lng                                                 │  │
│   │                                                                       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│          │                                                                   │
│          ▼                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                        RESPONSE (GeoJSON)                             │  │
│   │                                                                       │  │
│   │    {                                                                  │  │
│   │      "cells": [                                                      │  │
│   │        { "lat": 33.5731, "lng": -7.5898, "weight": 15 },            │  │
│   │        { "lat": 33.5740, "lng": -7.5905, "weight": 8 }              │  │
│   │      ],                                                              │  │
│   │      "totalUsers": 42,                                               │  │
│   │      "activityLevel": "high",                                        │  │
│   │      "generatedAt": "2026-01-30T14:32:00Z"                          │  │
│   │    }                                                                  │  │
│   │                                                                       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│          │                                                                   │
│          ▼                                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                    MAPBOX HEATMAP LAYER                               │  │
│   │                                                                       │  │
│   │    Seller App renders cells as HeatmapLayer                          │  │
│   │    Weight determines intensity/color                                 │  │
│   │                                                                       │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## API Endpoint

### GET /locations/heatmap

Returns aggregated user density data for heatmap rendering.

#### Request

```http
GET /locations/heatmap?lat=33.5731&lng=-7.5898&radius=500
Authorization: Bearer <jwt>
```

#### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `lat` | number | Yes | - | Center latitude |
| `lng` | number | Yes | - | Center longitude |
| `radius` | number | No | 500 | Radius in meters (max 1000) |
| `cellSize` | number | No | 4 | Decimal precision for cells |

#### Response

**Success (200 OK):**

```json
{
  "cells": [
    { "lat": 33.5731, "lng": -7.5898, "weight": 15 },
    { "lat": 33.5740, "lng": -7.5905, "weight": 8 },
    { "lat": 33.5725, "lng": -7.5912, "weight": 23 },
    { "lat": 33.5738, "lng": -7.5888, "weight": 5 }
  ],
  "totalUsers": 51,
  "activityLevel": "high",
  "generatedAt": "2026-01-30T14:32:00Z",
  "bounds": {
    "north": 33.578,
    "south": 33.568,
    "east": -7.584,
    "west": -7.595
  }
}
```

#### Response Schema

```typescript
interface HeatmapResponse {
  cells: HeatmapCell[];
  totalUsers: number;
  activityLevel: "low" | "medium" | "high";
  generatedAt: string;
  bounds: {
    north: number;
    south: number;
    east: number;
    west: number;
  };
}

interface HeatmapCell {
  lat: number;   // Cell center latitude
  lng: number;   // Cell center longitude
  weight: number; // User count in cell
}
```

#### Activity Level Thresholds

| Level | User Count | Color |
|-------|------------|-------|
| Low | < 10 users | Red |
| Medium | 10-30 users | Yellow |
| High | > 30 users | Green |

---

## Lambda Implementation

### Handler Code

```typescript
// packages/functions/src/locations/heatmap.ts

import { APIGatewayProxyHandlerV2 } from "aws-lambda";
import { neon } from "@neondatabase/serverless";
import { z } from "zod";
import { verifyToken } from "@frictionless/core/auth";

const sql = neon(process.env.DATABASE_URL!);

// Input validation
const HeatmapQuerySchema = z.object({
  lat: z.coerce.number().min(-90).max(90),
  lng: z.coerce.number().min(-180).max(180),
  radius: z.coerce.number().min(100).max(1000).default(500),
  cellSize: z.coerce.number().min(3).max(5).default(4),
});

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  try {
    // 1. Authenticate
    const authHeader = event.headers.authorization;
    if (!authHeader?.startsWith("Bearer ")) {
      return {
        statusCode: 401,
        body: JSON.stringify({ error: "UNAUTHORIZED" }),
      };
    }

    const token = authHeader.slice(7);
    const payload = await verifyToken(token);

    // Only sellers can view heatmaps
    if (payload.role !== "seller") {
      return {
        statusCode: 403,
        body: JSON.stringify({ error: "FORBIDDEN", message: "Sellers only" }),
      };
    }

    // 2. Validate query params
    const parseResult = HeatmapQuerySchema.safeParse(event.queryStringParameters);

    if (!parseResult.success) {
      return {
        statusCode: 400,
        body: JSON.stringify({
          error: "VALIDATION_ERROR",
          details: parseResult.error.errors,
        }),
      };
    }

    const { lat, lng, radius, cellSize } = parseResult.data;

    // 3. Execute PostGIS aggregation query
    const cells = await sql`
      WITH active_locations AS (
        -- Get only non-expired, unique user locations
        SELECT DISTINCT ON (user_id)
          user_id,
          geom
        FROM user_locations
        WHERE expires_at > NOW()
          AND ST_DWithin(
            geom::geography,
            ST_SetSRID(ST_MakePoint(${lng}, ${lat}), 4326)::geography,
            ${radius}
          )
        ORDER BY user_id, recorded_at DESC
      )
      SELECT
        ROUND(ST_Y(geom)::numeric, ${cellSize}) as lat,
        ROUND(ST_X(geom)::numeric, ${cellSize}) as lng,
        COUNT(*)::int as weight
      FROM active_locations
      GROUP BY
        ROUND(ST_Y(geom)::numeric, ${cellSize}),
        ROUND(ST_X(geom)::numeric, ${cellSize})
      ORDER BY weight DESC
    `;

    // 4. Calculate totals and activity level
    const totalUsers = cells.reduce((sum, cell) => sum + cell.weight, 0);
    const activityLevel = getActivityLevel(totalUsers);

    // 5. Calculate bounds
    const bounds = calculateBounds(lat, lng, radius);

    // 6. Return response
    return {
      statusCode: 200,
      headers: {
        "Content-Type": "application/json",
        "Cache-Control": "max-age=25", // Cache for 25 seconds
      },
      body: JSON.stringify({
        cells: cells.map((c) => ({
          lat: parseFloat(c.lat),
          lng: parseFloat(c.lng),
          weight: c.weight,
        })),
        totalUsers,
        activityLevel,
        generatedAt: new Date().toISOString(),
        bounds,
      }),
    };
  } catch (error) {
    console.error("Heatmap generation error:", error);

    return {
      statusCode: 500,
      body: JSON.stringify({ error: "INTERNAL_ERROR" }),
    };
  }
};

function getActivityLevel(count: number): "low" | "medium" | "high" {
  if (count < 10) return "low";
  if (count < 30) return "medium";
  return "high";
}

function calculateBounds(lat: number, lng: number, radius: number) {
  // Approximate degrees per meter at this latitude
  const latDelta = radius / 111320;
  const lngDelta = radius / (111320 * Math.cos(lat * (Math.PI / 180)));

  return {
    north: lat + latDelta,
    south: lat - latDelta,
    east: lng + lngDelta,
    west: lng - lngDelta,
  };
}
```

---

## PostGIS Query Deep Dive

### Core Aggregation Query

```sql
-- Main heatmap query with explanation
WITH active_locations AS (
  -- CTE 1: Get the most recent location per user within radius
  SELECT DISTINCT ON (user_id)
    user_id,
    geom
  FROM user_locations
  WHERE
    -- Only non-expired locations (TTL check)
    expires_at > NOW()

    -- Spatial filter: within radius of center point
    AND ST_DWithin(
      geom::geography,                                    -- User location as geography
      ST_SetSRID(ST_MakePoint(-7.5898, 33.5731), 4326)::geography,  -- Center point
      500                                                  -- Radius in meters
    )

  -- Keep only the most recent location per user
  ORDER BY user_id, recorded_at DESC
)
SELECT
  -- Cell buckets: round coordinates to create aggregation buckets
  ROUND(ST_Y(geom)::numeric, 4) as lat,  -- 4 decimals = ~11m cells
  ROUND(ST_X(geom)::numeric, 4) as lng,

  -- Count unique users in each cell
  COUNT(*)::int as weight

FROM active_locations
GROUP BY
  ROUND(ST_Y(geom)::numeric, 4),
  ROUND(ST_X(geom)::numeric, 4)
ORDER BY weight DESC;
```

### Query Breakdown

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         QUERY EXECUTION FLOW                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 1: SPATIAL FILTER                                                    │
│   ───────────────────────                                                   │
│   ST_DWithin(geom::geography, center, 500)                                 │
│                                                                              │
│   • Uses SP-GiST index on geom column                                      │
│   • ::geography cast enables meter-based distance                          │
│   • Returns all points within 500m of center                               │
│                                                                              │
│   Performance: O(log n) with spatial index                                 │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 2: TTL FILTER                                                        │
│   ──────────────────                                                        │
│   WHERE expires_at > NOW()                                                 │
│                                                                              │
│   • Uses B-tree index on expires_at                                        │
│   • Removes stale location data                                            │
│   • Combined with spatial filter using bitmap AND                          │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 3: DEDUPLICATION                                                     │
│   ─────────────────────                                                     │
│   DISTINCT ON (user_id) ... ORDER BY recorded_at DESC                      │
│                                                                              │
│   • Keeps only the most recent location per user                           │
│   • Prevents counting the same user multiple times                         │
│   • Uses index on (user_id, recorded_at DESC)                             │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Step 4: CELL AGGREGATION                                                  │
│   ────────────────────────                                                  │
│   ROUND(coordinate, 4) as cell_bucket                                        │
│                                                                              │
│   • Snaps coordinates to ~11m cells                                   │
│   • GROUP BY creates density buckets                                       │
│   • COUNT(*) gives user density per cell                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Cell Size Options

```
┌─────────────────────────────────────────────────────────────────┐
│                    CELL SIZE OPTIONS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Decimal Places    Cell Size    Use Case                        │
│  ───────────────────────────────────────────────────────────    │
│       3            ~111m        Large area overview             │
│       4            ~11m         Standard heatmap ✓              │
│       5            ~1.1m        Building-level detail           │
│                                                                  │
│  Recommendation: 4 decimal places                               │
│  • Fine enough to show clusters                                 │
│  • Coarse enough to aggregate meaningfully                      │
│  • ~11m cells match typical store frontage                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Alternative: Hexagonal Tiling (H3)

For more visually appealing heatmaps, consider using Uber's H3 hexagonal tiling system:

### H3 Implementation

```sql
-- Requires h3-pg extension
CREATE EXTENSION IF NOT EXISTS h3;

-- H3 hexagonal aggregation
WITH active_locations AS (
  SELECT DISTINCT ON (user_id) geom
  FROM user_locations
  WHERE expires_at > NOW()
    AND ST_DWithin(geom::geography, center, radius)
  ORDER BY user_id, recorded_at DESC
)
SELECT
  h3_lat_lng_to_cell(geom, 10) as h3_index,  -- Resolution 10 ≈ 15m hexagons
  COUNT(*) as weight,
  ST_AsGeoJSON(h3_cell_to_boundary(h3_lat_lng_to_cell(geom, 10))) as geojson
FROM active_locations
GROUP BY h3_lat_lng_to_cell(geom, 10);
```

### H3 Benefits

| Aspect | Square Cells | H3 Hexagons |
|--------|-------------|-------------|
| Visual appearance | Pixelated | Smooth, natural |
| Edge effects | 4 neighbors | 6 neighbors |
| Distance consistency | Varies | Uniform |
| Implementation | Simple | Requires extension |

---

## Alternative: DBSCAN Clustering

For dynamic cluster detection instead of fixed cell:

```sql
-- PostGIS DBSCAN clustering
SELECT
  cluster_id,
  ST_Centroid(ST_Collect(geom)) as center,
  COUNT(*) as weight,
  ST_ConvexHull(ST_Collect(geom)) as boundary
FROM (
  SELECT
    geom,
    ST_ClusterDBSCAN(geom, eps := 0.0005, minpoints := 3) OVER() as cluster_id
  FROM user_locations
  WHERE expires_at > NOW()
    AND ST_DWithin(geom::geography, center, radius)
) clustered
WHERE cluster_id IS NOT NULL
GROUP BY cluster_id;
```

### DBSCAN Parameters

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `eps` | 0.0005 | ~55m clustering distance |
| `minpoints` | 3 | Minimum users to form cluster |

---

## Performance Optimization

### Index Utilization

```sql
-- Verify query uses spatial index
EXPLAIN ANALYZE
SELECT ...
FROM user_locations
WHERE ST_DWithin(geom::geography, center, 500);

-- Expected output should show:
-- "Index Scan using idx_user_locations_geom on user_locations"
```

### Query Performance Targets

| Metric | Target | Method |
|--------|--------|--------|
| Query time (500m, 100 users) | < 50ms | SP-GiST index |
| Query time (500m, 1000 users) | < 200ms | SP-GiST index |
| Response size | < 10KB | Cell aggregation |

### Caching Strategy

```typescript
// Response header for client-side caching
return {
  statusCode: 200,
  headers: {
    "Cache-Control": "max-age=25, stale-while-revalidate=5",
    "ETag": generateETag(cells),
  },
  body: JSON.stringify(response),
};

// Client can use If-None-Match for conditional requests
```

---

## GeoJSON Output Format

For direct Mapbox rendering, return cells as GeoJSON FeatureCollection:

```typescript
// Alternative response format for Mapbox
interface GeoJSONResponse {
  type: "FeatureCollection";
  features: Array<{
    type: "Feature";
    geometry: {
      type: "Point";
      coordinates: [number, number]; // [lng, lat]
    };
    properties: {
      weight: number;
    };
  }>;
  metadata: {
    totalUsers: number;
    activityLevel: string;
    generatedAt: string;
  };
}

// Example response
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [-7.5898, 33.5731]
      },
      "properties": {
        "weight": 15
      }
    }
  ],
  "metadata": {
    "totalUsers": 51,
    "activityLevel": "high",
    "generatedAt": "2026-01-30T14:32:00Z"
  }
}
```

---

## Client-Side Rendering

### Mapbox Heatmap Layer Configuration

```typescript
// packages/apps/seller/src/components/HeatmapLayer.tsx

import MapboxGL from "@rnmapbox/maps";

const HEATMAP_STYLE = {
  // Radius increases with zoom level
  heatmapRadius: [
    "interpolate",
    ["linear"],
    ["zoom"],
    10, 15,   // At zoom 10: 15px radius
    15, 30,   // At zoom 15: 30px radius
  ],

  // Weight from data
  heatmapWeight: ["get", "weight"],

  // Intensity increases at higher zoom
  heatmapIntensity: [
    "interpolate",
    ["linear"],
    ["zoom"],
    10, 0.5,
    15, 1,
  ],

  // Color gradient from cool to hot
  heatmapColor: [
    "interpolate",
    ["linear"],
    ["heatmap-density"],
    0, "rgba(33, 33, 33, 0)",        // Transparent
    0.2, "rgba(74, 0, 128, 0.5)",    // Deep purple
    0.4, "rgba(157, 78, 221, 0.7)",  // Purple
    0.6, "rgba(255, 0, 110, 0.8)",   // Magenta
    0.8, "rgba(255, 107, 0, 0.9)",   // Orange
    1, "rgba(255, 184, 0, 1)",       // Amber (hot)
  ],

  // Opacity
  heatmapOpacity: 0.8,
};

export function HeatmapLayer({ data }: { data: HeatmapResponse }) {
  const geojson = useMemo(() => ({
    type: "FeatureCollection",
    features: data.cells.map((cell) => ({
      type: "Feature",
      geometry: {
        type: "Point",
        coordinates: [cell.lng, cell.lat],
      },
      properties: {
        weight: cell.weight,
      },
    })),
  }), [data]);

  return (
    <MapboxGL.ShapeSource id="heatmap-source" shape={geojson}>
      <MapboxGL.HeatmapLayer
        id="heatmap-layer"
        style={HEATMAP_STYLE}
      />
    </MapboxGL.ShapeSource>
  );
}
```

---

## Free Tier: Activity Level Only

For free-tier sellers, return only aggregate activity level:

```typescript
// Simplified response for free tier
if (!isPro) {
  return {
    statusCode: 200,
    body: JSON.stringify({
      totalUsers,
      activityLevel,
      generatedAt: new Date().toISOString(),
      // No cells array for free tier
    }),
  };
}
```

---

## Monitoring

### Key Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `HeatmapQueryTime` | P99 query duration | > 500ms |
| `HeatmapCellCount` | Cells returned | > 1000 (may affect rendering) |
| `HeatmapUserCount` | Total users in response | Monitoring only |
| `HeatmapRequests` | Request volume | Capacity planning |

### Logging

```typescript
import { Logger } from "@aws-lambda-powertools/logger";

const logger = new Logger({ serviceName: "heatmap-service" });

logger.info("Heatmap generated", {
  sellerId: payload.userId,
  center: { lat, lng },
  radius,
  cellCount: cells.length,
  totalUsers,
  queryTimeMs: endTime - startTime,
});
```

---

## Summary

| Aspect | Implementation |
|--------|----------------|
| **Endpoint** | `GET /locations/heatmap` |
| **Auth** | JWT Bearer token (sellers only) |
| **Spatial query** | PostGIS ST_DWithin |
| **Aggregation** | Cell buckets (4 decimal = ~11m) |
| **Deduplication** | DISTINCT ON user_id |
| **Index** | SP-GiST on geom column |
| **Response format** | JSON or GeoJSON |
| **Cache** | 25 seconds client-side |
| **Free tier** | Activity level only (no cells) |

---

**Previous:** [TECH-02: Location Ingestion](./TECH-02-location-ingestion.md)
**Next:** [TECH-04: Redemption Security →](./TECH-04-redemption-security.md)

## Related Documents

**Dependencies**
- TECH-01: Section 3
- TECH-02: Section 2

**Related Specs**
- TECH-05: Section 2
- DES-01: Section 1
- DATA-01: Section 2
- THREAD-01: Section 3

**Implementation Guides**
- GUIDE-02: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Engineering Lead | Updated terminology for cell aggregation |
| 1.1 | 2026-01-30 | Engineering Lead | Added data dictionary and thread references |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
