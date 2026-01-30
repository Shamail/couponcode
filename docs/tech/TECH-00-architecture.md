---
document_id: TECH-00
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - STRAT-01
related_documents:
  - TECH-01
  - TECH-06
  - TECH-07
---

# TECH-00: System Architecture

## Executive Summary

Frictionless is built on a serverless-first architecture designed for rapid scaling and low operational overhead. The stack uses SST v3 on AWS, Neon Postgres with PostGIS, and lightweight Lambda-based APIs to power real-time location experiences.

This document defines the system blueprint, infrastructure decisions, and data access approach that enable low-latency heatmaps and secure redemption workflows.

## Overview

Frictionless is built on a serverless-first architecture optimized for cost efficiency, operational simplicity, and the "Zero Friction" philosophy. This document outlines the infrastructure blueprint using AWS services orchestrated via SST (Serverless Stack) v3.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRICTIONLESS ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐     ┌──────────────┐                                     │
│   │  Buyer App   │     │  Seller App  │                                     │
│   │ (React Native│     │ (React Native│                                     │
│   │    Expo)     │     │    Expo)     │                                     │
│   └──────┬───────┘     └──────┬───────┘                                     │
│          │                    │                                              │
│          │    HTTPS/REST      │                                              │
│          └────────┬───────────┘                                              │
│                   ▼                                                          │
│   ┌───────────────────────────────┐                                         │
│   │        AWS CloudFront         │                                         │
│   │    (CDN + API Distribution)   │                                         │
│   └───────────────┬───────────────┘                                         │
│                   │                                                          │
│                   ▼                                                          │
│   ┌───────────────────────────────┐                                         │
│   │       AWS API Gateway         │                                         │
│   │      (HTTP API / REST)        │                                         │
│   └───────────────┬───────────────┘                                         │
│                   │                                                          │
│                   ▼                                                          │
│   ┌───────────────────────────────┐                                         │
│   │       AWS Lambda              │                                         │
│   │   (TypeScript Functions)      │                                         │
│   │                               │                                         │
│   │  ┌─────────┐ ┌─────────────┐  │                                         │
│   │  │Location │ │   Deals     │  │                                         │
│   │  │  API    │ │   API       │  │                                         │
│   │  └─────────┘ └─────────────┘  │                                         │
│   │  ┌─────────┐ ┌─────────────┐  │                                         │
│   │  │Heatmap  │ │ Redemption  │  │                                         │
│   │  │  API    │ │   API       │  │                                         │
│   │  └─────────┘ └─────────────┘  │                                         │
│   └───────────────┬───────────────┘                                         │
│                   │                                                          │
│                   │  @neondatabase/serverless                               │
│                   │  (WebSocket / HTTP)                                     │
│                   ▼                                                          │
│   ┌───────────────────────────────┐                                         │
│   │         Neon DB               │                                         │
│   │   (Serverless PostgreSQL)     │                                         │
│   │      + PostGIS Extension      │                                         │
│   └───────────────────────────────┘                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Infrastructure Components

### 1. SST v3 Configuration

SST v3 provides infrastructure-as-code with TypeScript, enabling type-safe deployments and seamless local development.

```typescript
// sst.config.ts
import type { SSTConfig } from "sst";
import { API } from "./stacks/ApiStack";
import { Database } from "./stacks/DatabaseStack";

export default {
  config(_input) {
    return {
      name: "frictionless",
      region: "eu-west-1", // Closest AWS region to Morocco
    };
  },
  stacks(app) {
    app.stack(Database).stack(API);
  },
} satisfies SSTConfig;
```

### 2. AWS Lambda (Compute Layer)

Lambda functions handle all API requests with the following configuration:

| Setting | Value | Rationale |
|---------|-------|-----------|
| Runtime | Node.js 20.x | Latest LTS with TypeScript support |
| Memory | 512 MB | Balanced for spatial queries |
| Timeout | 10 seconds | Sufficient for DB operations |
| Architecture | arm64 | 20% cost reduction vs x86 |

```typescript
// stacks/ApiStack.ts
import { Api, Function } from "sst/constructs";

export function API({ stack }) {
  const api = new Api(stack, "Api", {
    defaults: {
      function: {
        runtime: "nodejs20.x",
        architecture: "arm64",
        timeout: "10 seconds",
        memorySize: 512,
        environment: {
          DATABASE_URL: process.env.DATABASE_URL!,
        },
      },
    },
    routes: {
      // Location APIs
      "POST /locations": "packages/functions/src/locations/update.handler",
      "GET /locations/heatmap": "packages/functions/src/locations/heatmap.handler",

      // Deals APIs
      "GET /deals": "packages/functions/src/deals/list.handler",
      "GET /deals/nearby": "packages/functions/src/deals/nearby.handler",
      "GET /deals/{id}": "packages/functions/src/deals/get.handler",

      // Redemption APIs
      "POST /redemptions/initiate": "packages/functions/src/redemptions/initiate.handler",
      "POST /redemptions/verify": "packages/functions/src/redemptions/verify.handler",
      "GET /redemptions/{id}": "packages/functions/src/redemptions/get.handler",
    },
  });

  stack.addOutputs({
    ApiEndpoint: api.url,
  });

  return { api };
}
```

### 3. CloudFront (CDN & Distribution)

CloudFront sits in front of API Gateway for:

- **Global Edge Caching**: Static assets cached at edge locations
- **DDoS Protection**: AWS Shield Standard included
- **SSL/TLS Termination**: Free ACM certificates
- **Geographic Routing**: Optimal latency for Moroccan users

```typescript
// CloudFront is automatically configured by SST's Api construct
// For custom domain:
const api = new Api(stack, "Api", {
  customDomain: {
    domainName: "api.frictionless.ma",
    hostedZone: "frictionless.ma",
  },
  // ... routes
});
```

---

## Database Connection Strategy

### The Lambda Cold Start Problem

Traditional PostgreSQL connections use TCP, which creates challenges in serverless:

1. **Connection Overhead**: TCP handshake adds ~50-100ms latency
2. **Connection Limits**: Neon has connection limits per compute
3. **Cold Starts**: Each Lambda instance needs its own connection

### Solution: Neon Serverless Driver

We use `@neondatabase/serverless` which provides two connection modes:

#### Option A: HTTP Mode (Recommended for Frictionless)

```typescript
// packages/core/src/db/client.ts
import { neon } from "@neondatabase/serverless";

// HTTP mode - stateless, no connection management needed
export const sql = neon(process.env.DATABASE_URL!);

// Usage in Lambda handler
export async function handler(event: APIGatewayProxyEvent) {
  const results = await sql`
    SELECT id, name FROM users WHERE id = ${userId}
  `;
  return { statusCode: 200, body: JSON.stringify(results) };
}
```

**Benefits of HTTP Mode:**
- Zero connection management
- No cold start penalty for connections
- Scales infinitely with Lambda concurrency
- Perfect for simple queries

#### Option B: WebSocket Mode (For Complex Queries)

```typescript
// packages/core/src/db/pooled-client.ts
import { Pool, neonConfig } from "@neondatabase/serverless";
import ws from "ws";

// Enable WebSocket for pooled connections
neonConfig.webSocketConstructor = ws;

// Create a pool for transaction support
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Usage for transactions
export async function redeemDeal(userId: string, dealId: string) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    // ... transaction logic
    await client.query("COMMIT");
  } catch (e) {
    await client.query("ROLLBACK");
    throw e;
  } finally {
    client.release();
  }
}
```

### Connection Pooling with Neon

Neon provides built-in connection pooling via PgBouncer:

```
# Direct connection (for migrations)
DATABASE_URL=postgresql://user:pass@ep-xxx.eu-west-1.aws.neon.tech/frictionless

# Pooled connection (for Lambda)
DATABASE_URL=postgresql://user:pass@ep-xxx-pooler.eu-west-1.aws.neon.tech/frictionless
```

**Pooler Configuration:**
- Mode: Transaction (connections returned after each transaction)
- Max Client Connections: 10,000
- Pool Size: Scales with Neon compute size

---

## Polling Strategy: The 30-Second Interval

### Why Polling Over WebSockets?

The Seller App displays a heatmap of nearby buyers. We chose polling over WebSockets/subscriptions for these reasons:

```
┌─────────────────────────────────────────────────────────────────┐
│                    POLLING vs WEBSOCKETS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WEBSOCKETS                        POLLING (30s)                │
│  ───────────                       ─────────────                │
│  ✗ Connection management           ✓ Stateless                  │
│  ✗ Reconnection logic              ✓ Simple HTTP calls          │
│  ✗ AWS API Gateway WS costs        ✓ Standard REST API          │
│  ✗ Complex infrastructure          ✓ Same Lambda functions      │
│  ✗ State synchronization           ✓ Fresh data each request    │
│                                                                  │
│  Real-time: ~100ms latency         Real-time: ~30s latency      │
│                                                                  │
│  Use when: Chat, gaming,           Use when: Dashboards,        │
│  collaborative editing             analytics, monitoring        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 30 Seconds: The Sweet Spot

| Interval | UX Impact | Cost Impact | Recommendation |
|----------|-----------|-------------|----------------|
| 5s | Unnecessary freshness | 6x API calls | Over-engineering |
| 15s | Slightly fresher | 2x API calls | Acceptable |
| **30s** | **"Real-time enough"** | **Baseline** | **Recommended** |
| 60s | Noticeably stale | 0.5x API calls | Frustrating UX |

**Justification:**

1. **Retail Context**: A shopper's decision to visit a store takes minutes, not seconds. A 30-second delay is imperceptible in this context.

2. **Battery Life**: Frequent polling drains mobile batteries. 30s is aggressive enough for freshness, conservative enough for battery.

3. **Cost Efficiency**: At 30s intervals:
   - 1,000 sellers × 2 polls/minute × 60 minutes × 12 hours = 1.44M requests/day
   - At $0.20/1M requests (API Gateway) = ~$0.29/day
   - WebSocket connection hours would cost significantly more

4. **Simplicity**: No connection state, no reconnection logic, no message queuing. Just HTTP requests that either succeed or fail.

### Implementation

```typescript
// Seller App - React Native
import { useQuery } from "@tanstack/react-query";

function useHeatmapData(sellerId: string, location: Coordinates) {
  return useQuery({
    queryKey: ["heatmap", sellerId, location],
    queryFn: () => fetchHeatmap(sellerId, location),
    refetchInterval: 30_000, // 30 seconds
    refetchIntervalInBackground: false, // Stop when app backgrounded
    staleTime: 25_000, // Data considered fresh for 25s
  });
}
```

---

## API Endpoints

### Location Update (Buyer App → API)

```
POST /locations
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "latitude": 33.5731,
  "longitude": -7.5898,
  "accuracy": 10.5,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Response:** `201 Created`

### Heatmap Data (API → Seller App)

```
GET /locations/heatmap?lat=33.5731&lng=-7.5898&radius=500
Authorization: Bearer <jwt>
```

**Response:**
```json
{
  "cells": [
    { "lat": 33.573, "lng": -7.589, "weight": 15 },
    { "lat": 33.574, "lng": -7.590, "weight": 8 }
  ],
  "totalUsers": 42,
  "generatedAt": "2024-01-15T10:30:00Z"
}
```

---

## Security Considerations

### 1. Authentication

JWT-based auth with short-lived access tokens:

```typescript
// packages/core/src/auth/verify.ts
import { jwtVerify } from "jose";

export async function verifyToken(token: string) {
  const { payload } = await jwtVerify(
    token,
    new TextEncoder().encode(process.env.JWT_SECRET)
  );
  return payload as { userId: string; role: "buyer" | "seller" };
}
```

### 2. Rate Limiting

Prevent abuse with per-user rate limits:

```typescript
// Implemented at API Gateway level
const api = new Api(stack, "Api", {
  defaults: {
    throttle: {
      rate: 100,  // requests per second
      burst: 200, // burst capacity
    },
  },
});
```

### 3. Data Privacy

- Location data expires after 15 minutes
- No persistent location history stored
- Heatmap data is aggregated, never individual positions

---

## Deployment Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   GitHub    │───▶│  GitHub     │───▶│    AWS      │
│   Push      │    │  Actions    │    │  (SST)      │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │   Environments        │
              │                       │
              │   main → production   │
              │   dev  → staging      │
              │   PR   → preview      │
              └───────────────────────┘
```

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main, dev]
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npx sst deploy --stage ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Cost Estimation (Monthly)

| Service | Usage | Cost |
|---------|-------|------|
| Lambda | 10M invocations, 512MB, 500ms avg | ~$20 |
| API Gateway | 10M requests | ~$35 |
| CloudFront | 100GB transfer | ~$8 |
| Neon DB | Pro plan (autoscaling) | ~$19 |
| **Total** | | **~$82/month** |

*Costs scale linearly with usage. Neon's autoscaling ensures you only pay for active compute time.*

---

## Summary

This architecture delivers:

- **Zero Friction**: Simple REST APIs, no connection management
- **Serverless Scale**: Lambda handles traffic spikes automatically
- **Cost Efficiency**: Pay-per-use pricing, no idle resources
- **Operational Simplicity**: SST manages infrastructure as code
- **Morocco-Optimized**: eu-west-1 region provides lowest latency

The 30-second polling strategy strikes the perfect balance between real-time UX and operational simplicity, aligning with the retail decision-making cadence while keeping infrastructure costs minimal.

## Related Documents

**Dependencies**
- STRAT-01: Section 6

**Related Specs**
- TECH-01: Section 2
- TECH-06: Section 2
- TECH-07: Section 3
- ADR-001: Section 4

**Implementation Guides**
- IMPL-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Engineering Lead | Added ADR reference |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
