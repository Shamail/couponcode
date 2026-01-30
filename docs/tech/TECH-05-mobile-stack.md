---
document_id: TECH-05
version: 1.2
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-00
  - DES-01
related_documents:
  - PRD-01
  - PRD-02
  - DES-03
  - THREAD-03
---

# TECH-05: Mobile Stack Architecture

## Executive Summary

This document defines the mobile engineering standards for the Buyer and Seller apps, including project structure, Mapbox setup, and API client patterns. The goal is a shared, maintainable codebase with consistent UX across apps.

It is the reference for mobile setup, dependencies, and shared components.

> Frictionless Mobile Apps — Expo + Mapbox + TanStack Query

## Overview

This document defines the mobile engineering standards for both the **Buyer App** and **Seller App (Shadow App)**. Both apps share a common codebase structure using Expo with React Native.

---

## 1. Project Structure

```
apps/
├── buyer/                    # Buyer mobile app
│   ├── app/                  # Expo Router file-based routing
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── app.config.ts
├── seller/                   # Seller mobile app (Shadow App)
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   └── app.config.ts
└── shared/                   # Shared code between apps
    ├── api/                  # API client & TanStack Query hooks
    ├── components/           # Shared UI components
    ├── constants/
    └── types/
```

---

## 2. Mapbox Setup

### 2.1 Installation

```bash
npx expo install @rnmapbox/maps
```

### 2.2 Environment Configuration

Create `.env` files for each app:

```bash
# apps/buyer/.env
EXPO_PUBLIC_MAPBOX_ACCESS_TOKEN=pk.your_public_token_here
EXPO_PUBLIC_API_URL=https://api.frictionless.ma

# apps/seller/.env
EXPO_PUBLIC_MAPBOX_ACCESS_TOKEN=pk.your_public_token_here
EXPO_PUBLIC_API_URL=https://api.frictionless.ma
EXPO_PUBLIC_MAPBOX_STYLE_URL=mapbox://styles/frictionless/dark-heatmap
```

### 2.3 Mapbox Provider Setup

```typescript
// shared/lib/mapbox.ts
import Mapbox from '@rnmapbox/maps';

const MAPBOX_ACCESS_TOKEN = process.env.EXPO_PUBLIC_MAPBOX_ACCESS_TOKEN;

if (!MAPBOX_ACCESS_TOKEN) {
  throw new Error('MAPBOX_ACCESS_TOKEN is not defined in environment');
}

Mapbox.setAccessToken(MAPBOX_ACCESS_TOKEN);

// Disable telemetry for privacy
Mapbox.setTelemetryEnabled(false);

export { Mapbox };
```

### 2.4 App Configuration (app.config.ts)

```typescript
// apps/buyer/app.config.ts
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: 'Frictionless',
  slug: 'frictionless-buyer',
  version: '1.0.0',
  orientation: 'portrait',
  icon: './assets/icon.png',
  scheme: 'frictionless',
  plugins: [
    [
      '@rnmapbox/maps',
      {
        RNMapboxMapsDownloadToken: process.env.MAPBOX_DOWNLOAD_TOKEN,
        RNMapboxMapsVersion: '11.0.0',
      },
    ],
    [
      'expo-location',
      {
        locationAlwaysAndWhenInUsePermission:
          'Frictionless uses your location to show nearby deals and offers.',
        locationAlwaysPermission:
          'Frictionless uses your location to notify you of nearby deals.',
        locationWhenInUsePermission:
          'Frictionless uses your location to show nearby deals.',
        isAndroidBackgroundLocationEnabled: false,
        isAndroidForegroundServiceEnabled: true,
      },
    ],
    [
      'expo-camera',
      {
        cameraPermission: 'Frictionless needs camera access to scan QR codes for redemption.',
      },
    ],
  ],
  ios: {
    supportsTablet: false,
    bundleIdentifier: 'ma.frictionless.buyer',
    infoPlist: {
      NSLocationWhenInUseUsageDescription:
        'Frictionless uses your location to show nearby deals.',
      NSLocationAlwaysAndWhenInUseUsageDescription:
        'Frictionless uses your location to show nearby deals and offers.',
      UIBackgroundModes: ['location'],
    },
  },
  android: {
    package: 'ma.frictionless.buyer',
    permissions: [
      'ACCESS_COARSE_LOCATION',
      'ACCESS_FINE_LOCATION',
      'FOREGROUND_SERVICE',
      'FOREGROUND_SERVICE_LOCATION',
      'CAMERA',
    ],
  },
});
```

---

## 3. Data Fetching with TanStack Query

### 3.1 Installation

```bash
npx expo install @tanstack/react-query
```

### 3.2 Query Client Setup

```typescript
// shared/lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Stale time: 20 seconds (data considered fresh)
      staleTime: 20 * 1000,
      // Cache time: 5 minutes
      gcTime: 5 * 60 * 1000,
      // Retry failed requests 2 times
      retry: 2,
      // Refetch on window focus for real-time feel
      refetchOnWindowFocus: true,
    },
    mutations: {
      // Retry mutations once
      retry: 1,
    },
  },
});
```

### 3.3 API Client

```typescript
// shared/api/client.ts
const API_URL = process.env.EXPO_PUBLIC_API_URL;

if (!API_URL) {
  throw new Error('API_URL is not defined in environment');
}

type RequestOptions = {
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
  body?: unknown;
  headers?: Record<string, string>;
  abort?: AbortController['signal'];
};

class ApiError extends Error {
  constructor(
    public status: number,
    public statusText: string,
    public data?: unknown
  ) {
    super(`API Error: ${status} ${statusText}`);
    this.name = 'ApiError';
  }
}

export async function apiClient<T>(
  endpoint: string,
  options: RequestOptions = {}
): Promise<T> {
  const { method = 'GET', body, headers = {}, abort } = options;

  const response = await fetch(`${API_URL}${endpoint}`, {
    method,
    headers: {
      'Content-Type': 'application/json',
      ...headers,
    },
    body: body ? JSON.stringify(body) : undefined,
    signal: abort,
  });

  if (!response.ok) {
    const data = await response.json().catch(() => null);
    throw new ApiError(response.status, response.statusText, data);
  }

  return response.json();
}
```

### 3.4 Heatmap Query Hook (Seller App)

```typescript
// apps/seller/hooks/useHeatmapData.ts
import { useQuery } from '@tanstack/react-query';
import { apiClient } from '@shared/api/client';

type HeatmapPoint = {
  latitude: number;
  longitude: number;
  weight: number; // User density weight
};

type HeatmapResponse = {
  points: HeatmapPoint[];
  total_users: number;
  updated_at: string;
};

type UseHeatmapOptions = {
  latitude: number;
  longitude: number;
  radius_km?: number;
  enabled?: boolean;
};

/**
 * Fetches heatmap data for the seller's nearby area.
 * Auto-refreshes every 30 seconds.
 */
export function useHeatmapData({
  latitude,
  longitude,
  radius_km = 2,
  enabled = true,
}: UseHeatmapOptions) {
  return useQuery({
    queryKey: ['heatmap', { latitude, longitude, radius_km }],
    queryFn: ({ signal }) =>
      apiClient<HeatmapResponse>(
        `/v1/heatmap?lat=${latitude}&lng=${longitude}&radius=${radius_km}`,
        { signal }
      ),
    enabled,
    // ⚡ CRITICAL: 30-second polling interval
    refetchInterval: 30_000,
    // Keep polling even when tab/app is in background
    refetchIntervalInBackground: false,
    // Consider data stale immediately for real-time feel
    staleTime: 0,
  });
}
```

### 3.5 Location Update Mutation (Buyer App)

```typescript
// apps/buyer/hooks/useLocationUpdate.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '@shared/api/client';

type LocationUpdate = {
  latitude: number;
  longitude: number;
  accuracy: number;
  timestamp: number;
};

type LocationResponse = {
  success: boolean;
  nearby_offers: number;
};

/**
 * Sends location updates to the API.
 * Called when foreground location changes.
 */
export function useLocationUpdate() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (location: LocationUpdate) =>
      apiClient<LocationResponse>('/v1/location', {
        method: 'POST',
        body: location,
      }),
    onSuccess: (data) => {
      // Invalidate nearby offers when location changes
      if (data.nearby_offers > 0) {
        queryClient.invalidateQueries({ queryKey: ['nearby-offers'] });
      }
    },
    // Silent retry - don't block UX for location updates
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 10000),
  });
}
```

### 3.6 QR Redemption Mutation

```typescript
// shared/api/hooks/useRedemption.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { apiClient } from '@shared/api/client';

type RedemptionRequest = {
  qr_payload: string;
  color_code: string;
  scanned_at: number;
};

type RedemptionResponse = {
  success: boolean;
  offer_id: string;
  offer_title: string;
  discount_value: string;
  message: string;
};

export function useRedemption() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: RedemptionRequest) =>
      apiClient<RedemptionResponse>('/v1/redeem', {
        method: 'POST',
        body: data,
      }),
    onSuccess: () => {
      // Invalidate user's redemption history
      queryClient.invalidateQueries({ queryKey: ['redemptions'] });
      queryClient.invalidateQueries({ queryKey: ['wallet'] });
    },
    // No retry for redemptions - single attempt only
    retry: false,
  });
}
```

---

## 4. Location Permissions

### 4.1 Permission Hook

```typescript
// shared/hooks/useLocationPermission.ts
import { useState, useEffect, useCallback } from 'react';
import * as Location from 'expo-location';
import { Alert, Linking, Platform } from 'react-native';

type PermissionStatus = 'undetermined' | 'granted' | 'denied' | 'restricted';

type UseLocationPermissionResult = {
  status: PermissionStatus;
  isLoading: boolean;
  requestPermission: () => Promise<boolean>;
  openSettings: () => void;
};

export function useLocationPermission(): UseLocationPermissionResult {
  const [status, setStatus] = useState<PermissionStatus>('undetermined');
  const [isLoading, setIsLoading] = useState(true);

  // Check current permission status on mount
  useEffect(() => {
    checkPermission();
  }, []);

  const checkPermission = async () => {
    setIsLoading(true);
    try {
      const { status: foregroundStatus } =
        await Location.getForegroundPermissionsAsync();
      setStatus(mapExpoStatus(foregroundStatus));
    } catch (error) {
      console.error('Error checking location permission:', error);
      setStatus('denied');
    } finally {
      setIsLoading(false);
    }
  };

  const requestPermission = useCallback(async (): Promise<boolean> => {
    setIsLoading(true);
    try {
      // First request foreground permission
      const { status: foregroundStatus } =
        await Location.requestForegroundPermissionsAsync();

      if (foregroundStatus !== 'granted') {
        setStatus(mapExpoStatus(foregroundStatus));
        showPermissionDeniedAlert();
        return false;
      }

      setStatus('granted');
      return true;
    } catch (error) {
      console.error('Error requesting location permission:', error);
      setStatus('denied');
      return false;
    } finally {
      setIsLoading(false);
    }
  }, []);

  const openSettings = useCallback(() => {
    if (Platform.OS === 'ios') {
      Linking.openURL('app-settings:');
    } else {
      Linking.openSettings();
    }
  }, []);

  return {
    status,
    isLoading,
    requestPermission,
    openSettings,
  };
}

function mapExpoStatus(
  status: Location.PermissionStatus
): PermissionStatus {
  switch (status) {
    case Location.PermissionStatus.GRANTED:
      return 'granted';
    case Location.PermissionStatus.DENIED:
      return 'denied';
    case Location.PermissionStatus.UNDETERMINED:
      return 'undetermined';
    default:
      return 'restricted';
  }
}

function showPermissionDeniedAlert() {
  Alert.alert(
    'Location Access Required',
    'Frictionless needs your location to show nearby deals. Please enable location access in Settings.',
    [
      { text: 'Cancel', style: 'cancel' },
      { text: 'Open Settings', onPress: () => Linking.openSettings() },
    ]
  );
}
```

### 4.2 Permission Flow Component

```typescript
// shared/components/LocationPermissionGate.tsx
import React from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  ActivityIndicator,
} from 'react-native';
import { useLocationPermission } from '../hooks/useLocationPermission';

type Props = {
  children: React.ReactNode;
};

export function LocationPermissionGate({ children }: Props) {
  const { status, isLoading, requestPermission, openSettings } =
    useLocationPermission();

  if (isLoading) {
    return (
      <View style={styles.container}>
        <ActivityIndicator size="large" color="#6366F1" />
        <Text style={styles.text}>Checking permissions...</Text>
      </View>
    );
  }

  if (status === 'granted') {
    return <>{children}</>;
  }

  if (status === 'undetermined') {
    return (
      <View style={styles.container}>
        <Text style={styles.title}>Enable Location</Text>
        <Text style={styles.description}>
          Frictionless uses your location to show you amazing deals nearby.
          Your location is only used while the app is open.
        </Text>
        <TouchableOpacity style={styles.button} onPress={requestPermission}>
          <Text style={styles.buttonText}>Enable Location</Text>
        </TouchableOpacity>
      </View>
    );
  }

  // Permission denied or restricted
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Location Required</Text>
      <Text style={styles.description}>
        We can't show you nearby deals without location access.
        Please enable it in your device settings.
      </Text>
      <TouchableOpacity style={styles.button} onPress={openSettings}>
        <Text style={styles.buttonText}>Open Settings</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
    backgroundColor: '#0F0F0F',
  },
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: '#FFFFFF',
    marginBottom: 16,
    textAlign: 'center',
  },
  description: {
    fontSize: 16,
    color: '#A1A1AA',
    textAlign: 'center',
    marginBottom: 32,
    lineHeight: 24,
  },
  text: {
    fontSize: 16,
    color: '#A1A1AA',
    marginTop: 16,
  },
  button: {
    backgroundColor: '#6366F1',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 12,
  },
  buttonText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#FFFFFF',
  },
});
```

### 4.3 Foreground Location Tracking (Buyer App)

```typescript
// apps/buyer/hooks/useLocationTracking.ts
import { useEffect, useRef, useCallback } from 'react';
import * as Location from 'expo-location';
import { AppState, AppStateStatus } from 'react-native';
import { useLocationUpdate } from './useLocationUpdate';

type UseLocationTrackingOptions = {
  enabled: boolean;
  minDistance?: number; // Minimum distance in meters before update
  minInterval?: number; // Minimum time in ms between updates
};

export function useLocationTracking({
  enabled,
  minDistance = 50, // 50 meters
  minInterval = 10_000, // 10 seconds
}: UseLocationTrackingOptions) {
  const subscriptionRef = useRef<Location.LocationSubscription | null>(null);
  const lastUpdateRef = useRef<number>(0);
  const { mutate: updateLocation } = useLocationUpdate();

  const startTracking = useCallback(async () => {
    if (subscriptionRef.current) {
      return; // Already tracking
    }

    subscriptionRef.current = await Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.Balanced,
        distanceInterval: minDistance,
        timeInterval: minInterval,
      },
      (location) => {
        const now = Date.now();
        if (now - lastUpdateRef.current < minInterval) {
          return; // Throttle updates
        }
        lastUpdateRef.current = now;

        updateLocation({
          latitude: location.coords.latitude,
          longitude: location.coords.longitude,
          accuracy: location.coords.accuracy ?? 0,
          timestamp: location.timestamp,
        });
      }
    );
  }, [minDistance, minInterval, updateLocation]);

  const stopTracking = useCallback(() => {
    if (subscriptionRef.current) {
      subscriptionRef.current.remove();
      subscriptionRef.current = null;
    }
  }, []);

  // Handle app state changes (foreground/background)
  useEffect(() => {
    const handleAppStateChange = (state: AppStateStatus) => {
      if (state === 'active' && enabled) {
        startTracking();
      } else {
        stopTracking();
      }
    };

    const subscription = AppState.addEventListener('change', handleAppStateChange);

    // Start tracking if enabled and app is active
    if (enabled && AppState.currentState === 'active') {
      startTracking();
    }

    return () => {
      subscription.remove();
      stopTracking();
    };
  }, [enabled, startTracking, stopTracking]);
}
```

---

## 5. Provider Setup

```typescript
// apps/buyer/app/_layout.tsx (and apps/seller/app/_layout.tsx)
import { QueryClientProvider } from '@tanstack/react-query';
import { Stack } from 'expo-router';
import { queryClient } from '@shared/lib/query-client';
import '@shared/lib/mapbox'; // Initialize Mapbox

export default function RootLayout() {
  return (
    <QueryClientProvider client={queryClient}>
      <Stack
        screenOptions={{
          headerShown: false,
          contentStyle: { backgroundColor: '#0F0F0F' },
        }}
      />
    </QueryClientProvider>
  );
}
```

---

## 6. Environment Variables Reference

| Variable | App | Description |
|----------|-----|-------------|
| `EXPO_PUBLIC_MAPBOX_ACCESS_TOKEN` | Both | Public Mapbox token for map rendering |
| `EXPO_PUBLIC_API_URL` | Both | Backend API base URL |
| `EXPO_PUBLIC_MAPBOX_STYLE_URL` | Seller | Custom dark heatmap style URL |
| `MAPBOX_DOWNLOAD_TOKEN` | Build | Secret token for Mapbox SDK download (CI only) |

---

## 7. Build & Development

### Development

```bash
# Start Buyer app
cd apps/buyer && npx expo start

# Start Seller app
cd apps/seller && npx expo start
```

### Building with EAS

```bash
# Build for iOS
eas build --platform ios --profile production

# Build for Android
eas build --platform android --profile production
```

### EAS Configuration

```json
// apps/buyer/eas.json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal"
    },
    "production": {}
  },
  "submit": {
    "production": {}
  }
}
```

---

## 8. Error Handling Best Practices

```typescript
// shared/components/QueryErrorBoundary.tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

type Props = {
  children: React.ReactNode;
};

export function QueryErrorBoundary({ children }: Props) {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <View style={styles.container}>
              <Text style={styles.title}>Something went wrong</Text>
              <Text style={styles.message}>{error.message}</Text>
              <TouchableOpacity
                style={styles.button}
                onPress={resetErrorBoundary}
              >
                <Text style={styles.buttonText}>Try Again</Text>
              </TouchableOpacity>
            </View>
          )}
        >
          {children}
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 32,
    backgroundColor: '#0F0F0F',
  },
  title: {
    fontSize: 20,
    fontWeight: '700',
    color: '#FFFFFF',
    marginBottom: 8,
  },
  message: {
    fontSize: 14,
    color: '#A1A1AA',
    textAlign: 'center',
    marginBottom: 24,
  },
  button: {
    backgroundColor: '#6366F1',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 8,
  },
  buttonText: {
    fontSize: 14,
    fontWeight: '600',
    color: '#FFFFFF',
  },
});
```

## 9. Android Fragmentation & Device Strategy

### 9.1 Minimum API Level

- **Minimum supported:** API 23 (Android 6.0)
- **Rationale:** Captures 95%+ of Morocco Android market share while keeping runtime stable

### 9.2 Device Testing Matrix

| Priority | Device | Test Focus |
| --- | --- | --- |
| P0 | Samsung Galaxy A series | Full regression |
| P0 | Xiaomi Redmi series | Battery + location |
| P1 | Infinix Hot/Note | Memory constraints |
| P1 | Tecno Spark | Low RAM (<3GB) |
| P2 | iPhone SE/XR | iOS baseline |

### 9.3 RAM-Based Feature Gating

| RAM | Polling Interval | Cached Deals |
| --- | --- | --- |
| <2GB | 60s | 20 |
| 2-4GB | 30s | 50 |
| >4GB | 20s | 100 |

## Related Documents

**Dependencies**
- TECH-00: Section 2
- DES-01: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 3
- DES-03: Section 2
- THREAD-03: Section 3

**Implementation Guides**
- GUIDE-01: Section 2
- GUIDE-02: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Engineering Lead | Updated request abort handling |
| 1.1 | 2026-01-30 | Engineering Lead | Added Android fragmentation strategy |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
