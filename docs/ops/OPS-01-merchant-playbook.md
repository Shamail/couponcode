---
document_id: OPS-01
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Operations Lead
dependencies:
  - PRD-02
  - GUIDE-02
related_documents:
  - OPS-02
  - OPS-03
  - TECH-04
---

# OPS-01: Merchant Onboarding Playbook

**Version:** 1.0
**Last Updated:** January 2026
**Audience:** Field Operations Team, Merchant Success Managers

---

## Executive Summary

This playbook guides field and merchant success teams through onboarding a merchant from first contact to their first redemption. It is designed for a 15-minute activation window and emphasizes simplicity, clarity, and immediate value demonstration.

Use this alongside the Seller app quickstart and redemption protocol when training new merchants.

## Overview

This playbook guides operations staff through the complete merchant onboarding process for Frictionless. The goal is to get a merchant from zero to broadcasting their first deal in under 15 minutes.

**Key Principle:** Zero Friction. If the merchant struggles, we've failed.

---

## Part 1: The Setup

### 1.1 Pre-Visit Checklist

Before visiting the merchant, ensure you have:

- [ ] Merchant's phone number (for OTP verification)
- [ ] Store address (for geofence setup)
- [ ] Business license number (for account verification)
- [ ] Your tablet/phone with the Seller App installed (for demo purposes)

### 1.2 App Installation

#### Step 1: Download the App

**For iOS:**
1. Open the App Store
2. Search for "Frictionless Seller" or scan the QR code (provide printed QR)
3. Tap "Get" → Authenticate with Face ID/Touch ID
4. Wait for installation to complete

**For Android:**
1. Open the Google Play Store
2. Search for "Frictionless Seller" or scan the QR code
3. Tap "Install"
4. Wait for installation to complete

> **Troubleshooting:** If the merchant has storage issues, help them clear space. The app requires ~50MB.

#### Step 2: Grant Permissions

On first launch, the app will request permissions. Guide the merchant through each:

| Permission | Why We Need It | What to Say |
|------------|----------------|-------------|
| **Location** | To center the map on their store | "This lets you see customers near YOUR store, not someone else's." |
| **Camera** | For QR code scanning during redemption | "This is how you'll verify customer coupons—quick scan, done." |
| **Notifications** | For alerts when customers are nearby | "Get notified when there's a crowd forming near you." |

> **Important:** Location must be set to "While Using the App" at minimum. "Always" is not required for the Seller app.

### 1.3 Account Creation

#### Step 1: Phone Verification

1. Tap "Create Account"
2. Enter the merchant's Moroccan phone number (+212 format)
3. Tap "Send Code"
4. Wait for SMS (typically 5-10 seconds)
5. Enter the 6-digit OTP
6. Tap "Verify"

> **Troubleshooting:** If OTP doesn't arrive within 30 seconds, tap "Resend Code." If it still fails, check for SMS blockers or try WhatsApp verification as backup.

#### Step 2: Business Profile

Complete the following fields:

| Field | Instructions |
|-------|--------------|
| **Business Name** | Official store name (as customers know it) |
| **Business Category** | Select from dropdown (Cafe, Restaurant, Retail, etc.) |
| **Business License** | Enter RC number for verification |
| **Owner Name** | Legal name for account recovery |
| **Email** (optional) | For receipts and reports |

#### Step 3: Profile Photo

1. Tap the camera icon
2. Take a photo of the storefront OR upload existing logo
3. Crop to fit the circular frame
4. Tap "Save"

> **Tip:** A good storefront photo helps customers recognize the business when they receive a deal.

---

### 1.4 Defining the Store Geofence

This is the most critical step. The geofence determines which customers appear on the merchant's heatmap.

#### Step 1: Access Geofence Setup

1. From the dashboard, tap the **gear icon** (Settings)
2. Tap **"Store Location"**
3. Tap **"Set My Zone"**

#### Step 2: Center the Map

The map will attempt to center on the device's current location.

1. If the pin is accurate, proceed to Step 3
2. If the pin is wrong:
   - Use two fingers to pan the map
   - Center the blue pin exactly on the store entrance
   - Tap **"Confirm Location"**

#### Step 3: Draw the Geofence

1. Tap **"Draw Zone"**
2. Use your finger to trace around the store's customer catchment area
3. Include:
   - The storefront
   - Nearby sidewalk/pedestrian areas
   - Adjacent parking lot (if applicable)
4. The zone will auto-close when you lift your finger

**Recommended Zone Sizes:**

| Store Type | Radius | Notes |
|------------|--------|-------|
| Small Cafe | 50-100m | Just immediate foot traffic |
| Restaurant | 100-200m | Include nearby streets |
| Shopping Mall Kiosk | 200-300m | Cover mall common areas |
| Large Retail | 300-500m | Include parking and approaches |

#### Step 4: Confirm and Save

1. Review the highlighted zone on the map
2. Tap **"Save Zone"**
3. A confirmation message appears: "Your zone is active"

> **Important:** Explain to the merchant that they can resize this zone anytime. Bigger isn't always better—a huge zone shows too many people they can't realistically reach.

---

## Part 2: The "Limited-Time" Drill

### 2.1 Understanding the Heatmap

Before teaching the drill, ensure the merchant understands what they're looking at.

#### What the Colors Mean

| Color | Meaning | Action |
|-------|---------|--------|
| **Blue dots** | 1-2 people in that spot | Low priority |
| **Yellow cluster** | 3-5 people nearby | Worth watching |
| **Orange cluster** | 6-10 people | Good opportunity |
| **Red hotspot** | 10+ people concentrated | SEND NOW |

#### The 30-Second Refresh

Explain to the merchant:

> "The map refreshes every 30 seconds. These aren't live location check-ins—it's a snapshot. If you see a hotspot, act fast. Those people might move on."

### 2.2 The Limited-Time Drill: Step-by-Step

Practice this sequence with the merchant 3 times until they can do it in under 10 seconds.

#### The Scenario

"It's 12:30pm. You glance at your phone. You see a red hotspot 100 meters away—looks like people gathering near the food court."

#### The Drill

**Step 1: Spot** (2 seconds)
- Open the app (or it's already open in background)
- Look at the heatmap
- Identify the hottest cluster

**Step 2: Tap** (1 second)
- Tap the **"+" button** (bottom center, can't miss it)
- This opens the Quick Deal composer

**Step 3: Select** (3 seconds)
- Choose a pre-configured deal from your templates:
  - "20% off any coffee"
  - "Buy 1 Get 1 Free"
  - "Free dessert with meal"
- Or tap "Custom" to type something specific

**Step 4: Broadcast** (2 seconds)
- Review the preview card
- Tap **"FLASH IT"**
- Watch the pulse animation confirm broadcast

**Step 5: Prepare** (Immediately after)
- The deal is now live for 30 minutes
- Customers in your zone will see it
- Get ready for foot traffic

### 2.3 Deal Templates Setup

Before leaving the merchant, help them create 3-5 deal templates:

1. Tap **Settings** → **"My Deals"**
2. Tap **"+ New Template"**
3. Fill in:
   - **Title:** Short, punchy (e.g., "Happy Hour Special")
   - **Description:** What they get (e.g., "50% off all drinks 3-5pm")
   - **Discount Type:** Percentage / Fixed Amount / BOGO / Freebie
   - **Value:** The discount amount
4. Tap **"Save Template"**

**Recommended starter templates:**

| Template Name | Best Used When |
|---------------|----------------|
| "Lunchtime Rush" | 11am-2pm, attracts workers |
| "Afternoon Slump" | 3-5pm, slow period boost |
| "Closing Time" | Last 2 hours, clear inventory |
| "New Customer Welcome" | Anytime, low-value trial offer |

### 2.4 Reading the Results

After a limited-time deal, teach the merchant to check results:

1. Tap the **chart icon** (bottom nav)
2. View "Today's Broadcasts"
3. For each broadcast, see:
   - **Sent to:** Number of users who received it
   - **Viewed:** How many opened the notification
   - **Redeemed:** How many actually came in

> **Expectation Setting:** A 5-10% redemption rate is excellent. If 100 people see your deal and 8 show up, that's 8 customers who weren't coming otherwise.

---

## Part 3: First Week Check-In Points

### Day 1 (Onboarding Day)
- [x] App installed
- [x] Account created
- [x] Geofence defined
- [x] 3+ deal templates created
- [x] Completed 3 limited-time drills
- [ ] Merchant broadcasts their FIRST real deal (with your supervision)

### Day 2 (Phone Check-In)
- Call the merchant
- Ask: "Did you limited-time anything yesterday after I left?"
- If no: Encourage them to try during their next slow period
- If yes: "How did it go? Any questions?"

### Day 7 (In-Person Follow-Up)
- Visit the store
- Review their dashboard together
- Check:
  - [ ] Total broadcasts sent
  - [ ] Total redemptions
  - [ ] Any support issues
- Adjust geofence if needed
- Add more deal templates based on what worked

---

## Appendix A: Quick Reference Card

Print and leave this with the merchant:

```
╔═══════════════════════════════════════════╗
║         FRICTIONLESS FLASH GUIDE          ║
╠═══════════════════════════════════════════╣
║                                           ║
║  1. LOOK at the map                       ║
║  2. SEE the red dots                      ║
║  3. TAP the + button                      ║
║  4. PICK a deal                           ║
║  5. HIT "Send Deal"                              ║
║                                           ║
║  That's it. Customers incoming.           ║
║                                           ║
╠═══════════════════════════════════════════╣
║  Support: +212 XXX-XXXXXX                 ║
║  WhatsApp: Same number                    ║
╚═══════════════════════════════════════════╝
```

---

## Appendix B: Onboarding Completion Checklist

Use this checklist before leaving the merchant site:

```
MERCHANT: _______________________
DATE: _______________________
OPS AGENT: _______________________

[ ] App installed successfully
[ ] All permissions granted
[ ] Phone number verified
[ ] Business profile complete
[ ] Store photo uploaded
[ ] Geofence drawn and saved
[ ] At least 3 deal templates created
[ ] Limited-time drill completed 3x
[ ] Merchant can independently:
    [ ] Open app and read heatmap
    [ ] Broadcast a deal in <10 seconds
    [ ] Check redemption stats
[ ] Support contact saved in merchant's phone
[ ] Follow-up call scheduled for Day 2

NOTES:
_________________________________
_________________________________
_________________________________

Signature (Merchant): _________________
Signature (Ops Agent): ________________
```

---

*End of Document*

## Related Documents

**Dependencies**
- PRD-02: Section 3
- GUIDE-02: Section 2

**Related Specs**
- TECH-04: Section 2
- OPS-02: Section 2

**Implementation Guides**
- OPS-03: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Operations Lead | Updated limited-time broadcast terminology |
| 1.0 | 2026-01-30 | Operations Lead | Standardized metadata and cross-references |
