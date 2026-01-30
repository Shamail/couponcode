---
document_id: TECH-01
version: 1.2
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-00
related_documents:
  - TECH-02
  - TECH-03
  - TECH-09
  - DATA-01
  - ADR-001
  - THREAD-01
---

# TECH-01: Neon Database Schema

## Executive Summary

This document defines the core Postgres/PostGIS schema for Frictionless, including spatial tables, indexes, and relationships. The schema supports high-frequency location ingestion, heatmap aggregation, and secure redemption flows.

It should be updated only via migrations and remain the single source of truth for data modeling decisions.

## Overview

This document defines the PostgreSQL schema for Frictionless, leveraging Neon's serverless PostgreSQL with PostGIS for spatial operations. The schema follows the "Postgres for everything" philosophy—no separate caching layers or specialized databases.

---

## PostGIS Extension

PostGIS adds geographic object support to PostgreSQL, enabling spatial queries like "find all users within 500 meters."

```sql
-- Enable PostGIS extension (run once in Neon console or migration)
CREATE EXTENSION IF NOT EXISTS postgis;

-- Verify installation
SELECT PostGIS_Version();
-- Expected: "3.4 USE_GEOS=1 USE_PROJ=1 USE_STATS=1"
```

**Why PostGIS?**
- Native PostgreSQL integration (no external services)
- Battle-tested spatial indexing (20+ years of development)
- Rich function library (distance, containment, intersection)
- Works seamlessly with Neon's connection pooling

---

## Schema Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FRICTIONLESS SCHEMA                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────┐          ┌─────────────────────┐                   │
│  │       users         │          │       stores        │                   │
│  ├─────────────────────┤          ├─────────────────────┤                   │
│  │ id (PK)             │          │ id (PK)             │                   │
│  │ phone               │          │ name                │                   │
│  │ name                │          │ owner_id (FK)───────┼──┐                │
│  │ role                │◄─────────┤ location (POINT)    │  │                │
│  │ created_at          │          │ address             │  │                │
│  │ updated_at          │          │ created_at          │  │                │
│  └──────────┬──────────┘          └──────────┬──────────┘  │                │
│             │                                │              │                │
│             │ 1:N                            │ 1:N          │                │
│             ▼                                ▼              │                │
│  ┌─────────────────────┐          ┌─────────────────────┐  │                │
│  │   user_locations    │          │       deals         │  │                │
│  ├─────────────────────┤          ├─────────────────────┤  │                │
│  │ id (PK)             │          │ id (PK)             │  │                │
│  │ user_id (FK)────────┼──────────│ store_id (FK)───────┼──┘                │
│  │ geom (POINT)        │          │ title               │                   │
│  │ accuracy            │          │ description         │                   │
│  │ recorded_at         │          │ discount_type       │                   │
│  │ expires_at          │          │ discount_value      │                   │
│  └─────────────────────┘          │ radius_meters       │                   │
│            ▲                      │ max_redemptions     │                   │
│            │                      │ starts_at           │                   │
│            │                      │ expires_at          │                   │
│            │                      │ created_at          │                   │
│            │                      └──────────┬──────────┘                   │
│            │                                 │                              │
│            │                                 │ 1:N                          │
│            │                                 ▼                              │
│            │                      ┌─────────────────────┐                   │
│            │                      │    redemptions      │                   │
│            │                      ├─────────────────────┤                   │
│            └──────────────────────│ user_id (FK)        │                   │
│                                   │ deal_id (FK)────────┼───────────────────┤
│                                   │ seller_id (FK)      │                   │
│                                   │ status              │                   │
│                                   │ qr_code             │                   │
│                                   │ color_code          │                   │
│                                   │ initiated_at        │                   │
│                                   │ verified_at         │                   │
│                                   │ location (POINT)    │                   │
│                                   └─────────────────────┘                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Table Definitions

### 1. Users Table

Stores both buyers and sellers with role-based differentiation.

```sql
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone           VARCHAR(20) UNIQUE NOT NULL,
    phone_verified  BOOLEAN DEFAULT FALSE,
    name            VARCHAR(100),
    role            VARCHAR(10) NOT NULL CHECK (role IN ('buyer', 'seller')),
    avatar_url      TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Index for phone lookups during authentication
CREATE INDEX idx_users_phone ON users(phone);

-- Index for role-based queries
CREATE INDEX idx_users_role ON users(role);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

**Sample Data:**
```sql
INSERT INTO users (phone, name, role) VALUES
    ('+212600000001', 'Ahmed Buyer', 'buyer'),
    ('+212600000002', 'Fatima Seller', 'seller');
```

---

### 2. User Locations Table (Spatial)

Tracks real-time buyer locations with automatic expiration.

```sql
CREATE TABLE user_locations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    geom            GEOMETRY(Point, 4326) NOT NULL,  -- WGS84 coordinate system
    accuracy        REAL,                             -- GPS accuracy in meters
    recorded_at     TIMESTAMPTZ DEFAULT NOW(),
    expires_at      TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '30 minutes')
);

-- SP-GiST index for fast spatial queries
-- SP-GiST outperforms GiST for point data and nearest-neighbor queries
CREATE INDEX idx_user_locations_geom ON user_locations USING SPGIST(geom);

-- B-tree index for time-based queries (cleanup, freshness)
CREATE INDEX idx_user_locations_expires ON user_locations(expires_at);

-- Composite index for user's latest location
CREATE INDEX idx_user_locations_user_time ON user_locations(user_id, recorded_at DESC);
```

**Why SP-GiST over GiST?**

| Index Type | Best For | Performance |
|------------|----------|-------------|
| GiST | Overlapping geometries, polygons | Good for complex shapes |
| **SP-GiST** | **Point data, non-overlapping** | **2-3x faster for points** |
| BRIN | Sorted spatial data | Smallest size, slower queries |

SP-GiST (Space-Partitioned GiST) uses quadtree partitioning, which is optimal for point data like user locations.

**Location Insert Function:**
```sql
-- Function to insert/update user location
CREATE OR REPLACE FUNCTION upsert_user_location(
    p_user_id UUID,
    p_latitude DOUBLE PRECISION,
    p_longitude DOUBLE PRECISION,
    p_accuracy REAL DEFAULT NULL
)
RETURNS UUID AS $$
DECLARE
    v_location_id UUID;
BEGIN
    INSERT INTO user_locations (user_id, geom, accuracy)
    VALUES (
        p_user_id,
        ST_SetSRID(ST_MakePoint(p_longitude, p_latitude), 4326),
        p_accuracy
    )
    RETURNING id INTO v_location_id;

    RETURN v_location_id;
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
SELECT upsert_user_location(
    'a1b2c3d4-...',  -- user_id
    33.5731,          -- latitude
    -7.5898,          -- longitude
    10.5              -- accuracy in meters
);
```

#### 2.1 Partitioning Strategy (Scale Path)

High-frequency check-ins (every 30s) will grow `user_locations` rapidly. For scale, partition the table by day to keep ingestion fast and retention cleanup O(1).

```sql
CREATE TABLE user_locations_partitioned (
    id UUID DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    geom GEOMETRY(Point, 4326) NOT NULL,
    recorded_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (id, recorded_at)
) PARTITION BY RANGE (recorded_at);
```

**Partition manager cron (daily at midnight):**
- Create partitions 2 days ahead
- Drop partitions older than 3 days

**Read/Write Splitting Thresholds**

| Metric | Threshold | Action |
| --- | --- | --- |
| Read RPM | >10,000 | Route heatmap reads to replica |
| Write RPM | >5,000 | Consider batching check-ins |
| p95 latency | >500ms | Enable replica failover |

---

### 3. Stores Table

Represents physical retail locations owned by sellers.

```sql
CREATE TABLE stores (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id        UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name            VARCHAR(200) NOT NULL,
    description     TEXT,
    location        GEOMETRY(Point, 4326) NOT NULL,
    address         TEXT,
    category        VARCHAR(50),
    image_url       TEXT,
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Spatial index for store lookups
CREATE INDEX idx_stores_location ON stores USING SPGIST(location);

-- Index for owner queries
CREATE INDEX idx_stores_owner ON stores(owner_id);

-- Index for active stores
CREATE INDEX idx_stores_active ON stores(is_active) WHERE is_active = TRUE;

CREATE TRIGGER update_stores_updated_at
    BEFORE UPDATE ON stores
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

---

### 4. Deals Table (Spatial)

Stores promotional offers with spatial radius for targeting.

```sql
CREATE TABLE deals (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id            UUID NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
    title               VARCHAR(200) NOT NULL,
    description         TEXT,
    discount_type       VARCHAR(20) NOT NULL CHECK (discount_type IN ('percentage', 'fixed', 'bogo', 'freebie')),
    discount_value      DECIMAL(10, 2) NOT NULL,
    original_price      DECIMAL(10, 2),
    terms_conditions    TEXT,
    radius_meters       INTEGER DEFAULT 500,          -- Targeting radius
    max_redemptions     INTEGER,                      -- NULL = unlimited
    current_redemptions INTEGER DEFAULT 0,
    starts_at           TIMESTAMPTZ DEFAULT NOW(),
    expires_at          TIMESTAMPTZ NOT NULL,
    is_active           BOOLEAN DEFAULT TRUE,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    updated_at          TIMESTAMPTZ DEFAULT NOW()
);

-- Index for active deals lookup
CREATE INDEX idx_deals_active ON deals(is_active, starts_at, expires_at)
    WHERE is_active = TRUE;

-- Index for store's deals
CREATE INDEX idx_deals_store ON deals(store_id);

-- Partial index for deals with remaining redemptions
CREATE INDEX idx_deals_available ON deals(id)
    WHERE is_active = TRUE
    AND (max_redemptions IS NULL OR current_redemptions < max_redemptions);

CREATE TRIGGER update_deals_updated_at
    BEFORE UPDATE ON deals
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

**Sample Deal:**
```sql
INSERT INTO deals (store_id, title, description, discount_type, discount_value, radius_meters, expires_at)
VALUES (
    'store-uuid-here',
    '20% Off All Tagines',
    'Valid for dine-in only. Cannot combine with other offers.',
    'percentage',
    20.00,
    300,  -- 300 meter radius
    NOW() + INTERVAL '7 days'
);
```

---

### 5. Redemptions Table

Tracks the "SafeColor Verification" redemption process.

```sql
CREATE TABLE redemptions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    deal_id         UUID NOT NULL REFERENCES deals(id),
    seller_id       UUID NOT NULL REFERENCES users(id),  -- Staff who verified
    status          VARCHAR(20) DEFAULT 'initiated'
                    CHECK (status IN ('initiated', 'verified', 'expired', 'cancelled')),

    -- SafeColor Verification components
    qr_code         VARCHAR(64) NOT NULL,               -- Dynamic QR payload
    color_code      VARCHAR(7) NOT NULL,                -- Hex color (#FF5733)

    -- Location verification
    location        GEOMETRY(Point, 4326),              -- Where redemption occurred

    -- Timestamps
    initiated_at    TIMESTAMPTZ DEFAULT NOW(),
    expires_at      TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '5 minutes'),
    verified_at     TIMESTAMPTZ,

    -- Prevent duplicate redemptions
    UNIQUE(user_id, deal_id)
);

-- Index for QR code lookup (verification flow)
CREATE INDEX idx_redemptions_qr ON redemptions(qr_code) WHERE status = 'initiated';

-- Index for user's redemption history
CREATE INDEX idx_redemptions_user ON redemptions(user_id, initiated_at DESC);

-- Index for deal's redemption stats
CREATE INDEX idx_redemptions_deal ON redemptions(deal_id, status);

-- Index for seller's verification history
CREATE INDEX idx_redemptions_seller ON redemptions(seller_id, verified_at DESC);

-- Spatial index for redemption location analysis
CREATE INDEX idx_redemptions_location ON redemptions USING SPGIST(location);
```

**Redemption Status Flow:**
```
initiated → verified   (Success: QR + color matched)
initiated → expired    (Timeout: 5 minutes passed)
initiated → cancelled  (User cancelled)
```

---

## Spatial Queries

### Query 1: Find Nearby Active Users (Heatmap)

Returns aggregated user density for the seller's heatmap.

```sql
-- Get user counts in grid cells within radius
CREATE OR REPLACE FUNCTION get_heatmap_data(
    p_center_lat DOUBLE PRECISION,
    p_center_lng DOUBLE PRECISION,
    p_radius_meters INTEGER DEFAULT 500,
    p_cell_size_meters INTEGER DEFAULT 50
)
RETURNS TABLE (
    cell_lat DOUBLE PRECISION,
    cell_lng DOUBLE PRECISION,
    user_count BIGINT
) AS $$
BEGIN
    RETURN QUERY
    WITH center AS (
        SELECT ST_SetSRID(ST_MakePoint(p_center_lng, p_center_lat), 4326)::geography AS geog
    ),
    active_locations AS (
        SELECT DISTINCT ON (ul.user_id)
            ul.geom
        FROM user_locations ul
        CROSS JOIN center c
        WHERE ul.expires_at > NOW()
        AND ST_DWithin(ul.geom::geography, c.geog, p_radius_meters)
        ORDER BY ul.user_id, ul.recorded_at DESC
    )
    SELECT
        -- Snap to grid cells
        ROUND(ST_Y(geom)::numeric, 4)::DOUBLE PRECISION AS cell_lat,
        ROUND(ST_X(geom)::numeric, 4)::DOUBLE PRECISION AS cell_lng,
        COUNT(*)::BIGINT AS user_count
    FROM active_locations
    GROUP BY
        ROUND(ST_Y(geom)::numeric, 4),
        ROUND(ST_X(geom)::numeric, 4);
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
SELECT * FROM get_heatmap_data(33.5731, -7.5898, 500, 50);
```

### Query 2: Find Deals Near User

Returns deals within range, sorted by distance.

```sql
CREATE OR REPLACE FUNCTION get_nearby_deals(
    p_user_lat DOUBLE PRECISION,
    p_user_lng DOUBLE PRECISION,
    p_max_distance_meters INTEGER DEFAULT 1000
)
RETURNS TABLE (
    deal_id UUID,
    store_name VARCHAR,
    title VARCHAR,
    discount_type VARCHAR,
    discount_value DECIMAL,
    distance_meters DOUBLE PRECISION,
    expires_at TIMESTAMPTZ
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        d.id AS deal_id,
        s.name AS store_name,
        d.title,
        d.discount_type,
        d.discount_value,
        ST_Distance(
            s.location::geography,
            ST_SetSRID(ST_MakePoint(p_user_lng, p_user_lat), 4326)::geography
        ) AS distance_meters,
        d.expires_at
    FROM deals d
    JOIN stores s ON d.store_id = s.id
    WHERE d.is_active = TRUE
    AND d.starts_at <= NOW()
    AND d.expires_at > NOW()
    AND (d.max_redemptions IS NULL OR d.current_redemptions < d.max_redemptions)
    AND ST_DWithin(
        s.location::geography,
        ST_SetSRID(ST_MakePoint(p_user_lng, p_user_lat), 4326)::geography,
        LEAST(d.radius_meters, p_max_distance_meters)
    )
    ORDER BY distance_meters ASC
    LIMIT 50;
END;
$$ LANGUAGE plpgsql;
```

**Usage:**
```sql
SELECT * FROM get_nearby_deals(33.5731, -7.5898, 500);
```

---

## Data Maintenance

### Automatic Location Cleanup

Expired locations are cleaned up via a scheduled function.

```sql
-- Function to delete expired locations
CREATE OR REPLACE FUNCTION cleanup_expired_locations()
RETURNS INTEGER AS $$
DECLARE
    deleted_count INTEGER;
BEGIN
    DELETE FROM user_locations
    WHERE expires_at < NOW()
    RETURNING * INTO deleted_count;

    RETURN deleted_count;
END;
$$ LANGUAGE plpgsql;

-- Can be called via pg_cron or Lambda scheduled event
-- SELECT cleanup_expired_locations();
```

### Redemption Stats Update

Trigger to maintain deal redemption counts.

```sql
CREATE OR REPLACE FUNCTION update_deal_redemption_count()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' OR (TG_OP = 'UPDATE' AND NEW.status = 'verified' AND OLD.status != 'verified') THEN
        UPDATE deals
        SET current_redemptions = current_redemptions + 1
        WHERE id = NEW.deal_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_redemption_count
    AFTER INSERT OR UPDATE ON redemptions
    FOR EACH ROW
    WHEN (NEW.status = 'verified')
    EXECUTE FUNCTION update_deal_redemption_count();
```

---

## Migration Script

Complete migration for initial deployment:

```sql
-- migrations/001_initial_schema.sql

-- Enable extensions
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create updated_at trigger function
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Users table
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone           VARCHAR(20) UNIQUE NOT NULL,
    phone_verified  BOOLEAN DEFAULT FALSE,
    name            VARCHAR(100),
    role            VARCHAR(10) NOT NULL CHECK (role IN ('buyer', 'seller')),
    avatar_url      TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_role ON users(role);

CREATE TRIGGER update_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- User locations table
CREATE TABLE user_locations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    geom            GEOMETRY(Point, 4326) NOT NULL,
    accuracy        REAL,
    recorded_at     TIMESTAMPTZ DEFAULT NOW(),
    expires_at      TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '30 minutes')
);

CREATE INDEX idx_user_locations_geom ON user_locations USING SPGIST(geom);
CREATE INDEX idx_user_locations_expires ON user_locations(expires_at);
CREATE INDEX idx_user_locations_user_time ON user_locations(user_id, recorded_at DESC);

-- Stores table
CREATE TABLE stores (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id        UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name            VARCHAR(200) NOT NULL,
    description     TEXT,
    location        GEOMETRY(Point, 4326) NOT NULL,
    address         TEXT,
    category        VARCHAR(50),
    image_url       TEXT,
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_stores_location ON stores USING SPGIST(location);
CREATE INDEX idx_stores_owner ON stores(owner_id);
CREATE INDEX idx_stores_active ON stores(is_active) WHERE is_active = TRUE;

CREATE TRIGGER update_stores_updated_at
    BEFORE UPDATE ON stores
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Deals table
CREATE TABLE deals (
    id                  UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    store_id            UUID NOT NULL REFERENCES stores(id) ON DELETE CASCADE,
    title               VARCHAR(200) NOT NULL,
    description         TEXT,
    discount_type       VARCHAR(20) NOT NULL CHECK (discount_type IN ('percentage', 'fixed', 'bogo', 'freebie')),
    discount_value      DECIMAL(10, 2) NOT NULL,
    original_price      DECIMAL(10, 2),
    terms_conditions    TEXT,
    radius_meters       INTEGER DEFAULT 500,
    max_redemptions     INTEGER,
    current_redemptions INTEGER DEFAULT 0,
    starts_at           TIMESTAMPTZ DEFAULT NOW(),
    expires_at          TIMESTAMPTZ NOT NULL,
    is_active           BOOLEAN DEFAULT TRUE,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    updated_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_deals_active ON deals(is_active, starts_at, expires_at) WHERE is_active = TRUE;
CREATE INDEX idx_deals_store ON deals(store_id);
CREATE INDEX idx_deals_available ON deals(id) WHERE is_active = TRUE AND (max_redemptions IS NULL OR current_redemptions < max_redemptions);

CREATE TRIGGER update_deals_updated_at
    BEFORE UPDATE ON deals
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- Redemptions table
CREATE TABLE redemptions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id),
    deal_id         UUID NOT NULL REFERENCES deals(id),
    seller_id       UUID NOT NULL REFERENCES users(id),
    status          VARCHAR(20) DEFAULT 'initiated' CHECK (status IN ('initiated', 'verified', 'expired', 'cancelled')),
    qr_code         VARCHAR(64) NOT NULL,
    color_code      VARCHAR(7) NOT NULL,
    location        GEOMETRY(Point, 4326),
    initiated_at    TIMESTAMPTZ DEFAULT NOW(),
    expires_at      TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '5 minutes'),
    verified_at     TIMESTAMPTZ,
    UNIQUE(user_id, deal_id)
);

CREATE INDEX idx_redemptions_qr ON redemptions(qr_code) WHERE status = 'initiated';
CREATE INDEX idx_redemptions_user ON redemptions(user_id, initiated_at DESC);
CREATE INDEX idx_redemptions_deal ON redemptions(deal_id, status);
CREATE INDEX idx_redemptions_seller ON redemptions(seller_id, verified_at DESC);
CREATE INDEX idx_redemptions_location ON redemptions USING SPGIST(location);

-- Redemption count trigger
CREATE OR REPLACE FUNCTION update_deal_redemption_count()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status = 'verified' AND (TG_OP = 'INSERT' OR OLD.status != 'verified') THEN
        UPDATE deals SET current_redemptions = current_redemptions + 1 WHERE id = NEW.deal_id;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_redemption_count
    AFTER INSERT OR UPDATE ON redemptions
    FOR EACH ROW
    EXECUTE FUNCTION update_deal_redemption_count();
```

---

## Summary

| Table | Purpose | Spatial | Key Index |
|-------|---------|---------|-----------|
| `users` | Buyer/Seller profiles | No | `idx_users_phone` |
| `user_locations` | Real-time GPS tracking | Yes (Point) | SP-GiST on `geom` |
| `stores` | Retail locations | Yes (Point) | SP-GiST on `location` |
| `deals` | Promotional offers | Via store | Partial on active |
| `redemptions` | SafeColor Verification tracking | Yes (Point) | `idx_redemptions_qr` |

This schema provides:

- **Efficient Spatial Queries**: SP-GiST indexes for point data
- **Automatic Expiration**: Location data self-expires after 30 minutes
- **Referential Integrity**: Foreign keys ensure data consistency
- **Query Optimization**: Partial indexes for common access patterns
- **Audit Trail**: Timestamps on all tables for debugging/analytics

## Related Documents

**Dependencies**
- TECH-00: Section 2

**Related Specs**
- TECH-02: Section 3
- TECH-03: Section 2
- TECH-09: Section 2
- DATA-01: Section 2
- ADR-001: Section 4
- THREAD-01: Section 3
- THREAD-02: Section 3
- THREAD-03: Section 3

**Implementation Guides**
- IMPL-01: Section 3

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Engineering Lead | Updated SafeColor terminology |
| 1.1 | 2026-01-30 | Engineering Lead | Added partitioning strategy and aligned TTL |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
