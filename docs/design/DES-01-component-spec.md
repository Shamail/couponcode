---
document_id: DES-01
version: 1.2
status: Final
priority: P1
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - BRAND-01
related_documents:
  - PRD-01
  - PRD-02
  - DES-02
  - DES-03
  - TECH-05
  - THREAD-02
---

# DES-01: Component Specifications

## Executive Summary

This document defines the visual and behavioral specifications for core UI components in the Frictionless apps. It sets standards for map interactions, deal cards, and redemption flows to ensure a consistent, low-friction experience.

All product and engineering teams should reference this spec before implementing UI changes.

> Frictionless UI Component Design System

## Overview

This document defines the visual and behavioral specifications for core UI components in the Frictionless mobile apps. All components follow a **"Zero Friction"** design philosophy with dark mode as the default.

---

## 1. Map Component

### 1.1 Base Map Configuration

```typescript
// shared/components/Map/FrictionlessMap.tsx
import React, { useRef } from 'react';
import { StyleSheet, View } from 'react-native';
import Mapbox, { Camera, MapView, UserLocation } from '@rnmapbox/maps';

// Dark Mode Style URL (Mapbox Studio)
const DARK_STYLE_URL = 'mapbox://styles/mapbox/dark-v11';
// Custom Frictionless style (if available)
const FRICTIONLESS_STYLE_URL =
  process.env.EXPO_PUBLIC_MAPBOX_STYLE_URL || DARK_STYLE_URL;

type FrictionlessMapProps = {
  children?: React.ReactNode;
  initialCoordinates?: [number, number]; // [lng, lat]
  initialZoom?: number;
  showUserLocation?: boolean;
  onUserLocationUpdate?: (location: { latitude: number; longitude: number }) => void;
};

export function FrictionlessMap({
  children,
  initialCoordinates,
  initialZoom = 14,
  showUserLocation = true,
  onUserLocationUpdate,
}: FrictionlessMapProps) {
  const cameraRef = useRef<Camera>(null);

  return (
    <View style={styles.container}>
      <MapView
        style={styles.map}
        styleURL={FRICTIONLESS_STYLE_URL}
        logoEnabled={false}
        attributionEnabled={false}
        compassEnabled={false}
        scaleBarEnabled={false}
      >
        <Camera
          ref={cameraRef}
          defaultSettings={{
            centerCoordinate: initialCoordinates,
            zoomLevel: initialZoom,
          }}
          followUserLocation={showUserLocation && !initialCoordinates}
          followZoomLevel={initialZoom}
        />

        {showUserLocation && (
          <UserLocation
            visible
            showsUserHeadingIndicator
            onUpdate={(location) => {
              if (onUserLocationUpdate && location.coords) {
                onUserLocationUpdate({
                  latitude: location.coords.latitude,
                  longitude: location.coords.longitude,
                });
              }
            }}
            renderMode="native"
          />
        )}

        {children}
      </MapView>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#121212',
  },
  map: {
    flex: 1,
  },
});
```

### 1.2 Custom Dark Theme Specification

For Mapbox Studio custom style:

| Layer | Color | Opacity |
|-------|-------|---------|
| Background | `#121212` | 100% |
| Land | `#1E1E1E` | 100% |
| Water | `#0B1220` | 100% |
| Roads - Primary | `#2D2D2D` | 100% |
| Roads - Secondary | `#252525` | 80% |
| Buildings | `#232323` | 60% |
| Labels - Primary | `#F9FAFB` | 80% |
| Labels - Secondary | `#9CA3AF` | 60% |
| POI Icons | `#4F46E5` | 100% |

### 1.3 Heatmap Layer (Seller App)

```typescript
// apps/seller/components/HeatmapLayer.tsx
import React from 'react';
import { HeatmapLayer, ShapeSource } from '@rnmapbox/maps';

type HeatmapPoint = {
  latitude: number;
  longitude: number;
  weight: number;
};

type Props = {
  points: HeatmapPoint[];
};

export function UserHeatmapLayer({ points }: Props) {
  const geoJson: GeoJSON.FeatureCollection = {
    type: 'FeatureCollection',
    features: points.map((point) => ({
      type: 'Feature',
      properties: {
        weight: point.weight,
      },
      geometry: {
        type: 'Point',
        coordinates: [point.longitude, point.latitude],
      },
    })),
  };

  return (
    <ShapeSource id="heatmap-source" shape={geoJson}>
      <HeatmapLayer
        id="heatmap-layer"
        sourceID="heatmap-source"
        style={{
          // Increase weight based on zoom level
          heatmapWeight: [
            'interpolate',
            ['linear'],
            ['get', 'weight'],
            0, 0,
            1, 1,
          ],
          // Increase intensity as zoom level increases
          heatmapIntensity: [
            'interpolate',
            ['linear'],
            ['zoom'],
            0, 1,
            15, 3,
          ],
          // Color ramp: transparent → blue → amber → red
          heatmapColor: [
            'interpolate',
            ['linear'],
            ['heatmap-density'],
            0, 'rgba(0, 0, 0, 0)',
            0.2, 'rgba(59, 130, 246, 0.35)',  // Blue
            0.4, 'rgba(245, 158, 11, 0.5)',   // Amber
            0.6, 'rgba(245, 158, 11, 0.7)',   // Amber (denser)
            0.8, 'rgba(239, 68, 68, 0.85)',   // Red
            1, 'rgba(239, 68, 68, 1)',        // Red (hot)
          ],
          // Adjust radius by zoom level
          heatmapRadius: [
            'interpolate',
            ['linear'],
            ['zoom'],
            0, 2,
            15, 30,
          ],
          // Fade out at high zoom levels
          heatmapOpacity: [
            'interpolate',
            ['linear'],
            ['zoom'],
            14, 1,
            18, 0.6,
          ],
        }}
      />
    </ShapeSource>
  );
}
```

### 1.4 Offer Markers (Buyer App)

```typescript
// apps/buyer/components/OfferMarker.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { MarkerView } from '@rnmapbox/maps';

type Props = {
  id: string;
  coordinate: [number, number]; // [lng, lat]
  discount: string;
  onPress?: () => void;
};

export function OfferMarker({ id, coordinate, discount, onPress }: Props) {
  return (
    <MarkerView
      id={id}
      coordinate={coordinate}
      allowOverlap={false}
      anchor={{ x: 0.5, y: 1 }}
    >
      <View style={styles.container} onTouchEnd={onPress}>
        <View style={styles.bubble}>
          <Text style={styles.discount}>{discount}</Text>
        </View>
        <View style={styles.arrow} />
        <View style={styles.pulse} />
      </View>
    </MarkerView>
  );
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
  bubble: {
    backgroundColor: '#F59E0B',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 20,
    shadowColor: '#000000',
    shadowOffset: { width: 0, height: 6 },
    shadowOpacity: 0.25,
    shadowRadius: 10,
    elevation: 6,
  },
  discount: {
    color: '#FFFFFF',
    fontSize: 14,
    fontWeight: '700',
  },
  arrow: {
    width: 0,
    height: 0,
    borderLeftWidth: 8,
    borderRightWidth: 8,
    borderTopWidth: 8,
    borderLeftColor: 'transparent',
    borderRightColor: 'transparent',
    borderTopColor: '#F59E0B',
    marginTop: -1,
  },
  pulse: {
    position: 'absolute',
    bottom: -4,
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: '#F59E0B',
    opacity: 0.4,
  },
});
```

---

### 1.5 Home Screen Overlays

These components sit above the map on the Home screen.

**FloatingHeader**
- Height: 44-52px, left-aligned location label
- Right-aligned notification bell
- Background: `#1E1E1E` at ~85% opacity (optional blur)

**FloatingSearchBar**
- Pill shape, min height 44px
- Placeholder: "Search deals..."
- Left icon + input, right clear button
- Background: `#1E1E1E`, border `#2D2D2D`

**Limited-Time Deal Toast**
- Pill banner anchored above the tab bar
- Example: "⚡ 3 limited-time deals nearby"
- Uses `colors.map.flashDeal` for icon accent

### 1.6 Bottom Tab Bar

**Tabs:** Home, Explore, Wallet, Profile, Menu
- Height: 64px + safe area
- Active icon/text: `colors.brand.primary`
- Inactive: `colors.text.tertiary`
- Background: `#1E1E1E`

### 1.7 Lite Mode (Low-Bandwidth)

Lite Mode preserves core functionality when connectivity or device constraints are poor.

**Activation Criteria**
- Network RTT > 500ms
- `Save-Data` header present
- `effectiveType === '2g'` or `slow-2g`

**Lite Mode Constraints**

| Asset | Normal | Lite |
| --- | --- | --- |
| Map tiles | Mapbox vector | Disabled (list view) |
| Deal images | 200KB | 50KB max |
| Animations | Enabled | Disabled |
| Polling | 30s | 60s |

## 2. Deal Detail Components

### 2.1 Deal Bottom Sheet

- Library: `@gorhom/bottom-sheet`
- Snap points: 40%, 70%, 92%
- Background: `colors.background.elevated` (`#2D2D2D`)
- Handle: 32px x 4px, radius 2px, color `#6B7280`
- Use native spring for open/close
### 2.2 Deal Value Card

- Purpose: highlight the discount in a single glance
- Background: `#1E1E1E`, border `#2D2D2D`
- Padding: 16px, radius: 12px
- Uses `colors.map.dealMarker` for icon accent

### 2.3 Claim Button

- Minimum height: 52px
- Full width in the bottom sheet
- Primary color: `colors.brand.primary`
- Label: "Claim Deal (Free)" or "Claim Deal"

### 2.4 Wallet Deal Card

- Used in "My Deals" list and History
- Primary action: "Use Now"
- Status chips: Ready / Redeemed / Expired

## 3. Scanner Component

### 3.1 QR Scanner with Square Viewport

```typescript
// shared/components/Scanner/QRScanner.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  StyleSheet,
  Dimensions,
  Animated,
  Easing,
} from 'react-native';
import { CameraView, useCameraPermissions } from 'expo-camera';
import { BlurView } from 'expo-blur';

const { width: SCREEN_WIDTH } = Dimensions.get('window');
const VIEWPORT_SIZE = SCREEN_WIDTH * 0.7; // 70% of screen width
const CORNER_SIZE = 24;
const CORNER_WIDTH = 4;

type Props = {
  onScan: (data: string) => void;
  enabled?: boolean;
};

export function QRScanner({ onScan, enabled = true }: Props) {
  const [permission, requestPermission] = useCameraPermissions();
  const [scanned, setScanned] = useState(false);
  const scanLineAnim = new Animated.Value(0);

  // Animate scan line
  useEffect(() => {
    if (enabled && !scanned) {
      Animated.loop(
        Animated.sequence([
          Animated.timing(scanLineAnim, {
            toValue: 1,
            duration: 2000,
            easing: Easing.linear,
            useNativeDriver: true,
          }),
          Animated.timing(scanLineAnim, {
            toValue: 0,
            duration: 2000,
            easing: Easing.linear,
            useNativeDriver: true,
          }),
        ])
      ).start();
    }
  }, [enabled, scanned]);

  const handleBarCodeScanned = ({ data }: { data: string }) => {
    if (scanned || !enabled) return;
    setScanned(true);
    onScan(data);
  };

  if (!permission) {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>Requesting camera permission...</Text>
      </View>
    );
  }

  if (!permission.granted) {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>Camera access required</Text>
        <Text style={styles.subtext} onPress={requestPermission}>
          Tap to enable camera
        </Text>
      </View>
    );
  }

  const scanLineTranslate = scanLineAnim.interpolate({
    inputRange: [0, 1],
    outputRange: [0, VIEWPORT_SIZE - 4],
  });

  return (
    <View style={styles.container}>
      <CameraView
        style={StyleSheet.absoluteFill}
        facing="back"
        barcodeScannerSettings={{
          barcodeTypes: ['qr'],
        }}
        onBarcodeScanned={enabled ? handleBarCodeScanned : undefined}
      />

      {/* Dark overlay with transparent viewport */}
      <View style={styles.overlay}>
        {/* Top overlay */}
        <View style={styles.overlayTop} />

        {/* Middle row with viewport */}
        <View style={styles.overlayMiddle}>
          <View style={styles.overlaySide} />

          {/* Transparent viewport */}
          <View style={styles.viewport}>
            {/* Corner indicators */}
            <View style={[styles.corner, styles.cornerTL]} />
            <View style={[styles.corner, styles.cornerTR]} />
            <View style={[styles.corner, styles.cornerBL]} />
            <View style={[styles.corner, styles.cornerBR]} />

            {/* Scan line */}
            <Animated.View
              style={[
                styles.scanLine,
                { transform: [{ translateY: scanLineTranslate }] },
              ]}
            />
          </View>

          <View style={styles.overlaySide} />
        </View>

        {/* Bottom overlay */}
        <View style={styles.overlayBottom}>
          <Text style={styles.instruction}>
            Align QR code within the frame
          </Text>
        </View>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#000000',
    justifyContent: 'center',
    alignItems: 'center',
  },
  text: {
    color: '#FFFFFF',
    fontSize: 18,
    fontWeight: '600',
  },
  subtext: {
    color: '#4F46E5',
    fontSize: 14,
    marginTop: 8,
  },
  overlay: {
    ...StyleSheet.absoluteFillObject,
  },
  overlayTop: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.7)',
  },
  overlayMiddle: {
    flexDirection: 'row',
  },
  overlaySide: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.7)',
  },
  overlayBottom: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0.7)',
    alignItems: 'center',
    paddingTop: 32,
  },
  viewport: {
    width: VIEWPORT_SIZE,
    height: VIEWPORT_SIZE,
    position: 'relative',
  },
  corner: {
    position: 'absolute',
    width: CORNER_SIZE,
    height: CORNER_SIZE,
    borderColor: '#4F46E5',
  },
  cornerTL: {
    top: 0,
    left: 0,
    borderTopWidth: CORNER_WIDTH,
    borderLeftWidth: CORNER_WIDTH,
    borderTopLeftRadius: 4,
  },
  cornerTR: {
    top: 0,
    right: 0,
    borderTopWidth: CORNER_WIDTH,
    borderRightWidth: CORNER_WIDTH,
    borderTopRightRadius: 4,
  },
  cornerBL: {
    bottom: 0,
    left: 0,
    borderBottomWidth: CORNER_WIDTH,
    borderLeftWidth: CORNER_WIDTH,
    borderBottomLeftRadius: 4,
  },
  cornerBR: {
    bottom: 0,
    right: 0,
    borderBottomWidth: CORNER_WIDTH,
    borderRightWidth: CORNER_WIDTH,
    borderBottomRightRadius: 4,
  },
  scanLine: {
    position: 'absolute',
    left: 8,
    right: 8,
    height: 2,
    backgroundColor: '#4F46E5',
    shadowColor: '#000000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 6,
  },
  instruction: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '500',
    opacity: 0.8,
  },
});
```

### 3.2 SafeColor Verification

SafeColor Verification requires matching a dynamic color code displayed alongside the QR.

```typescript
// shared/components/Scanner/ColorCodeDisplay.tsx
import React, { useEffect, useRef } from 'react';
import { View, Animated, StyleSheet, Easing } from 'react-native';

const COLORS = {
  red: '#EF4444',
  amber: '#F59E0B',
  green: '#10B981',
  blue: '#3B82F6',
};

type ColorCode = keyof typeof COLORS;

type Props = {
  code: ColorCode;
  size?: number;
  pulsing?: boolean;
};

export function ColorCodeDisplay({ code, size = 80, pulsing = true }: Props) {
  const pulseAnim = useRef(new Animated.Value(1)).current;

  useEffect(() => {
    if (pulsing) {
      Animated.loop(
        Animated.sequence([
          Animated.timing(pulseAnim, {
            toValue: 1.05,
            duration: 1500,
            easing: Easing.inOut(Easing.ease),
            useNativeDriver: true,
          }),
          Animated.timing(pulseAnim, {
            toValue: 1,
            duration: 1500,
            easing: Easing.inOut(Easing.ease),
            useNativeDriver: true,
          }),
        ])
      ).start();
    }
  }, [pulsing]);

  const color = COLORS[code];

  return (
    <View style={styles.container}>
      {/* Main circle */}
      <Animated.View
        style={[
          styles.circle,
          {
            width: size,
            height: size,
            backgroundColor: color,
            borderRadius: size / 2,
            transform: [{ scale: pulseAnim }],
          },
        ]}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    justifyContent: 'center',
  },
  circle: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 6 },
    shadowOpacity: 0.2,
    shadowRadius: 10,
    elevation: 4,
  },
});
```

---

## 4. Feedback UI — Redemption Animations

### 4.1 Success Animation

```typescript
// shared/components/Feedback/SuccessAnimation.tsx
import React, { useEffect, useRef } from 'react';
import {
  View,
  Text,
  StyleSheet,
  Animated,
  Easing,
  Dimensions,
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';

const { width: SCREEN_WIDTH, height: SCREEN_HEIGHT } = Dimensions.get('window');

type Props = {
  title: string;
  subtitle?: string;
  discount?: string;
  onComplete?: () => void;
};

export function SuccessAnimation({
  title,
  subtitle,
  discount,
  onComplete,
}: Props) {
  const scaleAnim = useRef(new Animated.Value(0)).current;
  const opacityAnim = useRef(new Animated.Value(0)).current;
  const checkScaleAnim = useRef(new Animated.Value(0)).current;
  const rippleAnim = useRef(new Animated.Value(0)).current;
  const textAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    // Sequence: Background → Ripple → Checkmark → Text
    Animated.sequence([
      // Background fade in
      Animated.parallel([
        Animated.timing(opacityAnim, {
          toValue: 1,
          duration: 200,
          useNativeDriver: true,
        }),
        Animated.spring(scaleAnim, {
          toValue: 1,
          tension: 50,
          friction: 7,
          useNativeDriver: true,
        }),
      ]),
      // Ripple effect
      Animated.timing(rippleAnim, {
        toValue: 1,
        duration: 600,
        easing: Easing.out(Easing.ease),
        useNativeDriver: true,
      }),
      // Checkmark bounce
      Animated.spring(checkScaleAnim, {
        toValue: 1,
        tension: 100,
        friction: 8,
        useNativeDriver: true,
      }),
      // Text fade in
      Animated.timing(textAnim, {
        toValue: 1,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start();

    // Auto-dismiss after 3 seconds
    const timer = setTimeout(() => {
      Animated.timing(opacityAnim, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }).start(() => onComplete?.());
    }, 3000);

    return () => clearTimeout(timer);
  }, []);

  const rippleScale = rippleAnim.interpolate({
    inputRange: [0, 1],
    outputRange: [0, 4],
  });

  const rippleOpacity = rippleAnim.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [0.6, 0.3, 0],
  });

  return (
    <Animated.View
      style={[
        styles.container,
        {
          opacity: opacityAnim,
          transform: [{ scale: scaleAnim }],
        },
      ]}
    >
      {/* Ripple effect */}
      <Animated.View
        style={[
          styles.ripple,
          {
            transform: [{ scale: rippleScale }],
            opacity: rippleOpacity,
          },
        ]}
      />

      {/* Success icon */}
      <Animated.View
        style={[
          styles.iconContainer,
          { transform: [{ scale: checkScaleAnim }] },
        ]}
      >
        <View style={styles.iconCircle}>
          <Ionicons name="checkmark" size={80} color="#FFFFFF" />
        </View>
      </Animated.View>

      {/* Discount badge */}
      {discount && (
        <Animated.View style={[styles.discountBadge, { opacity: textAnim }]}>
          <Text style={styles.discountText}>{discount}</Text>
        </Animated.View>
      )}

      {/* Text */}
      <Animated.View style={[styles.textContainer, { opacity: textAnim }]}>
        <Text style={styles.title}>{title}</Text>
        {subtitle && <Text style={styles.subtitle}>{subtitle}</Text>}
      </Animated.View>

      {/* Confetti particles */}
      <ConfettiParticles />
    </Animated.View>
  );
}

function ConfettiParticles() {
  const particles = Array.from({ length: 20 }, (_, i) => {
    const anim = useRef(new Animated.Value(0)).current;
    const randomX = (Math.random() - 0.5) * SCREEN_WIDTH;
    const randomDelay = Math.random() * 500;
    const randomDuration = 1000 + Math.random() * 500;

    useEffect(() => {
      Animated.timing(anim, {
        toValue: 1,
        duration: randomDuration,
        delay: randomDelay + 400,
        easing: Easing.out(Easing.quad),
        useNativeDriver: true,
      }).start();
    }, []);

    const translateY = anim.interpolate({
      inputRange: [0, 1],
      outputRange: [0, SCREEN_HEIGHT * 0.5],
    });

    const opacity = anim.interpolate({
      inputRange: [0, 0.8, 1],
      outputRange: [1, 1, 0],
    });

    const colors = ['#10B981', '#4F46E5', '#F59E0B', '#3B82F6', '#EF4444'];
    const color = colors[i % colors.length];

    return (
      <Animated.View
        key={i}
        style={[
          styles.particle,
          {
            backgroundColor: color,
            left: SCREEN_WIDTH / 2 + randomX,
            transform: [{ translateY }],
            opacity,
          },
        ]}
      />
    );
  });

  return <>{particles}</>;
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(18, 18, 18, 0.96)',
    justifyContent: 'center',
    alignItems: 'center',
    zIndex: 1000,
  },
  ripple: {
    position: 'absolute',
    width: 120,
    height: 120,
    borderRadius: 60,
    backgroundColor: '#10B981',
  },
  iconContainer: {
    marginBottom: 24,
  },
  iconCircle: {
    width: 140,
    height: 140,
    borderRadius: 70,
    backgroundColor: '#10B981',
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#000000',
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.25,
    shadowRadius: 16,
    elevation: 8,
  },
  discountBadge: {
    backgroundColor: '#FFFFFF',
    paddingHorizontal: 24,
    paddingVertical: 12,
    borderRadius: 30,
    marginBottom: 24,
  },
  discountText: {
    fontSize: 32,
    fontWeight: '800',
    color: '#10B981',
  },
  textContainer: {
    alignItems: 'center',
    paddingHorizontal: 32,
  },
  title: {
    fontSize: 28,
    fontWeight: '700',
    color: '#FFFFFF',
    textAlign: 'center',
    marginBottom: 8,
  },
  subtitle: {
    fontSize: 16,
    color: '#A1A1AA',
    textAlign: 'center',
  },
  particle: {
    position: 'absolute',
    top: SCREEN_HEIGHT * 0.4,
    width: 8,
    height: 8,
    borderRadius: 4,
  },
});
```

### 4.2 Failure Animation

```typescript
// shared/components/Feedback/FailureAnimation.tsx
import React, { useEffect, useRef } from 'react';
import {
  View,
  Text,
  StyleSheet,
  Animated,
  Easing,
  TouchableOpacity,
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import * as Haptics from 'expo-haptics';

type Props = {
  title: string;
  message?: string;
  onRetry?: () => void;
  onDismiss?: () => void;
};

export function FailureAnimation({
  title,
  message,
  onRetry,
  onDismiss,
}: Props) {
  const scaleAnim = useRef(new Animated.Value(0)).current;
  const opacityAnim = useRef(new Animated.Value(0)).current;
  const shakeAnim = useRef(new Animated.Value(0)).current;
  const iconScaleAnim = useRef(new Animated.Value(0)).current;
  const textAnim = useRef(new Animated.Value(0)).current;

  useEffect(() => {
    // Trigger haptic feedback
    Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);

    // Animation sequence
    Animated.sequence([
      // Background fade in
      Animated.parallel([
        Animated.timing(opacityAnim, {
          toValue: 1,
          duration: 200,
          useNativeDriver: true,
        }),
        Animated.spring(scaleAnim, {
          toValue: 1,
          tension: 50,
          friction: 7,
          useNativeDriver: true,
        }),
      ]),
      // Icon appear with shake
      Animated.parallel([
        Animated.spring(iconScaleAnim, {
          toValue: 1,
          tension: 100,
          friction: 8,
          useNativeDriver: true,
        }),
        Animated.sequence([
          Animated.timing(shakeAnim, {
            toValue: 10,
            duration: 50,
            useNativeDriver: true,
          }),
          Animated.timing(shakeAnim, {
            toValue: -10,
            duration: 50,
            useNativeDriver: true,
          }),
          Animated.timing(shakeAnim, {
            toValue: 10,
            duration: 50,
            useNativeDriver: true,
          }),
          Animated.timing(shakeAnim, {
            toValue: -10,
            duration: 50,
            useNativeDriver: true,
          }),
          Animated.timing(shakeAnim, {
            toValue: 0,
            duration: 50,
            useNativeDriver: true,
          }),
        ]),
      ]),
      // Text fade in
      Animated.timing(textAnim, {
        toValue: 1,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start();
  }, []);

  const handleDismiss = () => {
    Animated.timing(opacityAnim, {
      toValue: 0,
      duration: 200,
      useNativeDriver: true,
    }).start(() => onDismiss?.());
  };

  return (
    <Animated.View
      style={[
        styles.container,
        {
          opacity: opacityAnim,
          transform: [{ scale: scaleAnim }],
        },
      ]}
    >
      {/* Error icon with shake */}
      <Animated.View
        style={[
          styles.iconContainer,
          {
            transform: [
              { scale: iconScaleAnim },
              { translateX: shakeAnim },
            ],
          },
        ]}
      >
        <View style={styles.iconCircle}>
          <Ionicons name="close" size={80} color="#FFFFFF" />
        </View>
      </Animated.View>

      {/* Text */}
      <Animated.View style={[styles.textContainer, { opacity: textAnim }]}>
        <Text style={styles.title}>{title}</Text>
        {message && <Text style={styles.message}>{message}</Text>}
      </Animated.View>

      {/* Action buttons */}
      <Animated.View style={[styles.buttonContainer, { opacity: textAnim }]}>
        {onRetry && (
          <TouchableOpacity style={styles.retryButton} onPress={onRetry}>
            <Ionicons name="refresh" size={20} color="#FFFFFF" />
            <Text style={styles.retryText}>Try Again</Text>
          </TouchableOpacity>
        )}
        <TouchableOpacity style={styles.dismissButton} onPress={handleDismiss}>
          <Text style={styles.dismissText}>Dismiss</Text>
        </TouchableOpacity>
      </Animated.View>
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    backgroundColor: 'rgba(18, 18, 18, 0.96)',
    justifyContent: 'center',
    alignItems: 'center',
    zIndex: 1000,
    paddingHorizontal: 32,
  },
  iconContainer: {
    marginBottom: 32,
  },
  iconCircle: {
    width: 140,
    height: 140,
    borderRadius: 70,
    backgroundColor: '#EF4444',
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#000000',
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.25,
    shadowRadius: 16,
    elevation: 8,
  },
  textContainer: {
    alignItems: 'center',
    marginBottom: 40,
  },
  title: {
    fontSize: 28,
    fontWeight: '700',
    color: '#FFFFFF',
    textAlign: 'center',
    marginBottom: 12,
  },
  message: {
    fontSize: 16,
    color: '#A1A1AA',
    textAlign: 'center',
    lineHeight: 24,
  },
  buttonContainer: {
    width: '100%',
    gap: 12,
  },
  retryButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#4F46E5',
    paddingVertical: 16,
    borderRadius: 12,
    gap: 8,
  },
  retryText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#FFFFFF',
  },
  dismissButton: {
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 16,
  },
  dismissText: {
    fontSize: 16,
    fontWeight: '500',
    color: '#71717A',
  },
});
```

### 4.3 Redemption Flow Manager

```typescript
// shared/components/Feedback/RedemptionFeedback.tsx
import React from 'react';
import { SuccessAnimation } from './SuccessAnimation';
import { FailureAnimation } from './FailureAnimation';

type RedemptionState =
  | { status: 'idle' }
  | { status: 'success'; title: string; discount: string; subtitle?: string }
  | { status: 'error'; title: string; message?: string };

type Props = {
  state: RedemptionState;
  onComplete: () => void;
  onRetry?: () => void;
};

export function RedemptionFeedback({ state, onComplete, onRetry }: Props) {
  if (state.status === 'idle') {
    return null;
  }

  if (state.status === 'success') {
    return (
      <SuccessAnimation
        title={state.title}
        subtitle={state.subtitle}
        discount={state.discount}
        onComplete={onComplete}
      />
    );
  }

  return (
    <FailureAnimation
      title={state.title}
      message={state.message}
      onRetry={onRetry}
      onDismiss={onComplete}
    />
  );
}
```

---

## 5. Design Tokens

### 5.1 Colors

```typescript
// shared/constants/colors.ts
export const colors = {
  // Background
  background: {
    primary: '#121212',
    secondary: '#1E1E1E',
    elevated: '#2D2D2D',
  },

  // Text
  text: {
    primary: '#F9FAFB',
    secondary: '#9CA3AF',
    tertiary: '#6B7280',
    inverse: '#121212',
  },

  // Brand
  brand: {
    primary: '#4F46E5', // Muted indigo
    accent: '#10B981',  // Emerald (savings)
  },

  // Map
  map: {
    userLocation: '#3B82F6',
    dealMarker: '#F59E0B',
    flashDeal: '#EF4444',
  },

  // Semantic
  semantic: {
    success: '#10B981',
    error: '#EF4444',
    warning: '#F59E0B',
    info: '#3B82F6',
  },

  // Heatmap gradient
  heatmap: {
    cool: 'rgba(59, 130, 246, 0.35)',
    warm: 'rgba(245, 158, 11, 0.6)',
    hot: 'rgba(239, 68, 68, 0.9)',
  },
} as const;
```

### 5.2 Typography

```typescript
// shared/constants/typography.ts
export const typography = {
  // Display
  displayLarge: {
    fontSize: 48,
    fontWeight: '800' as const,
    lineHeight: 56,
  },
  displayMedium: {
    fontSize: 32,
    fontWeight: '700' as const,
    lineHeight: 40,
  },

  // Headings
  h1: {
    fontSize: 28,
    fontWeight: '700' as const,
    lineHeight: 36,
  },
  h2: {
    fontSize: 24,
    fontWeight: '600' as const,
    lineHeight: 32,
  },
  h3: {
    fontSize: 20,
    fontWeight: '600' as const,
    lineHeight: 28,
  },

  // Body
  bodyLarge: {
    fontSize: 18,
    fontWeight: '400' as const,
    lineHeight: 28,
  },
  bodyMedium: {
    fontSize: 16,
    fontWeight: '400' as const,
    lineHeight: 24,
  },
  bodySmall: {
    fontSize: 14,
    fontWeight: '400' as const,
    lineHeight: 20,
  },

  // Labels
  labelLarge: {
    fontSize: 16,
    fontWeight: '600' as const,
    lineHeight: 24,
  },
  labelMedium: {
    fontSize: 14,
    fontWeight: '500' as const,
    lineHeight: 20,
  },
  labelSmall: {
    fontSize: 12,
    fontWeight: '500' as const,
    lineHeight: 16,
  },
} as const;
```

### 5.3 Spacing

```typescript
// shared/constants/spacing.ts
export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
  xxxl: 64,
} as const;
```

---

## 6. Component Usage Examples

### 6.1 Buyer App — Map Screen

```typescript
// apps/buyer/app/(tabs)/map.tsx
import { View, StyleSheet } from 'react-native';
import { FrictionlessMap } from '@shared/components/Map/FrictionlessMap';
import { OfferMarker } from '../components/OfferMarker';
import { LocationPermissionGate } from '@shared/components/LocationPermissionGate';
import { useNearbyOffers } from '../hooks/useNearbyOffers';
import { useLocationTracking } from '../hooks/useLocationTracking';

export default function MapScreen() {
  const { data: offers } = useNearbyOffers();

  // Enable foreground location tracking
  useLocationTracking({ enabled: true });

  return (
    <LocationPermissionGate>
      <View style={styles.container}>
        <FrictionlessMap showUserLocation>
          {offers?.map((offer) => (
            <OfferMarker
              key={offer.id}
              id={offer.id}
              coordinate={[offer.longitude, offer.latitude]}
              discount={offer.discount_label}
              onPress={() => router.push(`/offer/${offer.id}`)}
            />
          ))}
        </FrictionlessMap>
      </View>
    </LocationPermissionGate>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```

### 6.2 Seller App — Heatmap Screen

```typescript
// apps/seller/app/(tabs)/heatmap.tsx
import { View, Text, StyleSheet } from 'react-native';
import { FrictionlessMap } from '@shared/components/Map/FrictionlessMap';
import { UserHeatmapLayer } from '../components/HeatmapLayer';
import { LocationPermissionGate } from '@shared/components/LocationPermissionGate';
import { useHeatmapData } from '../hooks/useHeatmapData';
import { useUserLocation } from '../hooks/useUserLocation';

export default function HeatmapScreen() {
  const { location } = useUserLocation();
  const { data, isLoading, dataUpdatedAt } = useHeatmapData({
    latitude: location?.latitude ?? 0,
    longitude: location?.longitude ?? 0,
    enabled: !!location,
  });

  return (
    <LocationPermissionGate>
      <View style={styles.container}>
        <FrictionlessMap
          initialCoordinates={
            location ? [location.longitude, location.latitude] : undefined
          }
          showUserLocation
        >
          {data && <UserHeatmapLayer points={data.points} />}
        </FrictionlessMap>

        {/* Stats overlay */}
        <View style={styles.statsOverlay}>
          <Text style={styles.statsText}>
            {data?.total_users ?? 0} users nearby
          </Text>
          <Text style={styles.refreshText}>
            {isLoading ? 'Refreshing...' : `Updated ${formatTime(dataUpdatedAt)}`}
          </Text>
        </View>
      </View>
    </LocationPermissionGate>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  statsOverlay: {
    position: 'absolute',
    top: 60,
    left: 16,
    right: 16,
    backgroundColor: 'rgba(15, 15, 15, 0.9)',
    padding: 16,
    borderRadius: 12,
  },
  statsText: {
    fontSize: 18,
    fontWeight: '600',
    color: '#FFFFFF',
  },
  refreshText: {
    fontSize: 12,
    color: '#71717A',
    marginTop: 4,
  },
});
```

---

## 7. Accessibility Guidelines

| Component | Accessibility Feature |
|-----------|----------------------|
| Map | `accessibilityLabel` on markers, voice-over hints |
| QR Scanner | Audio feedback on successful scan |
| Success Animation | Screen reader announcement of redemption |
| Failure Animation | Haptic feedback + screen reader announcement |
| Buttons | Minimum 44x44pt touch target |

```typescript
// Example: Accessible button
<TouchableOpacity
  style={styles.button}
  accessibilityRole="button"
  accessibilityLabel="Scan QR code to redeem offer"
  accessibilityHint="Opens camera to scan merchant QR code"
>
  <Text>Scan to Redeem</Text>
</TouchableOpacity>
```

## Related Documents

**Dependencies**
- BRAND-01: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 3
- DES-02: Section 2
- DES-03: Section 2
- TECH-05: Section 9
- THREAD-02: Section 3
- THREAD-01: Section 3

**Implementation Guides**
- GUIDE-01: Section 2
- GUIDE-02: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Design Lead | Updated palette and new components |
| 1.1 | 2026-01-30 | Design Lead | Added Lite Mode specification |
| 1.0 | 2026-01-30 | Design Lead | Standardized metadata and cross-references |
