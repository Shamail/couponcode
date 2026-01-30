# Design Pivot Plan: "Modern Utility/Lifestyle" Rebrand

## Overview

Transform Frictionless from a "Cyberpunk/Gaming" aesthetic to a "Clean Modern Dark Mode" design inspired by Uber, Airbnb, and modern banking apps. Target demographic shifts from gamers to broader 18-50 Moroccan consumers.

**Key Shift:** From "deal-hunting radar" to "smart shopping companion"

---

## Phase 1: Brand Foundation (Priority: Critical)

### 1.1 Update BRAND-01-voice-tone.md (Complete Rewrite)

**Current State:**
- Tagline: "Hunt. Claim. Win."
- Vibe: "Cyberpunk Meets Souk"
- Voice: Sharp, Electric, Insider, Playful
- Copy: "Loot secured", "Signal detected", "Quiet sector"

**New Direction:**
- Tagline: "Discover. Save. Enjoy." (or "Find deals near you.")
- Vibe: "Modern Discovery"
- Voice: Clear, Helpful, Confident, Warm
- Copy: "Deal claimed", "3 deals nearby", "No deals here yet"

**Files:** `/docs/brand/BRAND-01-voice-tone.md`

### 1.2 Update BRAND-02-gamification.md (Complete Rewrite → Rename to "Membership & Rewards")

**Current Tiers:**
| Tier | Points |
|------|--------|
| Scout | 0 |
| Hunter | 500 |
| Tracker | 2,000 |
| Predator | 10,000 |
| Phantom | 50,000 |

**New Tiers:**
| Tier | Points | Perks |
|------|--------|-------|
| Bronze | 0 | Basic access |
| Silver | 500 | See stock levels |
| Gold | 2,000 | 5-min early access |
| Platinum | 10,000 | 10-min early access + exclusive deals |

**Files:** `/docs/brand/BRAND-02-gamification.md`

### 1.3 Update APPENDIX-A-glossary.md

Replace all gaming terms with utility terms per the mapping below.

**Files:** `/docs/appendix/APPENDIX-A-glossary.md`

---

## Phase 2: Design System Updates

### 2.1 Update Color Tokens in DES-01-component-spec.md

**Current (Cyberpunk):**
```
User ping: #00F5FF (Electric Cyan)
Active deal: #FF00FF (Neon Magenta)
Flash deal: #FF6B00 (Warning Orange)
Heatmap hot: #FF0080 (Plasma Pink)
```

**New (Clean Modern):**
```typescript
colors = {
  background: {
    primary: '#121212',    // Slightly softer black
    secondary: '#1E1E1E',  // Card backgrounds
    elevated: '#2D2D2D',   // Bottom sheets
  },
  brand: {
    primary: '#4F46E5',    // Muted indigo (less saturated)
    accent: '#10B981',     // Emerald (success/savings)
  },
  map: {
    userLocation: '#3B82F6',  // Blue (Uber-like)
    dealMarker: '#F59E0B',    // Amber (warmer)
    flashDeal: '#EF4444',     // Red (urgency)
  },
  text: {
    primary: '#F9FAFB',    // Soft white
    secondary: '#9CA3AF',  // Gray-400
    tertiary: '#6B7280',   // Gray-500
  }
}
```

**Files:** `/docs/design/DES-01-component-spec.md`

### 2.2 Update Motion Guidelines in DES-03

- Reduce pulse intensity: Scale 0.95-1.05 (was 0.9-1.1)
- Slower animations: 3s cycles (was 2s)
- Remove neon glows, use neutral depth shadows
- Use platform-native springs for bottom sheets

**Files:** `/docs/design/DES-03-motion-guidelines.md`

---

## Phase 3: Screen Specifications (Based on ASCII Mockups)

### 3.1 Home Screen (Map Interface)

```
┌──────────────────────────────────────────┐
│  📍 Maarif, Casablanca          [ 🔔 ]   │  <- FloatingHeader
├──────────────────────────────────────────┤
│         [ 🔍 Search Deals... ]           │  <- FloatingSearchBar
│                                          │
│               🏷️                         │  <- DealMarker (amber)
│         🔵                               │  <- UserLocation (blue)
│                    🏷️                    │
│                                          │
│      [ ⚡ 3 Flash Deals Nearby ]         │  <- FlashDealToast
├──────────────────────────────────────────┤
│  🏠      🧭       💼       👤      ⚙️    │  <- 5-tab BottomNav
│ Home   Explore   Wallet   Profile  Menu  │
└──────────────────────────────────────────┘
```

**New Components:**
- `FloatingHeader` - Location context + notification bell
- `FloatingSearchBar` - Rounded pill search input
- `FlashDealToast` - Floating pill alert
- `BottomTabBar` - 5 tabs (was 4)

### 3.2 Deal Discovery (Bottom Sheet)

```
┌──────────────────────────────────────────┐
│              ____                        │  <- Drag handle
│   [  IMAGE  ]   **Café Milano**          │
│   [         ]   Maarif • 120m away       │
│   ┌──────────────────────────────────┐   │
│   │  ☕ 30% OFF ALL COFFEE           │   │  <- DealValueCard
│   └──────────────────────────────────┘   │
│   Valid for: 14 mins                     │
│   ⭐⭐⭐⭐⭐ (4.8)                       │
│   [       CLAIM DEAL (FREE)      ]       │  <- ClaimButton
└──────────────────────────────────────────┘
```

**New Components:**
- `DealBottomSheet` - Using @gorhom/bottom-sheet
- `DealValueCard` - Highlighted discount box
- `ClaimButton` - Large accessible CTA (min 52px height)

### 3.3 Wallet Screen (My Deals)

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

**Changes:**
- Rename from "Loot" to "My Deals"
- Use SectionList with status groupings
- `WalletDealCard` component with USE NOW action

### 3.4 Redemption Screen

```
┌──────────────────────────────────────────┐
│  < Back                                  │
├──────────────────────────────────────────┤
│  Show to Cashier                         │
│       [QR CODE]                          │
│    SafeColor™ ID:                        │
│    [   🟢   PULSING GREEN   🟢   ]       │  <- Contained card (not full-screen)
│    Café Milano • -15 DH                  │
│          [ Cancel Redemption ]           │
└──────────────────────────────────────────┘
```

**Changes:**
- Remove full-screen pulsing green background
- Rename "Visual Handshake" → "SafeColor Verification"
- Contained color display card
- Professional, banking-app feel

### 3.5 Profile Screen

```
┌──────────────────────────────────────────┐
│  Profile                       [Edit]    │
├──────────────────────────────────────────┤
│  ( Photo )   Youssef Benali              │
│              Gold Member 🌟              │  <- Membership tier, not "Hunter Rank"
│  ┌────────────────────────────────────┐  │
│  │  TOTAL SAVINGS: 420 DH             │  │  <- SavingsCard
│  └────────────────────────────────────┘  │
│  YOUR ACTIVITY                           │
│  • Deals Redeemed: 12                    │
│  • Current Streak: 4 Days 🔥             │
│  SETTINGS                                │
│  > Notifications                    >    │
│  > Location Privacy                 >    │
└──────────────────────────────────────────┘
```

**Changes:**
- "Gold Member" instead of "Hunter Rank"
- "Total Savings" instead of "Points/Loot"
- Clean settings rows with chevrons

**Files:** `/docs/prd/PRD-01-buyer-app.md`, `/docs/design/DES-01-component-spec.md`

---

## Phase 4: Terminology Updates (All Docs)

### Master Mapping Table

| Old (Gaming) | New (Utility) | Affected Docs |
|--------------|---------------|---------------|
| Loot | Deal(s) | All |
| Hunt/Hunting | Browse/Discover/Find | All |
| Hunter | Shopper/Member | All |
| Hunter Rank | Membership Tier | PRD-01, BRAND-02 |
| Grid | Map | BRAND-01, PRD-01 |
| Signal | Nearby Deal | BRAND-01, PRD-01 |
| Sector | Area/Neighborhood | BRAND-01 |
| Ping | Check-in/Open | BRAND-01, BRAND-02 |
| Lock (deal) | Save/Claim | BRAND-01 |
| Visual Handshake | SafeColor Verification | GUIDE-03, PRD-01, DES-01 |
| Deal Drop | New Deal Alert | BRAND-01 |
| Flash Deal | Limited-Time Deal | All (keep "Flash" optional) |

### Copy Updates

| Context | Old | New |
|---------|-----|-----|
| App launch | "Scanning network..." | "Finding deals nearby..." |
| Deals found | "3 signals detected" | "3 deals nearby" |
| Deal claimed | "Loot secured" | "Deal claimed!" |
| Success | "Mission complete" | "Saved 45 DH!" |
| Empty state | "Quiet sector" | "No deals here yet" |
| Error | "Lost your signal" | "Can't reach server" |
| Notification | "[PROXIMITY ALERT]" | "Café Atlas - 25% off" |

---

## Phase 5: Document Update Order

### Priority 1 (Foundation - Do First)
1. `/docs/brand/BRAND-01-voice-tone.md` - Complete rewrite
2. `/docs/brand/BRAND-02-gamification.md` - Complete rewrite, rename file
3. `/docs/appendix/APPENDIX-A-glossary.md` - Term updates

### Priority 2 (Design System)
4. `/docs/design/DES-01-component-spec.md` - New color tokens, component specs
5. `/docs/design/DES-03-motion-guidelines.md` - Reduced animation intensity

### Priority 3 (Product Requirements)
6. `/docs/prd/PRD-01-buyer-app.md` - Screen specs, terminology, user stories
7. `/docs/prd/PRD-02-seller-app.md` - Terminology updates

### Priority 4 (Guides)
8. `/docs/guides/GUIDE-01-buyer-app-quickstart.md` - Terminology
9. `/docs/guides/GUIDE-03-visual-handshake.md` - Rename to SafeColor, update copy

### Priority 5 (Operations/Strategy)
10. `/docs/ops/OPS-02-support-scripts.md` - Customer-facing language
11. `/docs/ops/OPS-04-buyer-onboarding.md` - Onboarding copy
12. `/docs/strategy/STRAT-01-executive-summary.md` - Brand positioning

### Priority 6 (Cleanup)
13. Global grep for remaining gaming terms across all `/docs/` files
14. Update `DOCUMENT-TRACKER.md` with new versions

---

## Verification Steps

1. **Terminology Check:** Run `grep -r "Loot\|Hunt\|Grid\|Signal\|Sector" docs/` - should return 0 matches in user-facing content
2. **Color Validation:** Verify all color hex values in DES-01 use new palette
3. **Tier Consistency:** Confirm Bronze/Silver/Gold/Platinum used everywhere
4. **Copy Review:** Sample 10 notifications/messages - all should follow new voice
5. **Screen Specs:** Validate 5 main screens match ASCII mockups in PRD-01

---

## Summary

| Category | Scope | Estimated Changes |
|----------|-------|-------------------|
| Brand docs | 2 files | Complete rewrite |
| Design docs | 2 files | Major updates |
| PRD docs | 2 files | 40-50% rewrite |
| Guide docs | 3 files | Moderate updates |
| Ops/Strategy | 3 files | Minor updates |
| Cleanup | All docs | Terminology pass |

**Total: ~12-15 files require updates, with 2 complete rewrites.**
