---
document_id: PRD-01
version: 1.2
status: Final
priority: P0
last_updated: 2026-01-30
owner: Product Lead
dependencies:
  - STRAT-01
  - BRAND-01
  - DES-01
related_documents:
  - PRD-02
  - TECH-06
  - METRICS-03
  - GUIDE-01
  - DATA-01
  - THREAD-01
  - THREAD-02
  - THREAD-03
---

# PRD-01: Buyer App

## Product Requirements Document

**Version:** 1.0
**Date:** January 2026
**Platform:** React Native (Expo)
**Classification:** Product Specification

---

## Executive Summary

The Buyer App is the consumer-facing mobile application that makes nearby savings easy to discover. Users see deals on a clean, map-first interface, claim offers, and redeem them via SafeColor Verification (dynamic QR + color code).

**Core Value Proposition:** "If a deal is nearby and worth your time, you'll see it."

---

## User Stories

### Epic 1: Onboarding

| ID | As a... | I want to... | So that... | Priority |
|----|---------|--------------|------------|----------|
| B-001 | New user | Sign up with my phone number | I can start discovering deals | P0 |
| B-002 | New user | Grant location permission | The app can show deals near me | P0 |
| B-003 | New user | Understand the app in under 30 seconds | I don't abandon during setup | P0 |
| B-004 | User | Skip detailed profile setup initially | I can explore immediately | P1 |

### Epic 2: Deal Discovery

| ID | As a... | I want to... | So that... | Priority |
|----|---------|--------------|------------|----------|
| B-010 | Buyer | See a map with my location | I know where I am relative to deals | P0 |
| B-011 | Buyer | See deal markers on the map | I can identify nearby offers | P0 |
| B-012 | Buyer | Filter deals by category | I find relevant offers faster | P1 |
| B-013 | Buyer | See deal details when tapping an icon | I can evaluate the offer | P0 |
| B-014 | Buyer | See distance to each deal | I can decide if it's worth walking | P0 |
| B-015 | Buyer | See time remaining on deals | I feel urgency to act | P0 |
| B-016 | Buyer | Receive push notifications for limited-time deals | I don't miss time-sensitive offers | P0 |

### Epic 3: Deal Claiming & Redemption

| ID | As a... | I want to... | So that... | Priority |
|----|---------|--------------|------------|----------|
| B-020 | Buyer | Claim a deal with one tap | The offer is reserved for me | P0 |
| B-021 | Buyer | See claimed deals in my "Wallet" | I can access them later | P0 |
| B-022 | Buyer | Generate a QR code for redemption | The seller can verify my claim | P0 |
| B-023 | Buyer | See a dynamic color code with my QR | Fraud is prevented | P0 |
| B-024 | Buyer | Know my redemption succeeded | I have peace of mind | P0 |

### Epic 4: Membership & Rewards

| ID | As a... | I want to... | So that... | Priority |
|----|---------|--------------|------------|----------|
| B-030 | Buyer | Earn points for daily app opens | I'm rewarded for engagement | P1 |
| B-031 | Buyer | See my streak count | I'm motivated to maintain it | P1 |
| B-032 | Buyer | See my membership tier | I understand my benefits | P1 |
| B-033 | Buyer | Unlock member benefits | I feel rewarded | P2 |

---

## Feature Specifications

### F1: Map Interface (Mapbox Integration)

#### Description
The primary interface is a full-screen dark-mode map powered by Mapbox GL JS via `@rnmapbox/maps`.

#### Technical Requirements

```typescript
// Map configuration
const mapConfig = {
  styleURL: "mapbox://styles/mapbox/dark-v11", // Custom dark style
  centerCoordinate: [userLongitude, userLatitude],
  zoomLevel: 15, // Street-level default
  pitchEnabled: false, // 2D view only
  rotateEnabled: true,
  attributionEnabled: false,
};
```

#### Visual Specifications

| Element | Appearance | Behavior |
|---------|------------|----------|
| User location | Blue dot (#3B82F6) | Gentle pulse (scale 0.95-1.05) |
| Deal marker (standard) | Amber tag (#F59E0B) | Subtle pulse, 3s cycle |
| Limited-time deal | Red tag (#EF4444) | Subtle pulse, 3s cycle |
| Deal expiring soon | Red outline | Urgency indicator (no shake) |
| Range ring | Thin gray circle | 500m radius from user |
| Heatmap (optional) | Blue → amber → red | Shows general activity |

#### Home Screen Overlays

- **FloatingHeader:** location context + notification bell
- **FloatingSearchBar:** rounded search pill
- **Limited-Time Deal Toast:** "3 limited-time deals nearby"
- **BottomTabBar:** 5 tabs (Home, Explore, Wallet, Profile, Menu)
#### Component Structure

```typescript
// packages/apps/buyer/src/screens/MapScreen.tsx

import MapboxGL from "@rnmapbox/maps";

export function MapScreen() {
  return (
    <MapboxGL.MapView style={styles.map} styleURL={DARK_STYLE_URL}>
      {/* User location */}
      <MapboxGL.UserLocation
        visible={true}
        renderMode="native"
        animated={true}
      />

      {/* Deal markers layer */}
      <MapboxGL.ShapeSource id="deals" shape={dealsGeoJSON}>
        <MapboxGL.SymbolLayer
          id="deal-icons"
          style={dealIconStyle}
        />
      </MapboxGL.ShapeSource>

      {/* Range ring */}
      <MapboxGL.ShapeSource id="range" shape={rangeCircle}>
        <MapboxGL.LineLayer
          id="range-ring"
          style={rangeRingStyle}
        />
      </MapboxGL.ShapeSource>
    </MapboxGL.MapView>
  );
}
```

#### Home Screen Layout (Map)

```
┌──────────────────────────────────────────┐
│  📍 Maarif, Casablanca          [ 🔔 ]   │
├──────────────────────────────────────────┤
│         [ 🔍 Search Deals... ]           │
│                                          │
│               🏷️                         │
│         🔵                               │
│                    🏷️                    │
│                                          │
│      [ ⚡ 3 Limited-Time Deals Nearby ]  │
├──────────────────────────────────────────┤
│  🏠      🧭       💼       👤      ⚙️    │
│ Home   Explore   Wallet   Profile  Menu  │
└──────────────────────────────────────────┘
```

#### Deal Discovery (Bottom Sheet)

```
┌──────────────────────────────────────────┐
│              ____                        │
│   [  IMAGE  ]   **Café Milano**          │
│   [         ]   Maarif • 120m away       │
│   ┌──────────────────────────────────┐   │
│   │  ☕ 30% OFF ALL COFFEE           │   │
│   └──────────────────────────────────┘   │
│   Valid for: 14 mins                     │
│   ⭐⭐⭐⭐⭐ (4.8)                       │
│   [       CLAIM DEAL (FREE)      ]       │
└──────────────────────────────────────────┘
```

---

### F2: Foreground Location Tracking

#### Description
The app tracks user location while in the foreground to enable deal discovery and contribute to the network's activity data.

#### Technical Requirements

**Battery-Optimized Strategy:**

```typescript
// packages/apps/buyer/src/hooks/useLocationTracking.ts

import * as Location from "expo-location";
import { useEffect, useRef } from "react";

const LOCATION_CONFIG = {
  accuracy: Location.Accuracy.Balanced, // ~100m accuracy, battery-friendly
  distanceInterval: 50, // Update every 50m of movement
  timeInterval: 30000, // Or every 30 seconds, whichever comes first
};

export function useLocationTracking() {
  const lastSentRef = useRef<number>(0);
  const locationBuffer = useRef<Location.LocationObject[]>([]);

  useEffect(() => {
    let subscription: Location.LocationSubscription;

    const startTracking = async () => {
      const { status } = await Location.requestForegroundPermissionsAsync();
      if (status !== "granted") return;

      subscription = await Location.watchPositionAsync(
        LOCATION_CONFIG,
        handleLocationUpdate
      );
    };

    startTracking();
    return () => subscription?.remove();
  }, []);

  const handleLocationUpdate = (location: Location.LocationObject) => {
    locationBuffer.current.push(location);

    // Batch send every 30 seconds
    const now = Date.now();
    if (now - lastSentRef.current >= 30000) {
      sendLocationBatch(locationBuffer.current);
      locationBuffer.current = [];
      lastSentRef.current = now;
    }
  };

  const sendLocationBatch = async (locations: Location.LocationObject[]) => {
    if (locations.length === 0) return;

    // Send only the most recent location
    const latest = locations[locations.length - 1];

    await api.post("/locations", {
      latitude: truncateCoordinate(latest.coords.latitude),
      longitude: truncateCoordinate(latest.coords.longitude),
      accuracy: latest.coords.accuracy,
      timestamp: new Date(latest.timestamp).toISOString(),
    });
  };
}

// Privacy: Truncate to 3 decimal places (~111m precision)
function truncateCoordinate(coord: number): number {
  return Math.round(coord * 1000) / 1000;
}
```

**Location Permission Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│                    PERMISSION FLOW                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1. User opens app for first time                          │
│                     │                                        │
│                     ▼                                        │
│   2. Show value proposition screen:                         │
│      "Allow location to discover deals within 500m"         │
│                     │                                        │
│                     ▼                                        │
│   3. System permission prompt appears                       │
│                     │                                        │
│        ┌───────────┴───────────┐                            │
│        ▼                       ▼                            │
│   [GRANTED]               [DENIED]                          │
│        │                       │                            │
│        ▼                       ▼                            │
│   Start tracking          Show limited mode:                │
│   Show full map           "Enable location in Settings"     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Battery Considerations:**

| Setting | Value | Rationale |
|---------|-------|-----------|
| Accuracy | Balanced | 100m precision, uses WiFi/Cell, minimal GPS |
| Distance interval | 50m | Only update on significant movement |
| Time interval | 30s | Cap update frequency |
| Background tracking | Disabled | Preserves battery, respects privacy |

---

### F3: My Deals (Wallet)

#### Description
A dedicated section where users can view claimed deals grouped by status (Ready to redeem, Redeemed, Expired).

**Implementation note:** Use a `SectionList` grouped by status with a primary "Use Now" action.

#### UI Layout

```
┌──────────────────────────────────────────┐
│  My Deals                      History   │
├──────────────────────────────────────────┤
│  READY TO REDEEM (2)                     │
│  ┌────────────────────────────────────┐  │
│  │ 🏷️  30% Off Coffee                 │  │
│  │     Café Milano                    │  │
│  │     📍 120m • Expires: 14m         │  │
│  │     [ USE NOW > ]                  │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

#### Data Model

```typescript
interface ClaimedDeal {
  id: string;
  dealId: string;
  storeName: string;
  storeImageUrl: string;
  title: string;
  discountType: "percentage" | "fixed" | "bogo" | "freebie";
  discountValue: number;
  claimedAt: Date;
  expiresAt: Date;
  storeLocation: {
    latitude: number;
    longitude: number;
  };
  status: "claimed" | "redeemed" | "expired";
}
```

---

### F4: QR Redemption (SafeColor Verification)

#### Description
The redemption step — the buyer presents a dynamic QR code with a color pattern to the seller for verification.

#### Screen Layout

```
┌──────────────────────────────────────────┐
│  < Back                                  │
├──────────────────────────────────────────┤
│  Show to Cashier                         │
│       [QR CODE]                          │
│    SafeColor ID:                         │
│    [   🟢   PULSING GREEN   🟢   ]       │
│    Café Milano • -15 DH                  │
│          [ Cancel Redemption ]           │
└──────────────────────────────────────────┘
```

#### Technical Implementation

```typescript
// packages/apps/buyer/src/screens/RedemptionScreen.tsx

import QRCode from "react-native-qrcode-svg";
import { useEffect, useState } from "react";

interface RedemptionProps {
  dealId: string;
  redemptionId: string;
}

export function RedemptionScreen({ dealId, redemptionId }: RedemptionProps) {
  const [qrPayload, setQrPayload] = useState<string>("");
  const [colorCode, setColorCode] = useState<string[]>(["green", "green", "amber"]);
  const [expiresIn, setExpiresIn] = useState<number>(300); // 5 minutes

  // Generate initial QR and color code
  useEffect(() => {
    initializeRedemption();
  }, []);

  // Refresh QR every 30 seconds
  useEffect(() => {
    const interval = setInterval(refreshQrCode, 30000);
    return () => clearInterval(interval);
  }, []);

  // Countdown timer
  useEffect(() => {
    const interval = setInterval(() => {
      setExpiresIn((prev) => Math.max(0, prev - 1));
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  const initializeRedemption = async () => {
    const response = await api.post("/redemptions/initiate", {
      dealId,
    });

    setQrPayload(response.qrCode);
    setColorCode(response.colorCode);
  };

  const refreshQrCode = async () => {
    const response = await api.post("/redemptions/refresh", {
      redemptionId,
    });

    setQrPayload(response.qrCode);
    setColorCode(response.colorCode);
  };

  return (
    <RedemptionContainer>
      <RedemptionCard>
        <QRCode
          value={qrPayload}
          size={200}
          backgroundColor="transparent"
          color="#FFFFFF"
        />
        <ColorCodeDisplay colors={colorCode} />
        <Text>Show to cashier</Text>
        <Text>Expires in {formatTime(expiresIn)}</Text>
      </RedemptionCard>
    </RedemptionContainer>
  );
}
```

#### QR Payload Structure

```typescript
interface QRPayload {
  rid: string;      // Redemption ID
  uid: string;      // User ID (hashed)
  did: string;      // Deal ID
  ts: number;       // Timestamp (Unix)
  sig: string;      // HMAC signature
}

// Encoded as base64 JSON
// Example: eyJyaWQiOiJhYmMxMjMiLCJ1aWQiOiJ4eXoiLC4uLn0=
```

#### Color Code System

| Pattern | Meaning | Frequency |
|---------|---------|-----------|
| 🟢🟡 | Standard deal (2-color) | Most common |
| 🟢🟡🔵 | High-value deal (3-color) | >500 MAD savings |
| 🟢🟡🔵🔴 | Limited-time premium (4-color) | Limited edition |

**Color Codes:**
- 🟢 Green: `#10B981`
- 🟡 Amber: `#F59E0B`
- 🔵 Blue: `#3B82F6`
- 🔴 Red: `#EF4444`

---

### F5: Push Notifications

#### Description
Real-time alerts for limited-time deals and proximity-based triggers.

#### Notification Types

| Type | Trigger | Template |
|------|---------|----------|
| Limited-Time Deal | Seller broadcasts to nearby users | "⚡ Limited-time: {discount}% off at {store} — {distance}m away" |
| Proximity | User enters deal zone | "Deal nearby: {store} — {discount} (expires {time})" |
| Expiration | Claimed deal expiring soon | "⏰ Your deal at {store} expires in 15 min" |
| Achievement | User unlocks badge | "🏆 Badge unlocked: {badge_name}" |

#### Implementation

```typescript
// packages/apps/buyer/src/services/notifications.ts

import * as Notifications from "expo-notifications";
import * as Device from "expo-device";

export async function registerForPushNotifications() {
  if (!Device.isDevice) return;

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== "granted") {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== "granted") return;

  const token = await Notifications.getExpoPushTokenAsync();
  await api.post("/users/push-token", { token: token.data });
}

// Notification handler
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});
```

---

### F6: Profile

#### Description
The Profile screen summarizes membership tier, total savings, and activity, with quick access to settings.

#### UI Layout

```
┌──────────────────────────────────────────┐
│  Profile                       [Edit]    │
├──────────────────────────────────────────┤
│  ( Photo )   Youssef Benali              │
│              Gold Member 🌟              │
│  ┌────────────────────────────────────┐  │
│  │  TOTAL SAVINGS: 420 DH             │  │
│  └────────────────────────────────────┘  │
│  YOUR ACTIVITY                           │
│  • Deals Redeemed: 12                    │
│  • Current Streak: 4 Days 🔥             │
│  SETTINGS                                │
│  > Notifications                    >    │
│  > Location Privacy                 >    │
└──────────────────────────────────────────┘
```

## Screen Flow

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           BUYER APP FLOW                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐            │
│  │ Splash  │────▶│Onboard- │────▶│ Phone   │────▶│Location │            │
│  │         │     │ ing     │     │ Auth    │     │Permission│            │
│  └─────────┘     └─────────┘     └─────────┘     └────┬────┘            │
│                                                       │                   │
│                                                       ▼                   │
│                                                  ┌─────────┐             │
│                                                  │  HOME   │◀────┐       │
│                                                  │  (Map)  │     │       │
│                                                  └────┬────┘     │       │
│                                                       │          │       │
│                    ┌──────────────────────────────────┼──────────┘       │
│                    │                                  │                   │
│                    ▼                                  ▼                   │
│              ┌─────────┐                        ┌─────────┐              │
│              │ Wallet  │                        │  Deal   │              │
│              │ (Tab)   │                        │ Detail  │              │
│              └────┬────┘                        └────┬────┘              │
│                   │                                  │                   │
│                   │                                  ▼                   │
│                   │                            ┌─────────┐              │
│                   └──────────────────────────▶│ Redeem  │              │
│                                                │  (QR)   │              │
│                                                └────┬────┘              │
│                                                     │                   │
│                                                     ▼                   │
│                                                ┌─────────┐              │
│                                                │ Success │              │
│                                                └─────────┘              │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Tab Navigation

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│                         [Main Content Area]                        │
│                                                                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│    🏠      🧭       💼       👤      ⚙️                            │
│    Home   Explore   Wallet   Profile  Menu                         │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## API Endpoints Used

| Endpoint | Method | Description |
|----------|--------|-------------|
| `POST /auth/phone/request` | POST | Request OTP |
| `POST /auth/phone/verify` | POST | Verify OTP, get JWT |
| `POST /locations` | POST | Send location update |
| `GET /deals/nearby` | GET | Fetch deals near coordinates |
| `GET /deals/{id}` | GET | Get deal details |
| `POST /deals/{id}/claim` | POST | Claim a deal |
| `GET /wallet` | GET | Get claimed deals |
| `POST /redemptions/initiate` | POST | Start redemption flow |
| `POST /redemptions/refresh` | POST | Refresh QR code |
| `POST /users/push-token` | POST | Register push token |

---

## Non-Functional Requirements

### Performance

| Metric | Target |
|--------|--------|
| App launch to map visible | < 2 seconds |
| Location update latency | < 100ms |
| Deal list load time | < 500ms |
| QR code generation | < 200ms |

### Battery

| Scenario | Target Drain |
|----------|--------------|
| Active use (1 hour) | < 10% battery |
| Background (should not run) | 0% |

### Offline Behavior

| Feature | Offline Behavior |
|---------|------------------|
| Map | Show cached tiles, no deals |
| Wallet | Show cached claimed deals |
| Redemption | Requires connection (validation) |

---

## Success Metrics

| Metric | Target (Month 3) |
|--------|------------------|
| DAU | 5,000 |
| Deals viewed / user / session | 3+ |
| Claim rate (view → claim) | 20% |
| Redemption rate (claim → redeem) | 60% |
| 7-day retention | 35% |
| App store rating | 4.5+ |

---

## Open Questions

1. **Should we show deals from inactive stores?** (Stores that haven't opened the app in 7+ days)
2. **Maximum deals shown on map at once?** (Performance vs. discovery)
3. **Should expired-but-not-redeemed deals stay in wallet?** (History vs. clutter)

---

**Previous:** [BRAND-02: Membership & Rewards](../brand/BRAND-02-membership-rewards.md)
**Next:** [PRD-02: Seller App →](./PRD-02-seller-app.md)

## Related Documents

**Dependencies**
- STRAT-01: Section 2
- BRAND-01: Section 2
- DES-01: Section 1

**Related Specs**
- TECH-06: Section 2
- METRICS-03: Section 3
- DATA-01: Section 2
- THREAD-01: Section 3
- THREAD-02: Section 3
- THREAD-03: Section 3

**Implementation Guides**
- GUIDE-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Product Lead | Updated UI specs and terminology |
| 1.1 | 2026-01-30 | Product Lead | Added data dictionary and thread references |
| 1.0 | 2026-01-30 | Product Lead | Standardized metadata and cross-references |
