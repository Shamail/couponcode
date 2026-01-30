---
document_id: PRD-02
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
  - PRD-01
  - TECH-06
  - OPS-01
  - GUIDE-02
  - DATA-01
  - THREAD-02
  - THREAD-03
---

# PRD-02: Seller App (Shadow)

## Product Requirements Document

**Version:** 1.0
**Date:** January 2026
**Classification:** Product Requirements Document

---

## Executive Summary

The Seller App (codename: Shadow) provides merchants with real-time visibility into nearby foot traffic and the tools to convert demand into sales. It is optimized for simplicity, speed, and low training requirements.

This PRD defines the core workflows, requirements, and success metrics for the merchant-facing experience.

## Overview

The Seller App (codename: "Shadow") is the merchant-facing mobile application that provides real-time visibility into nearby foot traffic and enables deal management.

**Core value proposition:** "See your customers before they see you."

---

## Target Users

### Primary Persona: Karim (Small Boutique Owner)

- Age: 35-55
- Tech comfort: Medium (uses WhatsApp, Instagram)
- Business size: 1 location, 2-5 employees
- Pain point: "I see people walking past. Why don't they come in?"
- Goal: Convert foot traffic into paying customers

### Secondary Persona: Amina (Retail Chain Manager)

- Age: 28-40
- Tech comfort: High
- Business size: 3-10 locations
- Pain point: "I can't compare performance across locations"
- Goal: Optimize staffing and promotions based on real traffic data

---

## Platform Requirements

| Requirement | Specification |
|-------------|---------------|
| **Framework** | React Native (Expo) |
| **Platforms** | iOS 14+, Android 10+ |
| **Maps** | Mapbox GL JS via @rnmapbox/maps |
| **Camera** | expo-camera for QR scanning |
| **Auth** | Phone number + OTP |
| **Offline** | Cached deals, offline QR queue |

---

## Core Features

### 1. The Heatmap View

The primary screen. A Mapbox map centered on the merchant's store location with real-time user density overlay.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| HM-01 | Display Mapbox dark-mode map | P0 |
| HM-02 | Show merchant's store location as fixed pin | P0 |
| HM-03 | Overlay heatmap layer from GeoJSON data | P0 |
| HM-04 | Auto-refresh heatmap every 30 seconds | P0 |
| HM-05 | Show activity level badge (Low/Med/High) | P0 |
| HM-06 | Display user count within radius (Pro only) | P1 |
| HM-07 | Allow radius adjustment (100m/300m/500m) | P1 |
| HM-08 | Show last updated timestamp | P2 |

#### Polling Architecture

```typescript
// Heatmap refresh logic
const POLL_INTERVAL = 30000; // 30 seconds

useEffect(() => {
  const fetchHeatmap = async () => {
    const response = await api.get('/map/density', {
      params: {
        lat: storeLocation.latitude,
        lng: storeLocation.longitude,
        radius: selectedRadius
      }
    });
    setHeatmapGeoJSON(response.data);
    setLastUpdated(new Date());
  };

  fetchHeatmap(); // Initial fetch
  const interval = setInterval(fetchHeatmap, POLL_INTERVAL);
  
  return () => clearInterval(interval);
}, [storeLocation, selectedRadius]);
```

#### Heatmap Visualization

```typescript
// Mapbox heatmap layer configuration
const heatmapLayer = {
  id: 'user-density',
  type: 'heatmap',
  source: 'density-data',
  paint: {
    'heatmap-weight': ['get', 'weight'],
    'heatmap-intensity': 1,
    'heatmap-color': [
      'interpolate',
      ['linear'],
      ['heatmap-density'],
      0, 'rgba(0, 0, 0, 0)',
      0.2, 'rgba(59, 130, 246, 0.35)',
      0.4, 'rgba(245, 158, 11, 0.5)',
      0.6, 'rgba(245, 158, 11, 0.7)',
      0.8, 'rgba(239, 68, 68, 0.85)',
      1, 'rgba(239, 68, 68, 1)'
    ],
    'heatmap-radius': 30,
    'heatmap-opacity': 0.7
  }
};
```

#### Activity Level Logic

```typescript
type ActivityLevel = 'low' | 'medium' | 'high';

const getActivityLevel = (userCount: number): ActivityLevel => {
  if (userCount < 10) return 'low';
  if (userCount < 30) return 'medium';
  return 'high';
};

// UI Badge colors
const activityColors = {
  low: '#3B82F6',    // Blue
  medium: '#F59E0B', // Amber
  high: '#EF4444'    // Red
};
```

---

### 2. The QR Validator

Built-in camera view to scan and validate buyer QR codes for deal redemption.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| QR-01 | Camera view with QR scanning overlay | P0 |
| QR-02 | Parse JWT from QR data | P0 |
| QR-03 | Send JWT to API for verification | P0 |
| QR-04 | Display verification result (valid/invalid) | P0 |
| QR-05 | Show matching color code on success | P0 |
| QR-06 | Display customer name + deal details | P0 |
| QR-07 | Haptic feedback on scan | P1 |
| QR-08 | Offline queue for poor connectivity | P2 |

#### Validation Flow

```
┌─────────────────────────────────────────────────────────────┐
│                     QR VALIDATION FLOW                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. Seller taps "Scan" button                              │
│      └── Camera view opens with targeting frame             │
│                                                             │
│   2. Buyer shows QR code                                    │
│      └── QR contains signed JWT                             │
│                                                             │
│   3. Camera detects and decodes QR                          │
│      └── Extract JWT payload                                │
│                                                             │
│   4. App sends JWT to API                                   │
│      POST /redemption/verify                                │
│      Body: { token: "eyJ..." }                              │
│                                                             │
│   5. API responds with verification result                  │
│      {                                                      │
│        valid: true,                                         │
│        color_code: "#00FF66",                               │
│        deal: { title: "30% off", ... },                     │
│        customer: { name: "Yasmine" }                        │
│      }                                                      │
│                                                             │
│   6. App displays result                                    │
│      ├── Valid: Green screen + deal details                 │
│      └── Invalid: Red screen + error message                │
│                                                             │
│   7. Seller confirms visual handshake                       │
│      └── Color on seller app matches buyer app              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Scanner Component

```typescript
import { CameraView, useCameraPermissions } from 'expo-camera';

const QRScanner = () => {
  const [permission, requestPermission] = useCameraPermissions();
  const [scanned, setScanned] = useState(false);
  const [verificationResult, setVerificationResult] = useState(null);

  const handleBarCodeScanned = async ({ data }) => {
    if (scanned) return;
    setScanned(true);
    
    try {
      const result = await api.post('/redemption/verify', { token: data });
      setVerificationResult(result.data);
      
      if (result.data.valid) {
        Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
      } else {
        Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
      }
    } catch (error) {
      setVerificationResult({ valid: false, error: 'Network error' });
    }
  };

  return (
    <CameraView
      style={styles.camera}
      facing="back"
      barcodeScannerSettings={{
        barcodeTypes: ['qr']
      }}
      onBarcodeScanned={scanned ? undefined : handleBarCodeScanned}
    />
  );
};
```

#### Verification Result Screen

```typescript
const VerificationResult = ({ result }) => {
  if (result.valid) {
    return (
      <View style={[styles.result, { backgroundColor: result.color_code }]}>
        <Icon name="check-circle" size={80} color="white" />
        <Text style={styles.title}>Verified!</Text>
        <Text style={styles.dealTitle}>{result.deal.title}</Text>
        <Text style={styles.customer}>Customer: {result.customer.name}</Text>
        <Text style={styles.colorHint}>
          Match this color with customer's screen
        </Text>
      </View>
    );
  }

  return (
    <View style={[styles.result, { backgroundColor: '#FF3333' }]}>
      <Icon name="x-circle" size={80} color="white" />
      <Text style={styles.title}>Invalid</Text>
      <Text style={styles.error}>{result.error}</Text>
    </View>
  );
};
```

---

### 3. Store Setup

Onboarding flow where sellers configure their store location and profile.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| SS-01 | Store name input | P0 |
| SS-02 | Category selection (dropdown) | P0 |
| SS-03 | Map pin placement for store location | P0 |
| SS-04 | Address autocomplete | P1 |
| SS-05 | Business hours configuration | P1 |
| SS-06 | Logo upload | P2 |
| SS-07 | Contact information | P2 |

#### Pin Placement Flow

```typescript
const StoreLocationPicker = () => {
  const [markerPosition, setMarkerPosition] = useState(null);
  const [region, setRegion] = useState(CASABLANCA_DEFAULT);

  const handleMapPress = (event) => {
    const { coordinate } = event.nativeEvent;
    setMarkerPosition(coordinate);
  };

  const handleUseCurrentLocation = async () => {
    const location = await Location.getCurrentPositionAsync({});
    setMarkerPosition({
      latitude: location.coords.latitude,
      longitude: location.coords.longitude
    });
    setRegion({
      ...region,
      latitude: location.coords.latitude,
      longitude: location.coords.longitude
    });
  };

  return (
    <View style={styles.container}>
      <Text style={styles.instruction}>
        Tap the map to place your store pin
      </Text>
      
      <MapboxGL.MapView
        style={styles.map}
        styleURL="mapbox://styles/mapbox/dark-v11"
        onPress={handleMapPress}
      >
        <MapboxGL.Camera
          centerCoordinate={[region.longitude, region.latitude]}
          zoomLevel={15}
        />
        
        {markerPosition && (
          <MapboxGL.PointAnnotation
            id="store-location"
            coordinate={[markerPosition.longitude, markerPosition.latitude]}
          >
            <View style={styles.storeMarker}>
              <Icon name="store" size={24} color="#00FFFF" />
            </View>
          </MapboxGL.PointAnnotation>
        )}
      </MapboxGL.MapView>

      <Button
        title="Use Current Location"
        onPress={handleUseCurrentLocation}
      />
    </View>
  );
};
```

---

### 4. Deal Management

Interface for creating, editing, and managing active deals.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| DM-01 | Create new deal form | P0 |
| DM-02 | Deal title (max 50 chars) | P0 |
| DM-03 | Discount type (percentage/fixed) | P0 |
| DM-04 | Discount value | P0 |
| DM-05 | Expiration date/time | P0 |
| DM-06 | Deal preview card | P0 |
| DM-07 | Activate/deactivate toggle | P0 |
| DM-08 | View redemption count | P1 |
| DM-09 | Edit active deal | P1 |
| DM-10 | Deal templates (quick create) | P2 |

#### Deal Creation Form

```typescript
interface DealForm {
  title: string;
  description: string;
  discountType: 'percentage' | 'fixed';
  discountValue: number;
  expiresAt: Date;
  maxRedemptions?: number;
  terms?: string;
}

const CreateDealScreen = () => {
  const [form, setForm] = useState<DealForm>({
    title: '',
    description: '',
    discountType: 'percentage',
    discountValue: 0,
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000) // Default: 24h
  });

  const handleSubmit = async () => {
    const deal = await api.post('/deals', form);
    navigation.navigate('DealList', { newDeal: deal });
  };

  return (
    <ScrollView style={styles.container}>
      <Input
        label="Deal Title"
        placeholder="e.g., 30% off all items"
        value={form.title}
        onChangeText={(title) => setForm({ ...form, title })}
        maxLength={50}
      />

      <SegmentedControl
        label="Discount Type"
        options={[
          { label: '%', value: 'percentage' },
          { label: 'MAD', value: 'fixed' }
        ]}
        selected={form.discountType}
        onChange={(discountType) => setForm({ ...form, discountType })}
      />

      <Input
        label="Discount Value"
        keyboardType="numeric"
        value={form.discountValue.toString()}
        onChangeText={(v) => setForm({ ...form, discountValue: parseInt(v) || 0 })}
      />

      <DateTimePicker
        label="Expires At"
        value={form.expiresAt}
        onChange={(expiresAt) => setForm({ ...form, expiresAt })}
        minimumDate={new Date()}
      />

      <DealPreview deal={form} />

      <Button title="Create Deal" onPress={handleSubmit} />
    </ScrollView>
  );
};
```

---

### 5. Limited-Time Deal Broadcast (Flash) (Pro Feature)

Ability to push instant notifications to nearby users for limited-time deals.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FB-01 | Limited-time deal creation form | P0 |
| FB-02 | Radius selection (100m/300m/500m) | P0 |
| FB-03 | Preview user count in radius | P0 |
| FB-04 | Cost calculation display | P0 |
| FB-05 | Payment confirmation | P0 |
| FB-06 | Duration setting (30min-2hr) | P1 |
| FB-07 | Broadcast history/analytics | P1 |

#### Limited-Time Deal Creation Flow

```
┌─────────────────────────────────────────────────────────────┐
│                 LIMITED-TIME DEAL BROADCAST                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. Seller sees busy heatmap                               │
│      └── "47 users nearby" indicator                        │
│                                                             │
│   2. Taps "Limited-Time Deal" button                        │
│      └── Opens broadcast modal                              │
│                                                             │
│   3. Configures deal                                        │
│      ├── Select existing deal OR create new                 │
│      ├── Choose radius: 100m / 300m / 500m                  │
│      └── Set duration: 30min / 1hr / 2hr                    │
│                                                             │
│   4. Preview & Cost                                         │
│      ├── "Reach: 47 users"                                  │
│      └── "Cost: 35 MAD"                                     │
│                                                             │
│   5. Confirm & Pay                                          │
│      └── Deduct from wallet or charge card                  │
│                                                             │
│   6. Broadcast deployed                                     │
│      ├── Users receive push notification                    │
│      └── Deal highlighted on buyer maps                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### Broadcast Component

```typescript
const LimitedTimeBroadcast = ({ deal }) => {
  const [radius, setRadius] = useState(300);
  const [duration, setDuration] = useState(60); // minutes
  const [userCount, setUserCount] = useState(0);
  const [cost, setCost] = useState(0);

  // Fetch user count when radius changes
  useEffect(() => {
    const fetchCount = async () => {
      const response = await api.get('/map/density/count', {
        params: {
          lat: store.latitude,
          lng: store.longitude,
          radius
        }
      });
      setUserCount(response.data.count);
      setCost(calculateBroadcastCost(radius, response.data.count));
    };
    fetchCount();
  }, [radius]);

  const calculateBroadcastCost = (radius: number, users: number): number => {
    const baseFees = { 100: 10, 300: 20, 500: 30 };
    const perUserFees = { 100: 0.5, 300: 0.3, 500: 0.2 };
    const base = baseFees[radius];
    const perUser = perUserFees[radius];
    return Math.min(base + (users * perUser), 100); // Max 100 MAD
  };

  const handleBroadcast = async () => {
    await api.post('/flash', {
      dealId: deal.id,
      radius,
      duration
    });
    Alert.alert('Deal Deployed!', `${userCount} users notified`);
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Limited-Time Broadcast</Text>
      
      <RadiusSelector
        value={radius}
        onChange={setRadius}
        options={[100, 300, 500]}
      />

      <DurationSelector
        value={duration}
        onChange={setDuration}
        options={[30, 60, 120]}
      />

      <View style={styles.preview}>
        <Text style={styles.reach}>
          Reach: <Text style={styles.highlight}>{userCount} users</Text>
        </Text>
        <Text style={styles.cost}>
          Cost: <Text style={styles.highlight}>{cost} MAD</Text>
        </Text>
      </View>

      <Button
        title={`Deploy Deal (${cost} MAD)`}
        onPress={handleBroadcast}
        disabled={userCount === 0}
      />
    </View>
  );
};
```

---

### 6. Analytics Dashboard (Pro Feature)

Weekly performance metrics and insights.

#### Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| AD-01 | Total redemptions (this week) | P1 |
| AD-02 | Total deal views | P1 |
| AD-03 | Conversion rate | P1 |
| AD-04 | Peak hours chart | P1 |
| AD-05 | Comparison to previous week | P2 |
| AD-06 | Heatmap playback (animated history) | P3 |

#### Dashboard Component

```typescript
const AnalyticsDashboard = () => {
  const { data, isLoading } = useQuery('analytics', fetchAnalytics);

  if (isLoading) return <LoadingSpinner />;

  return (
    <ScrollView style={styles.container}>
      <StatsGroup>
        <StatCard
          label="Redemptions"
          value={data.redemptions}
          change={data.redemptionsChange}
        />
        <StatCard
          label="Deal Views"
          value={data.views}
          change={data.viewsChange}
        />
        <StatCard
          label="Conversion"
          value={`${data.conversionRate}%`}
          change={data.conversionChange}
        />
        <StatCard
          label="Limited-Time ROI"
          value={`${data.limitedTimeRoi}x`}
        />
      </StatsGroup>

      <Section title="Peak Hours">
        <HourlyChart data={data.hourlyActivity} />
      </Section>

      <Section title="Top Performing Deals">
        <DealList deals={data.topDeals} />
      </Section>
    </ScrollView>
  );
};
```

---

## Screen Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    SELLER APP NAVIGATION                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐                                           │
│   │   Splash    │                                           │
│   └──────┬──────┘                                           │
│          │                                                  │
│          ▼                                                  │
│   ┌─────────────┐     ┌─────────────┐                       │
│   │    Auth     │────▶│   Store     │ (first time only)     │
│   │   (OTP)     │     │   Setup     │                       │
│   └──────┬──────┘     └──────┬──────┘                       │
│          │                   │                              │
│          ▼                   ▼                              │
│   ┌─────────────────────────────────────────────────┐       │
│   │              BOTTOM TAB NAVIGATOR               │       │
│   ├─────────────┬─────────────┬─────────────────────┤       │
│   │   Heatmap   │    Scan     │      Deals          │       │
│   │   (Home)    │   (QR)      │    (Manage)         │       │
│   └──────┬──────┴──────┬──────┴──────┬──────────────┘       │
│          │             │             │                      │
│          ▼             ▼             ▼                      │
│   ┌───────────┐ ┌───────────┐ ┌───────────┐                 │
│   │ Limited   │ │  Result   │ │  Create   │                 │
│   │  Modal    │ │  Screen   │ │  Deal     │                 │
│   └───────────┘ └───────────┘ └───────────┘                 │
│                                                             │
│   Additional screens:                                       │
│   ├── Settings                                              │
│   ├── Analytics (Pro)                                       │
│   ├── Subscription Management                               │
│   └── Support                                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## API Dependencies

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/otp/send` | POST | Send OTP to phone |
| `/auth/otp/verify` | POST | Verify OTP, return token |
| `/stores` | POST | Create store profile |
| `/stores/:id` | PUT | Update store profile |
| `/map/density` | GET | Fetch heatmap GeoJSON |
| `/map/density/count` | GET | Get user count in radius |
| `/deals` | GET | List merchant's deals |
| `/deals` | POST | Create new deal |
| `/deals/:id` | PUT | Update deal |
| `/deals/:id` | DELETE | Delete deal |
| `/redemption/verify` | POST | Verify QR token |
| `/flash` | POST | Create flash broadcast |
| `/analytics` | GET | Fetch dashboard data |

---

## Offline Behavior

| Feature | Offline Behavior |
|---------|------------------|
| Heatmap | Show last cached data with "Offline" badge |
| QR Scan | Queue verification, process when online |
| Deals | View cached deals, queue changes |
| Limited-time broadcast | Disabled (requires real-time data) |

---

## Performance Requirements

| Metric | Target |
|--------|--------|
| App launch to heatmap | < 2 seconds |
| Heatmap refresh | < 500ms |
| QR scan to result | < 1 second |
| Memory usage | < 150MB |
| Battery impact | < 3% per hour active use |

---

## Security Considerations

- Store location stored server-side only
- QR validation requires network (no offline bypass)
- Heatmap data anonymized (no individual user data)
- Pro features gated by subscription status check
- Rate limiting on limited-time broadcasts

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Daily active merchants | 100 (Month 3) |
| QR scans per merchant per week | > 5 |
| Limited-time broadcasts per Pro merchant | > 2/month |
| App rating | > 4.5 stars |
| Crash-free sessions | > 99.5% |

---

*"Your store, your traffic, your data."*

— Frictionless Product Team

## Related Documents

**Dependencies**
- STRAT-01: Section 2
- BRAND-01: Section 2
- DES-01: Section 1

**Related Specs**
- TECH-06: Section 2
- OPS-01: Section 3
- DATA-01: Section 2
- THREAD-02: Section 3
- THREAD-03: Section 3

**Implementation Guides**
- GUIDE-02: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Product Lead | Updated terminology and broadcast naming |
| 1.1 | 2026-01-30 | Product Lead | Added data dictionary and thread references |
| 1.0 | 2026-01-30 | Product Lead | Standardized metadata and cross-references |
