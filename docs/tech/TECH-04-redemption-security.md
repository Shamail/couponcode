---
document_id: TECH-04
version: 1.2
status: Final
priority: P0
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - TECH-06
related_documents:
  - GUIDE-03
  - OPS-02
  - DATA-01
  - THREAD-02
  - TECH-11
---

# TECH-04: Redemption Security & QR Protocol

## Executive Summary

This document defines the security architecture for the SafeColor Verification redemption system. It outlines how rotating QR codes, color confirmation, and server-side validation combine to prevent fraud without hardware dependencies.

The protocol is designed for fast in-store use and minimal merchant friction.

> Frictionless SafeColor Verification — Secure, Hardware-Free Deal Verification

## Overview

This document defines the security architecture for the **SafeColor Verification** redemption system. The protocol enables secure deal verification without hardware integration (no NFC, no Bluetooth beacons) using only:

1. **Dynamic QR Codes** — Rotating signed tokens
2. **Color Code Verification** — Visual confirmation layer
3. **Server-Side Validation** — Cryptographic proof of legitimacy

**Philosophy:** Zero hardware. Zero friction. Maximum security.

---

## 1. Security Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        REDEMPTION SECURITY LAYERS                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   Layer 1: CRYPTOGRAPHIC                                                │
│   ┌──────────────────────────────────────────────────────────────┐     │
│   │  JWT Token (RS256 signed)                                     │     │
│   │  • deal_id, user_id, expiry                                   │     │
│   │  • 60-second rotation                                         │     │
│   │  • Server-side signature verification                         │     │
│   └──────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   Layer 2: VISUAL                                                       │
│   ┌──────────────────────────────────────────────────────────────┐     │
│   │  Color Code Matching                                          │     │
│   │  • Dynamically generated per redemption                       │     │
│   │  • Buyer screen displays color                                │     │
│   │  • Seller verifies color match                                │     │
│   └──────────────────────────────────────────────────────────────┘     │
│                                                                          │
│   Layer 3: TEMPORAL                                                     │
│   ┌──────────────────────────────────────────────────────────────┐     │
│   │  Time-Bound Validity                                          │     │
│   │  • QR expires every 60 seconds                                │     │
│   │  • Deal has max redemption window                             │     │
│   │  • One-time use enforcement                                   │     │
│   └──────────────────────────────────────────────────────────────┘     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Dynamic QR Code Generation

### 2.1 QR Token Structure

The QR code encodes a signed JWT containing the redemption claim:

```typescript
// Token payload structure
interface RedemptionTokenPayload {
  // Standard JWT claims
  iss: 'frictionless';           // Issuer
  sub: string;                    // User ID (UUID)
  iat: number;                    // Issued at (Unix timestamp)
  exp: number;                    // Expiration (Unix timestamp, +60s from iat)
  jti: string;                    // Unique token ID (UUID v4)

  // Custom claims
  deal_id: string;                // Deal being redeemed (UUID)
  store_id: string;               // Store where redemption occurs (UUID)
  nonce: string;                  // Random 8-char alphanumeric
}
```

### 2.2 Token Generation (Buyer App)

```typescript
// shared/lib/redemption/generateQRToken.ts
import * as Crypto from 'expo-crypto';
import { SignJWT } from 'jose';

const PRIVATE_KEY = process.env.EXPO_PUBLIC_REDEMPTION_PRIVATE_KEY;
const TOKEN_LIFETIME_SECONDS = 60;

interface GenerateQRTokenParams {
  userId: string;
  dealId: string;
  storeId: string;
}

export async function generateRedemptionToken({
  userId,
  dealId,
  storeId,
}: GenerateQRTokenParams): Promise<string> {
  const now = Math.floor(Date.now() / 1000);
  const jti = Crypto.randomUUID();
  const nonce = generateNonce(8);

  const token = await new SignJWT({
    deal_id: dealId,
    store_id: storeId,
    nonce,
  })
    .setProtectedHeader({ alg: 'RS256', typ: 'JWT' })
    .setIssuer('frictionless')
    .setSubject(userId)
    .setIssuedAt(now)
    .setExpirationTime(now + TOKEN_LIFETIME_SECONDS)
    .setJti(jti)
    .sign(PRIVATE_KEY);

  return token;
}

function generateNonce(length: number): string {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZabcdefghjkmnpqrstuvwxyz23456789';
  let result = '';
  const randomBytes = Crypto.getRandomBytes(length);
  for (let i = 0; i < length; i++) {
    result += chars[randomBytes[i] % chars.length];
  }
  return result;
}
```

### 2.3 QR Auto-Rotation (60-Second Cycle)

The buyer app automatically refreshes the QR code before expiration:

```typescript
// apps/buyer/hooks/useRedemptionQR.ts
import { useState, useEffect, useCallback, useRef } from 'react';
import { generateRedemptionToken } from '@shared/lib/redemption/generateQRToken';

const ROTATION_INTERVAL_MS = 55_000;  // Refresh 5s before expiry
const TOKEN_LIFETIME_MS = 60_000;

interface UseRedemptionQRParams {
  dealId: string;
  storeId: string;
  userId: string;
  enabled: boolean;
}

interface RedemptionQRState {
  token: string | null;
  expiresAt: number;
  colorCode: string;
  isGenerating: boolean;
  error: Error | null;
}

export function useRedemptionQR({
  dealId,
  storeId,
  userId,
  enabled,
}: UseRedemptionQRParams) {
  const [state, setState] = useState<RedemptionQRState>({
    token: null,
    expiresAt: 0,
    colorCode: '#000000',
    isGenerating: false,
    error: null,
  });

  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  const generateNewToken = useCallback(async () => {
    if (!enabled) return;

    setState(prev => ({ ...prev, isGenerating: true, error: null }));

    try {
      const token = await generateRedemptionToken({
        userId,
        dealId,
        storeId,
      });

      const colorCode = generateColorCode(token);
      const expiresAt = Date.now() + TOKEN_LIFETIME_MS;

      setState({
        token,
        expiresAt,
        colorCode,
        isGenerating: false,
        error: null,
      });
    } catch (error) {
      setState(prev => ({
        ...prev,
        isGenerating: false,
        error: error as Error,
      }));
    }
  }, [dealId, storeId, userId, enabled]);

  // Initial generation
  useEffect(() => {
    if (enabled) {
      generateNewToken();
    }
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [enabled]);

  // Auto-rotation every 55 seconds
  useEffect(() => {
    if (!enabled) return;

    intervalRef.current = setInterval(generateNewToken, ROTATION_INTERVAL_MS);

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, [generateNewToken, enabled]);

  return {
    ...state,
    refresh: generateNewToken,
    timeRemaining: Math.max(0, state.expiresAt - Date.now()),
  };
}

// Generate deterministic color from token
function generateColorCode(token: string): string {
  // Use last 6 chars of token hash for color
  const hash = simpleHash(token);
  const hue = hash % 360;
  // High saturation, medium lightness for visibility
  return hslToHex(hue, 80, 50);
}

function simpleHash(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  return Math.abs(hash);
}

function hslToHex(h: number, s: number, l: number): string {
  s /= 100;
  l /= 100;
  const a = s * Math.min(l, 1 - l);
  const f = (n: number) => {
    const k = (n + h / 30) % 12;
    const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
    return Math.round(255 * color).toString(16).padStart(2, '0');
  };
  return `#${f(0)}${f(8)}${f(4)}`.toUpperCase();
}
```

---

## 3. The SafeColor Verification Protocol

### 3.1 Five-Step Verification Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     VISUAL HANDSHAKE: 5-STEP FLOW                        │
└─────────────────────────────────────────────────────────────────────────┘

  BUYER APP                         API                         SELLER APP
      │                              │                              │
      │  ┌────────────────────┐      │                              │
      │  │ 1. DISPLAY QR CODE │      │                              │
      │  │    + Color Code    │      │                              │
      │  └────────────────────┘      │                              │
      │                              │                              │
      │                              │      ┌───────────────────┐   │
      │                              │      │ 2. SELLER SCANS   │   │
      │                              │◄─────│    QR CODE        │───│
      │                              │      └───────────────────┘   │
      │                              │                              │
      │                              │  ┌────────────────────────┐  │
      │                              │  │ 3. API VALIDATES:      │  │
      │                              │  │    • Signature (RS256) │  │
      │                              │  │    • Expiration        │  │
      │                              │  │    • Deal active?      │  │
      │                              │  │    • Already redeemed? │  │
      │                              │  └────────────────────────┘  │
      │                              │                              │
      │                              │      ┌───────────────────┐   │
      │                              │      │ 4. API RETURNS:   │   │
      │                              │─────►│    valid: true    │───│
      │                              │      │    color_code     │   │
      │                              │      └───────────────────┘   │
      │                              │                              │
      │  ┌────────────────────┐      │      ┌───────────────────┐   │
      │  │ Buyer shows        │      │      │ 5. SELLER COMPARES│   │
      │  │ COLOR SCREEN       │◄────────────│    COLOR MATCH    │───│
      │  │ (e.g., #00FF88)    │      │      │    ✓ = Success    │   │
      │  └────────────────────┘      │      └───────────────────┘   │
      │                              │                              │
```

### 3.2 Step-by-Step Implementation

#### Step 1: Buyer Displays QR + Color

```typescript
// apps/buyer/components/RedemptionScreen.tsx
import React from 'react';
import { View, StyleSheet, Text, Animated } from 'react-native';
import QRCode from 'react-native-qrcode-svg';
import { useRedemptionQR } from '../hooks/useRedemptionQR';

interface RedemptionScreenProps {
  dealId: string;
  storeId: string;
  userId: string;
  storeName: string;
  dealTitle: string;
}

export function RedemptionScreen({
  dealId,
  storeId,
  userId,
  storeName,
  dealTitle,
}: RedemptionScreenProps) {
  const { token, colorCode, timeRemaining, isGenerating } = useRedemptionQR({
    dealId,
    storeId,
    userId,
    enabled: true,
  });

  const pulseAnim = React.useRef(new Animated.Value(1)).current;

  // Pulse animation for color background
  React.useEffect(() => {
    const pulse = Animated.loop(
      Animated.sequence([
        Animated.timing(pulseAnim, {
          toValue: 0.85,
          duration: 500,
          useNativeDriver: true,
        }),
        Animated.timing(pulseAnim, {
          toValue: 1,
          duration: 500,
          useNativeDriver: true,
        }),
      ])
    );
    pulse.start();
    return () => pulse.stop();
  }, []);

  if (!token) {
    return <LoadingScreen />;
  }

  const secondsRemaining = Math.ceil(timeRemaining / 1000);

  return (
    <Animated.View
      style={[
        styles.container,
        { backgroundColor: colorCode, opacity: pulseAnim }
      ]}
    >
      <View style={styles.header}>
        <Text style={styles.storeName}>{storeName}</Text>
        <Text style={styles.dealTitle}>{dealTitle}</Text>
      </View>

      <View style={styles.qrContainer}>
        <View style={styles.qrWrapper}>
          <QRCode
            value={token}
            size={220}
            backgroundColor="white"
            color="black"
            quietZone={10}
          />
        </View>
        <Text style={styles.instruction}>Show this to the seller</Text>
      </View>

      <View style={styles.footer}>
        <View style={styles.timerContainer}>
          <Text style={styles.timerLabel}>Refreshing in</Text>
          <Text style={styles.timerValue}>{secondsRemaining}s</Text>
        </View>
        <View style={[styles.colorSwatch, { backgroundColor: colorCode }]}>
          <Text style={styles.colorCode}>{colorCode}</Text>
        </View>
      </View>
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-between',
    padding: 24,
  },
  header: {
    alignItems: 'center',
    paddingTop: 40,
  },
  storeName: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
    textShadowColor: 'rgba(0,0,0,0.3)',
    textShadowOffset: { width: 1, height: 1 },
    textShadowRadius: 3,
  },
  dealTitle: {
    fontSize: 18,
    color: 'white',
    marginTop: 8,
    textShadowColor: 'rgba(0,0,0,0.3)',
    textShadowOffset: { width: 1, height: 1 },
    textShadowRadius: 3,
  },
  qrContainer: {
    alignItems: 'center',
  },
  qrWrapper: {
    backgroundColor: 'white',
    padding: 16,
    borderRadius: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 8,
  },
  instruction: {
    marginTop: 16,
    fontSize: 16,
    color: 'white',
    fontWeight: '600',
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingBottom: 40,
  },
  timerContainer: {
    alignItems: 'center',
  },
  timerLabel: {
    fontSize: 12,
    color: 'rgba(255,255,255,0.8)',
  },
  timerValue: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
  },
  colorSwatch: {
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 8,
    borderWidth: 2,
    borderColor: 'white',
  },
  colorCode: {
    fontSize: 14,
    fontWeight: 'bold',
    color: 'white',
    textShadowColor: 'rgba(0,0,0,0.5)',
    textShadowOffset: { width: 1, height: 1 },
    textShadowRadius: 2,
  },
});
```

#### Step 2: Seller Scans QR

```typescript
// apps/seller/components/QRScanner.tsx
import React, { useState } from 'react';
import { View, StyleSheet, Text } from 'react-native';
import { CameraView, useCameraPermissions } from 'expo-camera';
import * as Haptics from 'expo-haptics';
import { useVerifyRedemption } from '../hooks/useVerifyRedemption';

export function QRScanner() {
  const [permission, requestPermission] = useCameraPermissions();
  const [scanned, setScanned] = useState(false);
  const { mutate: verifyRedemption, isPending, data, error } = useVerifyRedemption();

  const handleBarCodeScanned = ({ data: token }: { data: string }) => {
    if (scanned || isPending) return;

    setScanned(true);
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium);

    // Step 2: Send JWT to API
    verifyRedemption({ token });
  };

  const handleReset = () => {
    setScanned(false);
  };

  if (!permission?.granted) {
    return <PermissionRequest onRequest={requestPermission} />;
  }

  // Show verification result if we have one
  if (data || error) {
    return (
      <VerificationResult
        result={data}
        error={error}
        onReset={handleReset}
      />
    );
  }

  return (
    <View style={styles.container}>
      <CameraView
        style={styles.camera}
        facing="back"
        barcodeScannerSettings={{
          barcodeTypes: ['qr'],
        }}
        onBarcodeScanned={scanned ? undefined : handleBarCodeScanned}
      >
        <View style={styles.overlay}>
          <View style={styles.scanArea}>
            <View style={[styles.corner, styles.topLeft]} />
            <View style={[styles.corner, styles.topRight]} />
            <View style={[styles.corner, styles.bottomLeft]} />
            <View style={[styles.corner, styles.bottomRight]} />
          </View>
          <Text style={styles.instruction}>
            Scan customer's QR code
          </Text>
        </View>
      </CameraView>

      {isPending && (
        <View style={styles.loadingOverlay}>
          <Text style={styles.loadingText}>Verifying...</Text>
        </View>
      )}
    </View>
  );
}
```

#### Steps 3-4: API Verification

```typescript
// packages/api/src/routes/redemption.ts
import { Hono } from 'hono';
import { jwtVerify } from 'jose';
import { db } from '../lib/db';
import { HTTPException } from 'hono/http-exception';

const app = new Hono();

const PUBLIC_KEY = process.env.REDEMPTION_PUBLIC_KEY;

interface VerifyRedemptionRequest {
  token: string;
}

interface VerifyRedemptionResponse {
  valid: boolean;
  color_code: string;
  customer: {
    name: string;
    avatar_url: string | null;
  };
  deal: {
    id: string;
    title: string;
    discount_type: 'percentage' | 'fixed';
    discount_value: number;
  };
  redemption_id: string;
}

app.post('/api/v1/redeem', async (c) => {
  const sellerId = c.get('userId'); // From auth middleware
  const { token } = await c.req.json<VerifyRedemptionRequest>();

  // ═══════════════════════════════════════════════════════════════════════
  // STEP 3: API VALIDATES TOKEN
  // ═══════════════════════════════════════════════════════════════════════

  // 3.1: Verify JWT signature (RS256)
  let payload;
  try {
    const { payload: verified } = await jwtVerify(token, PUBLIC_KEY, {
      issuer: 'frictionless',
      algorithms: ['RS256'],
    });
    payload = verified;
  } catch (err) {
    if (err.code === 'ERR_JWT_EXPIRED') {
      throw new HTTPException(401, {
        message: 'QR code has expired. Ask customer to refresh.',
        cause: { code: 'EXPIRED_TOKEN' },
      });
    }
    throw new HTTPException(401, {
      message: 'Invalid QR code',
      cause: { code: 'INVALID_TOKEN' },
    });
  }

  const { sub: userId, deal_id: dealId, store_id: storeId, jti } = payload;

  // 3.2: Verify seller owns this store
  const store = await db.query.stores.findFirst({
    where: (stores, { eq, and }) =>
      and(eq(stores.id, storeId), eq(stores.seller_id, sellerId)),
  });

  if (!store) {
    throw new HTTPException(403, {
      message: 'You cannot redeem deals for this store',
      cause: { code: 'FORBIDDEN' },
    });
  }

  // 3.3: Check if deal is active
  const deal = await db.query.deals.findFirst({
    where: (deals, { eq, and, gt }) =>
      and(
        eq(deals.id, dealId),
        eq(deals.store_id, storeId),
        eq(deals.is_active, true),
        gt(deals.expires_at, new Date())
      ),
  });

  if (!deal) {
    throw new HTTPException(404, {
      message: 'This deal is no longer active',
      cause: { code: 'DEAL_INACTIVE' },
    });
  }

  // 3.4: Check if already redeemed (by this user for this deal)
  const existingRedemption = await db.query.redemptions.findFirst({
    where: (redemptions, { eq, and }) =>
      and(
        eq(redemptions.user_id, userId),
        eq(redemptions.deal_id, dealId),
        eq(redemptions.status, 'verified')
      ),
  });

  if (existingRedemption) {
    throw new HTTPException(409, {
      message: 'Customer has already redeemed this deal',
      cause: { code: 'ALREADY_REDEEMED' },
    });
  }

  // 3.5: Check max redemptions not exceeded
  if (deal.max_redemptions && deal.current_redemptions >= deal.max_redemptions) {
    throw new HTTPException(410, {
      message: 'This deal has reached its redemption limit',
      cause: { code: 'REDEMPTION_LIMIT' },
    });
  }

  // 3.6: Check token hasn't been used (jti = unique token ID)
  const usedToken = await db.query.usedTokens.findFirst({
    where: (tokens, { eq }) => eq(tokens.jti, jti),
  });

  if (usedToken) {
    throw new HTTPException(409, {
      message: 'This QR code has already been scanned',
      cause: { code: 'TOKEN_USED' },
    });
  }

  // ═══════════════════════════════════════════════════════════════════════
  // STEP 4: CREATE REDEMPTION & RETURN COLOR CODE
  // ═══════════════════════════════════════════════════════════════════════

  // Generate color code (deterministic from token)
  const colorCode = generateColorCode(token);

  // Get customer info
  const customer = await db.query.users.findFirst({
    where: (users, { eq }) => eq(users.id, userId),
    columns: { name: true, avatar_url: true },
  });

  // Create redemption record
  const [redemption] = await db
    .insert(redemptions)
    .values({
      user_id: userId,
      deal_id: dealId,
      seller_id: sellerId,
      store_id: storeId,
      status: 'verified',
      qr_code: jti,
      color_code: colorCode,
      verified_at: new Date(),
    })
    .returning({ id: redemptions.id });

  // Mark token as used
  await db.insert(usedTokens).values({
    jti,
    used_at: new Date(),
    expires_at: new Date(Date.now() + 5 * 60 * 1000), // Keep for 5 min then auto-delete
  });

  // Increment redemption count
  await db
    .update(deals)
    .set({ current_redemptions: deal.current_redemptions + 1 })
    .where(eq(deals.id, dealId));

  // ═══════════════════════════════════════════════════════════════════════
  // RETURN: valid: true + color_code
  // ═══════════════════════════════════════════════════════════════════════

  return c.json<VerifyRedemptionResponse>({
    valid: true,
    color_code: colorCode,
    customer: {
      name: customer?.name ?? 'Customer',
      avatar_url: customer?.avatar_url ?? null,
    },
    deal: {
      id: deal.id,
      title: deal.title,
      discount_type: deal.discount_type,
      discount_value: deal.discount_value,
    },
    redemption_id: redemption.id,
  });
});

function generateColorCode(token: string): string {
  let hash = 0;
  for (let i = 0; i < token.length; i++) {
    const char = token.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  const hue = Math.abs(hash) % 360;
  return hslToHex(hue, 80, 50);
}

function hslToHex(h: number, s: number, l: number): string {
  s /= 100;
  l /= 100;
  const a = s * Math.min(l, 1 - l);
  const f = (n: number) => {
    const k = (n + h / 30) % 12;
    const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
    return Math.round(255 * color).toString(16).padStart(2, '0');
  };
  return `#${f(0)}${f(8)}${f(4)}`.toUpperCase();
}

export default app;
```

#### Step 5: Seller Verifies Color Match

```typescript
// apps/seller/components/VerificationResult.tsx
import React, { useEffect } from 'react';
import { View, StyleSheet, Text, TouchableOpacity, Animated } from 'react-native';
import * as Haptics from 'expo-haptics';
import { Ionicons } from '@expo/vector-icons';

interface VerificationResultProps {
  result: {
    valid: boolean;
    color_code: string;
    customer: { name: string };
    deal: { title: string; discount_type: string; discount_value: number };
  } | null;
  error: Error | null;
  onReset: () => void;
}

export function VerificationResult({ result, error, onReset }: VerificationResultProps) {
  const scaleAnim = React.useRef(new Animated.Value(0)).current;
  const pulseAnim = React.useRef(new Animated.Value(1)).current;

  useEffect(() => {
    // Success haptic
    if (result?.valid) {
      Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    } else {
      Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
    }

    // Scale in animation
    Animated.spring(scaleAnim, {
      toValue: 1,
      friction: 5,
      tension: 100,
      useNativeDriver: true,
    }).start();

    // Pulse animation for color display
    if (result?.valid) {
      Animated.loop(
        Animated.sequence([
          Animated.timing(pulseAnim, {
            toValue: 0.9,
            duration: 400,
            useNativeDriver: true,
          }),
          Animated.timing(pulseAnim, {
            toValue: 1,
            duration: 400,
            useNativeDriver: true,
          }),
        ])
      ).start();
    }
  }, [result]);

  if (error || !result?.valid) {
    return (
      <View style={[styles.container, styles.errorContainer]}>
        <Animated.View style={[styles.iconContainer, { transform: [{ scale: scaleAnim }] }]}>
          <Ionicons name="close-circle" size={120} color="#FF4444" />
        </Animated.View>
        <Text style={styles.errorTitle}>Verification Failed</Text>
        <Text style={styles.errorMessage}>
          {error?.message || 'Unable to verify this QR code'}
        </Text>
        <TouchableOpacity style={styles.retryButton} onPress={onReset}>
          <Text style={styles.retryButtonText}>Try Again</Text>
        </TouchableOpacity>
      </View>
    );
  }

  const discountText = result.deal.discount_type === 'percentage'
    ? `${result.deal.discount_value}% OFF`
    : `${result.deal.discount_value} MAD OFF`;

  return (
    <Animated.View
      style={[
        styles.container,
        styles.successContainer,
        { backgroundColor: result.color_code, transform: [{ scale: pulseAnim }] }
      ]}
    >
      <Animated.View style={[styles.iconContainer, { transform: [{ scale: scaleAnim }] }]}>
        <Ionicons name="checkmark-circle" size={120} color="white" />
      </Animated.View>

      <Text style={styles.successTitle}>VERIFIED</Text>

      <View style={styles.detailsCard}>
        <Text style={styles.customerName}>{result.customer.name}</Text>
        <Text style={styles.dealTitle}>{result.deal.title}</Text>
        <View style={styles.discountBadge}>
          <Text style={styles.discountText}>{discountText}</Text>
        </View>
      </View>

      {/* COLOR VERIFICATION SECTION */}
      <View style={styles.colorVerifySection}>
        <Text style={styles.colorVerifyTitle}>VERIFY COLOR MATCH</Text>
        <View style={styles.colorCompare}>
          <View style={styles.colorBox}>
            <View style={[styles.colorSwatch, { backgroundColor: result.color_code }]} />
            <Text style={styles.colorLabel}>Your Screen</Text>
          </View>
          <Ionicons name="swap-horizontal" size={32} color="white" />
          <View style={styles.colorBox}>
            <View style={[styles.colorSwatch, { backgroundColor: result.color_code, opacity: 0.7 }]} />
            <Text style={styles.colorLabel}>Customer Screen</Text>
          </View>
        </View>
        <Text style={styles.colorCode}>{result.color_code}</Text>
        <Text style={styles.colorInstruction}>
          Confirm the customer's screen shows the same color
        </Text>
      </View>

      <TouchableOpacity style={styles.doneButton} onPress={onReset}>
        <Text style={styles.doneButtonText}>Done - Scan Next</Text>
      </TouchableOpacity>
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
  },
  errorContainer: {
    backgroundColor: '#1a1a2e',
  },
  successContainer: {
    // backgroundColor set dynamically to color_code
  },
  iconContainer: {
    marginBottom: 24,
  },
  successTitle: {
    fontSize: 36,
    fontWeight: 'bold',
    color: 'white',
    textShadowColor: 'rgba(0,0,0,0.3)',
    textShadowOffset: { width: 2, height: 2 },
    textShadowRadius: 4,
  },
  errorTitle: {
    fontSize: 28,
    fontWeight: 'bold',
    color: '#FF4444',
    marginBottom: 12,
  },
  errorMessage: {
    fontSize: 16,
    color: '#999',
    textAlign: 'center',
    marginBottom: 32,
  },
  detailsCard: {
    backgroundColor: 'rgba(255,255,255,0.95)',
    borderRadius: 16,
    padding: 20,
    alignItems: 'center',
    marginTop: 24,
    width: '100%',
    maxWidth: 320,
  },
  customerName: {
    fontSize: 20,
    fontWeight: 'bold',
    color: '#333',
  },
  dealTitle: {
    fontSize: 16,
    color: '#666',
    marginTop: 4,
  },
  discountBadge: {
    backgroundColor: '#00CC66',
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 20,
    marginTop: 12,
  },
  discountText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
  },
  colorVerifySection: {
    marginTop: 32,
    alignItems: 'center',
    backgroundColor: 'rgba(0,0,0,0.2)',
    padding: 20,
    borderRadius: 16,
    width: '100%',
  },
  colorVerifyTitle: {
    fontSize: 14,
    fontWeight: 'bold',
    color: 'white',
    letterSpacing: 2,
    marginBottom: 16,
  },
  colorCompare: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    gap: 16,
  },
  colorBox: {
    alignItems: 'center',
  },
  colorSwatch: {
    width: 60,
    height: 60,
    borderRadius: 8,
    borderWidth: 3,
    borderColor: 'white',
  },
  colorLabel: {
    fontSize: 12,
    color: 'rgba(255,255,255,0.8)',
    marginTop: 8,
  },
  colorCode: {
    fontSize: 20,
    fontWeight: 'bold',
    color: 'white',
    marginTop: 16,
    fontFamily: 'monospace',
  },
  colorInstruction: {
    fontSize: 14,
    color: 'rgba(255,255,255,0.9)',
    textAlign: 'center',
    marginTop: 8,
  },
  retryButton: {
    backgroundColor: '#FF4444',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 8,
  },
  retryButtonText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
  },
  doneButton: {
    backgroundColor: 'rgba(255,255,255,0.3)',
    paddingHorizontal: 32,
    paddingVertical: 16,
    borderRadius: 8,
    marginTop: 32,
    borderWidth: 2,
    borderColor: 'white',
  },
  doneButtonText: {
    fontSize: 18,
    fontWeight: 'bold',
    color: 'white',
  },
});
```

---

## 4. Security Considerations

### 4.1 Threat Model

| Threat | Mitigation |
|--------|------------|
| **QR Screenshot/Sharing** | 60-second expiration + color code visual verification |
| **Token Replay** | `jti` (JWT ID) tracked in `used_tokens` table, one-time use |
| **Token Forgery** | RS256 asymmetric signing, private key server-side only |
| **Man-in-the-Middle** | HTTPS only, certificate pinning in mobile apps |
| **Brute Force** | Rate limiting on verification endpoint (10 req/min per seller) |
| **Expired Deal Redemption** | Server validates deal `is_active` and `expires_at` |
| **Double Redemption** | Unique constraint on (user_id, deal_id) in redemptions table |

### 4.2 Key Management

```typescript
// Key generation (run once during setup)
// openssl genrsa -out private.pem 2048
// openssl rsa -in private.pem -pubout -out public.pem

// Environment variables
// REDEMPTION_PRIVATE_KEY - Used by buyer app (embedded, consider token service instead)
// REDEMPTION_PUBLIC_KEY  - Used by API for verification

// For production, consider:
// 1. Token generation service (buyer requests token from API, not self-generated)
// 2. AWS KMS or similar for key storage
// 3. Key rotation every 90 days
```

### 4.3 Production Recommendations

```typescript
// packages/api/src/middleware/redemptionRateLimit.ts
import { Hono } from 'hono';
import { rateLimiter } from 'hono-rate-limiter';

export const redemptionRateLimiter = rateLimiter({
  windowMs: 60 * 1000,  // 1 minute
  limit: 10,             // 10 verifications per minute per seller
  keyGenerator: (c) => c.get('userId'),
  message: {
    success: false,
    error: {
      code: 'RATE_LIMITED',
      message: 'Too many verification attempts. Please slow down.',
    },
  },
});
```

---

## 5. Offline Redemption Protocol

When connectivity is unstable, sellers can validate redemptions using signed offline tokens. This preserves the SafeColor Verification UX while deferring server verification until the device reconnects.

### 5.1 Signed Offline Token Payload

```typescript
interface OfflineTokenPayload {
  offline: true;
  deal_id: string;
  deal_snapshot: { title: string; discount_type: string; discount_value: number };
  device_id: string;
  sequence: number;   // Monotonic counter for fraud detection
  exp: number;        // 24-hour validity window
}
```

### 5.2 Fraud Risk Caps

| Cap | Limit | Rationale |
| --- | --- | --- |
| Per hour | 5 offline redemptions | Prevent burst abuse |
| Per day | 15 offline redemptions | Daily ceiling |
| Max value | 500 MAD | Exposure limit |
| Sequence gap | Alert if >3 skipped | Detect manipulation |

### 5.3 Deferred Reconciliation

- Seller app stores redemptions in an offline ledger (AsyncStorage)
- On reconnect, POST `/redeem/sync` with batch upload
- Server responds with: accepted, duplicate, conflict, rejected

### 5.4 Seller UX Notes

- Display “Offline Verified” badge until server reconciliation completes
- Prevent manual edits to offline entries once stored
---

## 6. Database Schema Additions

```sql
-- Token tracking table (for replay prevention)
CREATE TABLE used_tokens (
  jti VARCHAR(64) PRIMARY KEY,
  used_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL
);

-- Index for cleanup job
CREATE INDEX idx_used_tokens_expires ON used_tokens(expires_at);

-- Cleanup job (run every 5 minutes)
-- Removes tokens older than 5 minutes (since QR expires in 60s, 5min is generous)
DELETE FROM used_tokens WHERE expires_at < NOW();

-- Add color_code to redemptions table if not exists
ALTER TABLE redemptions
ADD COLUMN IF NOT EXISTS color_code VARCHAR(7);
```

---

## 7. Testing Checklist

### 7.1 Unit Tests

- [ ] Token generation produces valid JWT
- [ ] Token expires after 60 seconds
- [ ] Color code is deterministic (same token = same color)
- [ ] Invalid signature is rejected
- [ ] Expired token is rejected

### 7.2 Integration Tests

- [ ] Full redemption flow (buyer generate → seller scan → verify → success)
- [ ] Replay attack prevention (same token fails on second scan)
- [ ] Deal expiration prevents redemption
- [ ] Already-redeemed check works
- [ ] Rate limiting triggers after 10 attempts

### 7.3 Field Tests

- [ ] QR scans reliably in various lighting conditions
- [ ] Color is visible/distinguishable on both screens
- [ ] 60-second window is sufficient for transaction
- [ ] Network latency doesn't cause false failures

---

## 8. Sequence Diagram (Complete Flow)

```
┌──────────┐          ┌──────────┐          ┌──────────┐          ┌──────────┐
│  Buyer   │          │ Buyer    │          │   API    │          │  Seller  │
│  Wallet  │          │  App     │          │  Server  │          │   App    │
└────┬─────┘          └────┬─────┘          └────┬─────┘          └────┬─────┘
     │                     │                     │                     │
     │  Tap "Redeem"       │                     │                     │
     │────────────────────>│                     │                     │
     │                     │                     │                     │
     │                     │  Generate JWT       │                     │
     │                     │  (60s expiry)       │                     │
     │                     │──────────┐          │                     │
     │                     │          │          │                     │
     │                     │<─────────┘          │                     │
     │                     │                     │                     │
     │                     │  Display QR +       │                     │
     │                     │  Color Screen       │                     │
     │                     │──────────┐          │                     │
     │                     │          │          │                     │
     │                     │<─────────┘          │                     │
     │                     │                     │                     │
     │                     │                     │     Scan QR Code    │
     │                     │                     │<────────────────────│
     │                     │                     │                     │
     │                     │                     │  POST /redeem       │
     │                     │                     │  { token: "..." }   │
     │                     │                     │<────────────────────│
     │                     │                     │                     │
     │                     │                     │  Verify Signature   │
     │                     │                     │  Check Expiry       │
     │                     │                     │  Check Deal Active  │
     │                     │                     │  Check Not Redeemed │
     │                     │                     │──────────┐          │
     │                     │                     │          │          │
     │                     │                     │<─────────┘          │
     │                     │                     │                     │
     │                     │                     │  { valid: true,     │
     │                     │                     │    color_code }     │
     │                     │                     │────────────────────>│
     │                     │                     │                     │
     │                     │                     │                     │  Display Color
     │                     │                     │                     │  + Customer Info
     │                     │                     │                     │─────────┐
     │                     │                     │                     │         │
     │                     │  ◄──── Visual Comparison ────►           │<────────┘
     │                     │     (Seller looks at Buyer screen)       │
     │                     │                     │                     │
     │                     │                     │                     │  Confirm Match
     │                     │                     │                     │  ✓ SUCCESS
     │                     │                     │                     │─────────┐
     │                     │                     │                     │         │
     │                     │                     │                     │<────────┘
     │                     │                     │                     │
```

---

## Related Documents

**Dependencies**
- TECH-06: Section 3

**Related Specs**
- TECH-01: Section 4
- DES-01: Section 2
- DATA-01: Section 2
- THREAD-02: Section 3
- TECH-11: Section 3
- BRAND-02: Section 3

**Implementation Guides**
- GUIDE-03: Section 2
- OPS-02: Section 3

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Engineering Lead | Renamed to SafeColor Verification |
| 1.1 | 2026-01-30 | Engineering Lead | Added offline redemption protocol |
| 1.0 | 2026-01-30 | Engineering Lead | Standardized metadata and cross-references |
