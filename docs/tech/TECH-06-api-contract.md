---
document_id: TECH-06
version: 1.2
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-00
related_documents:
  - PRD-01
  - PRD-02
  - TECH-04
  - METRICS-03
  - DATA-01
  - THREAD-01
  - THREAD-02
---

# TECH-06: API Contract Specification

## Executive Summary

This document defines the HTTP API surface for Frictionless, including authentication, response formats, and endpoint schemas. It is the canonical reference for client and server implementations.

All API changes must be reflected here and in analytics instrumentation.

## Overview

**Framework:** Hono (lightweight, edge-optimized)
**Runtime:** AWS Lambda via SST v3
**Base URL:** `https://api.frictionless.ma/api/v1`
**Content-Type:** `application/json`
**Authentication:** Bearer JWT (where required)

---

## Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

JWT payload structure:
```typescript
interface JWTPayload {
  sub: string;        // User ID (UUID)
  role: 'buyer' | 'seller';
  iat: number;        // Issued at
  exp: number;        // Expiration (24h for buyers, 7d for sellers)
}
```

---

## Common Response Formats

### Success Response
```typescript
interface SuccessResponse<T> {
  success: true;
  data: T;
  meta?: {
    timestamp: string;  // ISO 8601
    requestId: string;  // For debugging/tracing
  };
}
```

### Error Response
```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;      // Machine-readable error code
    message: string;   // Human-readable message
    details?: object;  // Additional context (validation errors, etc.)
  };
  meta: {
    timestamp: string;
    requestId: string;
  };
}
```

### Error Codes
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Missing or invalid token |
| `FORBIDDEN` | 403 | Valid token but insufficient permissions |
| `NOT_FOUND` | 404 | Resource does not exist |
| `VALIDATION_ERROR` | 400 | Request body/params failed validation |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Unexpected server error |
| `EXPIRED_TOKEN` | 401 | QR/redemption token has expired |
| `ALREADY_REDEEMED` | 409 | Deal already redeemed by this user |

---

## Endpoints

### 1. POST `/api/v1/check-in`

**Purpose:** Buyer app sends location updates to track foot traffic.

**Authentication:** Required (Buyer)

**Rate Limit:** 60 requests/minute per user

#### Request

```typescript
interface CheckInRequest {
  lat: number;           // Latitude (-90 to 90)
  lng: number;           // Longitude (-180 to 180)
  accuracy: number;      // GPS accuracy in meters
  timestamp: string;     // ISO 8601, client timestamp
  altitude?: number;     // Altitude in meters (optional)
  speed?: number;        // Speed in m/s (optional)
  heading?: number;      // Heading in degrees (optional)
}
```

**Example Request:**
```bash
curl -X POST https://api.frictionless.ma/api/v1/check-in \
  -H "Authorization: Bearer eyJhbG..." \
  -H "Content-Type: application/json" \
  -d '{
    "lat": 33.5731,
    "lng": -7.5898,
    "accuracy": 10.5,
    "timestamp": "2024-01-15T14:30:00.000Z"
  }'
```

#### Response

**Success (200 OK):**
```json
{
  "success": true,
  "data": {
    "recorded": true,
    "nearbyDeals": 3
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_abc123"
  }
}
```

**Validation Error (400):**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": {
      "lat": "Must be between -90 and 90",
      "accuracy": "Must be a positive number"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_abc123"
  }
}
```

#### Implementation Notes

- Locations are stored with PostGIS `GEOGRAPHY(POINT, 4326)` type
- Stale check-ins (>5 minutes old based on client timestamp) are flagged but still recorded
- Accuracy threshold: check-ins with accuracy >100m are recorded but excluded from heatmap calculations

---

### 2. GET `/api/v1/sellers/heatmap`

**Purpose:** Returns GeoJSON FeatureCollection of anonymized buyer density for seller's heatmap display.

**Authentication:** Required (Seller)

**Rate Limit:** 30 requests/minute per user (aligned with 30s polling interval)

#### Request Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `lat` | number | Yes | - | Center latitude |
| `lng` | number | Yes | - | Center longitude |
| `radius` | number | No | 500 | Search radius in meters (max: 2000) |
| `resolution` | number | No | 50 | Cell size in meters (min: 25, max: 200) |
| `window` | number | No | 15 | Time window in minutes (max: 60) |

**Example Request:**
```bash
curl -X GET "https://api.frictionless.ma/api/v1/sellers/heatmap?lat=33.5731&lng=-7.5898&radius=500" \
  -H "Authorization: Bearer eyJhbG..."
```

#### Response

**Success (200 OK):**
```typescript
interface HeatmapResponse {
  success: true;
  data: {
    type: "FeatureCollection";
    features: HeatmapFeature[];
    metadata: {
      center: [number, number];  // [lng, lat]
      radius: number;
      resolution: number;
      window: number;
      generatedAt: string;       // ISO 8601
      totalCheckIns: number;     // Total check-ins in area
      uniqueUsers: number;       // Anonymized count
    };
  };
  meta: {
    timestamp: string;
    requestId: string;
  };
}

interface HeatmapFeature {
  type: "Feature";
  geometry: {
    type: "Point";
    coordinates: [number, number];  // [lng, lat] - grid cell center
  };
  properties: {
    weight: number;      // Normalized density (0-1)
    count: number;       // Raw check-in count in cell
    intensity: "low" | "medium" | "high";  // For easy styling
  };
}
```

**Example Response:**
```json
{
  "success": true,
  "data": {
    "type": "FeatureCollection",
    "features": [
      {
        "type": "Feature",
        "geometry": {
          "type": "Point",
          "coordinates": [-7.5898, 33.5731]
        },
        "properties": {
          "weight": 0.85,
          "count": 42,
          "intensity": "high"
        }
      },
      {
        "type": "Feature",
        "geometry": {
          "type": "Point",
          "coordinates": [-7.5902, 33.5735]
        },
        "properties": {
          "weight": 0.45,
          "count": 18,
          "intensity": "medium"
        }
      }
    ],
    "metadata": {
      "center": [-7.5898, 33.5731],
      "radius": 500,
      "resolution": 50,
      "window": 15,
      "generatedAt": "2024-01-15T14:30:00.123Z",
      "totalCheckIns": 1250,
      "uniqueUsers": 89
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_def456"
  }
}
```

#### Implementation Notes

- Uses PostGIS snapping-to-cell for spatial aggregation
- Privacy: minimum 3 unique users per cell to display (k-anonymity)
- Intensity thresholds: low (<0.33), medium (0.33-0.66), high (>0.66)
- Response is cached for 10 seconds at edge (CloudFront)

---

### 3. POST `/api/v1/redeem`

**Purpose:** Validates QR token during "SafeColor Verification" redemption process.

**Authentication:** Required (Buyer)

**Rate Limit:** 10 requests/minute per user

#### Request

```typescript
interface RedeemRequest {
  token: string;           // QR payload (JWT or signed token)
  colorCode: string;       // 6-char hex color displayed on seller screen
  dealId: string;          // UUID of the deal being redeemed
  location: {
    lat: number;
    lng: number;
    accuracy: number;
  };
}
```

**Example Request:**
```bash
curl -X POST https://api.frictionless.ma/api/v1/redeem \
  -H "Authorization: Bearer eyJhbG..." \
  -H "Content-Type: application/json" \
  -d '{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "colorCode": "FF6B35",
    "dealId": "550e8400-e29b-41d4-a716-446655440000",
    "location": {
      "lat": 33.5731,
      "lng": -7.5898,
      "accuracy": 5.2
    }
  }'
```

#### QR Token Structure

The QR code encodes a short-lived JWT:
```typescript
interface QRTokenPayload {
  sid: string;      // Seller ID
  did: string;      // Deal ID
  col: string;      // Color code (must match screen)
  exp: number;      // Expiration (30 seconds from generation)
  nonce: string;    // Single-use nonce
}
```

#### Response

**Success (200 OK):**
```json
{
  "success": true,
  "data": {
    "redemptionId": "red_xyz789",
    "deal": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "20% off Cappuccino",
      "merchant": "Café Central",
      "discount": {
        "type": "percentage",
        "value": 20
      },
      "expiresAt": "2024-01-15T23:59:59.000Z"
    },
    "redeemedAt": "2024-01-15T14:30:00.123Z",
    "verificationCode": "FRXN-7842"
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_ghi789"
  }
}
```

**Error - Expired Token (401):**
```json
{
  "success": false,
  "error": {
    "code": "EXPIRED_TOKEN",
    "message": "QR code has expired. Please scan again.",
    "details": {
      "expiredAt": "2024-01-15T14:29:30.000Z"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_ghi789"
  }
}
```

**Error - Color Mismatch (400):**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Color code does not match. Ensure you're scanning the current QR.",
    "details": {
      "field": "colorCode"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_ghi789"
  }
}
```

**Error - Already Redeemed (409):**
```json
{
  "success": false,
  "error": {
    "code": "ALREADY_REDEEMED",
    "message": "You have already redeemed this deal.",
    "details": {
      "redeemedAt": "2024-01-15T10:15:00.000Z"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_ghi789"
  }
}
```

#### Implementation Notes

- QR tokens are single-use; nonce is stored in Redis with 60s TTL
- Location verification: buyer must be within 100m of seller location
- Color code rotates every 10 seconds on seller screen
- Redemption creates record in `redemptions` table with ACID guarantees

---

### 4. POST `/api/v1/deals`

**Purpose:** Seller creates a new deal/offer.

**Authentication:** Required (Seller)

**Rate Limit:** 100 requests/hour per user

#### Request

```typescript
interface CreateDealRequest {
  title: string;              // Max 100 chars
  description: string;        // Max 500 chars
  discount: {
    type: "percentage" | "fixed" | "bogo";
    value: number;            // Percentage (1-100) or fixed amount in centimes
    maxDiscount?: number;     // Cap for percentage discounts (in centimes)
  };
  terms?: string;             // Max 1000 chars, optional T&C
  validity: {
    startsAt: string;         // ISO 8601
    expiresAt: string;        // ISO 8601
  };
  limits: {
    totalRedemptions?: number;   // Max total redemptions (null = unlimited)
    perUser?: number;            // Max per user (default: 1)
    dailyCap?: number;           // Max redemptions per day (null = unlimited)
  };
  location: {
    lat: number;
    lng: number;
    radius: number;           // Valid redemption radius in meters (max: 500)
  };
  categories: string[];       // Category IDs (max 3)
  images?: string[];          // S3 URLs (max 5)
  active?: boolean;           // Default: true
}
```

**Example Request:**
```bash
curl -X POST https://api.frictionless.ma/api/v1/deals \
  -H "Authorization: Bearer eyJhbG..." \
  -H "Content-Type: application/json" \
  -d '{
    "title": "20% off Any Coffee",
    "description": "Start your day right with 20% off any coffee drink.",
    "discount": {
      "type": "percentage",
      "value": 20,
      "maxDiscount": 3000
    },
    "validity": {
      "startsAt": "2024-01-15T00:00:00.000Z",
      "expiresAt": "2024-01-31T23:59:59.000Z"
    },
    "limits": {
      "totalRedemptions": 500,
      "perUser": 1,
      "dailyCap": 50
    },
    "location": {
      "lat": 33.5731,
      "lng": -7.5898,
      "radius": 100
    },
    "categories": ["cat_food_beverage", "cat_coffee"]
  }'
```

#### Response

**Success (201 Created):**
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "title": "20% off Any Coffee",
    "description": "Start your day right with 20% off any coffee drink.",
    "discount": {
      "type": "percentage",
      "value": 20,
      "maxDiscount": 3000
    },
    "validity": {
      "startsAt": "2024-01-15T00:00:00.000Z",
      "expiresAt": "2024-01-31T23:59:59.000Z"
    },
    "limits": {
      "totalRedemptions": 500,
      "perUser": 1,
      "dailyCap": 50
    },
    "location": {
      "lat": 33.5731,
      "lng": -7.5898,
      "radius": 100
    },
    "categories": ["cat_food_beverage", "cat_coffee"],
    "stats": {
      "views": 0,
      "redemptions": 0,
      "remaining": 500
    },
    "active": true,
    "createdAt": "2024-01-15T14:30:00.123Z",
    "updatedAt": "2024-01-15T14:30:00.123Z"
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_jkl012"
  }
}
```

**Validation Error (400):**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request body",
    "details": {
      "discount.value": "Percentage must be between 1 and 100",
      "validity.expiresAt": "Expiration must be after start date",
      "categories": "Maximum 3 categories allowed"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T14:30:00.123Z",
    "requestId": "req_jkl012"
  }
}
```

#### Implementation Notes

- Deal location stored as PostGIS `GEOGRAPHY(POINT, 4326)`
- Images must be pre-uploaded to S3; endpoint validates URLs belong to our bucket
- Categories are validated against `categories` table
- Deal becomes queryable immediately after creation (no moderation queue in v1)

---

## Additional Endpoints (Future Reference)

| Endpoint | Method | Purpose | Priority |
|----------|--------|---------|----------|
| `/api/v1/auth/register` | POST | User registration | P0 |
| `/api/v1/auth/login` | POST | User authentication | P0 |
| `/api/v1/auth/refresh` | POST | Refresh JWT token | P0 |
| `/api/v1/deals` | GET | List deals (with filters) | P1 |
| `/api/v1/deals/:id` | GET | Get deal details | P1 |
| `/api/v1/deals/:id` | PATCH | Update deal | P1 |
| `/api/v1/deals/:id` | DELETE | Soft delete deal | P1 |
| `/api/v1/sellers/qr` | GET | Generate dynamic QR + color | P0 |
| `/api/v1/buyers/history` | GET | Redemption history | P2 |
| `/api/v1/sellers/analytics` | GET | Dashboard metrics | P2 |

---

## Hono Implementation Example

```typescript
// packages/api/src/routes/v1/index.ts
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { jwt } from 'hono/jwt';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

const app = new Hono();

// Middleware
app.use('/*', cors());
app.use('/api/v1/*', jwt({ secret: process.env.JWT_SECRET! }));

// Schemas
const checkInSchema = z.object({
  lat: z.number().min(-90).max(90),
  lng: z.number().min(-180).max(180),
  accuracy: z.number().positive(),
  timestamp: z.string().datetime(),
  altitude: z.number().optional(),
  speed: z.number().optional(),
  heading: z.number().optional(),
});

// Routes
app.post('/api/v1/check-in', zValidator('json', checkInSchema), async (c) => {
  const payload = c.req.valid('json');
  const userId = c.get('jwtPayload').sub;

  // Insert into PostGIS
  await db.execute(sql`
    INSERT INTO location_checkins (user_id, location, accuracy, client_timestamp)
    VALUES (
      ${userId},
      ST_SetSRID(ST_MakePoint(${payload.lng}, ${payload.lat}), 4326)::geography,
      ${payload.accuracy},
      ${payload.timestamp}
    )
  `);

  // Count nearby deals
  const nearbyDeals = await db.execute(sql`
    SELECT COUNT(*) FROM deals
    WHERE active = true
    AND ST_DWithin(
      location,
      ST_SetSRID(ST_MakePoint(${payload.lng}, ${payload.lat}), 4326)::geography,
      1000
    )
  `);

  return c.json({
    success: true,
    data: {
      recorded: true,
      nearbyDeals: Number(nearbyDeals[0].count),
    },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: c.get('requestId'),
    },
  });
});

export default app;
```

---

## Rate Limiting Strategy

| Endpoint | Limit | Window | Key |
|----------|-------|--------|-----|
| `POST /check-in` | 60 | 1 min | `user:{userId}` |
| `GET /heatmap` | 30 | 1 min | `user:{userId}` |
| `POST /redeem` | 10 | 1 min | `user:{userId}` |
| `POST /deals` | 100 | 1 hour | `user:{userId}` |

Implementation: Redis sliding window counter via Upstash.

---

## Security Considerations

1. **Input Validation:** All inputs validated with Zod schemas
2. **SQL Injection:** Parameterized queries via Drizzle ORM
3. **JWT Security:** Short-lived tokens (24h buyer, 7h seller), refresh tokens in httpOnly cookies
4. **Location Privacy:** Buyer locations anonymized in heatmap (k-anonymity, k=3)
5. **QR Replay Protection:** Single-use nonces with Redis TTL
6. **Rate Limiting:** Per-user limits to prevent abuse
7. **CORS:** Restricted to known origins in production

---

## Versioning

API follows URL versioning (`/api/v1/`). Breaking changes require new version. Non-breaking additions (new fields, endpoints) do not.

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2024-01-15 | Initial API specification |

## Related Documents

**Dependencies**
- TECH-00: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 3
- TECH-04: Section 2
- METRICS-03: Section 2
- DATA-01: Section 2
- THREAD-01: Section 3
- THREAD-02: Section 3

**Implementation Guides**
- IMPL-02: Section 3

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Engineering Lead | Updated terminology and heatmap params |
| 1.1 | 2026-01-30 | Engineering Lead | Added data dictionary and thread references |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
