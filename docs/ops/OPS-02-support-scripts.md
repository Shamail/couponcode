---
document_id: OPS-02
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Support Lead
dependencies:
  - OPS-01
  - TECH-04
related_documents:
  - OPS-03
  - GUIDE-03
---

# OPS-02: Support Scripts & Troubleshooting Guide

**Version:** 1.0
**Last Updated:** January 2026
**Audience:** Customer Support Team, Field Operations, Merchant Success Managers

---

## Executive Summary

This document provides support scripts and troubleshooting guidance for common merchant and buyer issues. It is designed for fast resolution and clear escalation thresholds.

Use it as a live runbook during customer support interactions.

## Overview

This document provides word-for-word scripts for handling common merchant support scenarios. Each script includes:

- **The Scenario:** What the merchant is experiencing
- **Root Causes:** Technical reasons this happens
- **The Script:** Exactly what to say
- **Resolution Steps:** How to fix it
- **Escalation Criteria:** When to involve engineering

---

## Scenario 1: "The QR Code Won't Scan"

### The Scenario

A merchant is trying to redeem a customer's deal. They open the scanner, point it at the customer's phone, but nothing happens. The customer is waiting. The merchant is frustrated.

### Root Causes

| Cause | Likelihood | Fix Difficulty |
|-------|------------|----------------|
| Camera permission denied | 40% | Easy |
| Poor lighting conditions | 25% | Easy |
| Screen brightness too low (customer) | 15% | Easy |
| QR code expired | 10% | N/A (expected behavior) |
| Camera hardware issue | 5% | Escalate |
| App bug | 5% | Escalate |

### The Script

**Opening (Empathize + Assure):**

> "I understand—this is frustrating, especially with a customer waiting. Let's fix this right now. I'm going to walk you through a few quick checks. Most of the time, this is a simple permission or lighting issue."

**Step 1: Check Camera Permission**

> "First, let's make sure the app can use your camera. Can you close the Frictionless app completely?
>
> Now, go to your phone's Settings.
>
> [For iPhone]: Scroll down and tap 'Frictionless Seller.' Do you see 'Camera' listed? What does it say next to it?"
>
> [For Android]: Tap 'Apps' or 'Applications,' find 'Frictionless Seller,' tap it, then tap 'Permissions.' Is Camera turned on?"

**If permission is OFF:**

> "There it is! Camera is turned off. Just tap that toggle to turn it on. Perfect. Now let's go back to the app and try scanning again."

**If permission is already ON, proceed to Step 2.**

**Step 2: Check Lighting**

> "Okay, camera permission looks good. Let's check the lighting. Where are you trying to scan right now? Is it a dark area of the store?"

**If lighting is poor:**

> "QR codes need decent light to scan properly. Can you ask the customer to step closer to a light source? Or if you have a lamp nearby, that works too. Even turning on your phone's flashlight while scanning can help—tap the flash icon in the scanner if you see one."

**If lighting seems fine, proceed to Step 3.**

**Step 3: Check Customer's Screen**

> "Now let's check the customer's phone. Is the QR code showing clearly on their screen? Ask them to:
>
> 1. Turn their screen brightness all the way up
> 2. Remove any screen protector glare if there is one
> 3. Hold the phone steady, about 6 inches from your camera"

**Step 4: Check QR Code Validity**

> "Let me check one more thing. Can you look at the customer's screen? Below the QR code, there should be a timer or an expiration notice. What does it say?"

**If expired:**

> "Ah, that's the issue—the QR code has expired. Our codes are only valid for 30 minutes after the customer claims the deal. The customer would need to go back to the app and claim a new deal, if one is still available. This is a security feature to prevent code sharing."

**Step 5: Try the Color Code Fallback**

> "If the QR still won't scan, you can use the backup verification. Look at the customer's screen—do you see the SafeColor bar or circle around the QR code? It changes every few seconds. In your app, tap 'Manual Verify' and select the color you see on their screen. If it matches our system, it will confirm the redemption."

**Step 6: Force Restart (Last Resort)**

> "Let's do a full restart of the app:
>
> 1. Close the app completely—swipe it away from your recent apps
> 2. Wait 5 seconds
> 3. Open it again
> 4. Try scanning one more time"

### Resolution Confirmation

> "Did that work? Great! Just so you know, about 90% of scanning issues are either camera permissions or lighting. You're now an expert at fixing it. If it ever happens again, remember: check permission, check lighting, check brightness."

### Escalation Criteria

Escalate to Tier 2 / Engineering if:

- [ ] Camera permission is ON, but camera view is black/frozen
- [ ] Same merchant reports scanning issues 3+ times in one week
- [ ] Multiple merchants in the same area report issues simultaneously
- [ ] Customer's valid (non-expired) QR code fails even with manual SafeColor verification

### Escalation Template

```
ESCALATION: QR Scanning Failure
Merchant: [Name]
Phone: [Number]
Device: [iPhone/Android, Model]
OS Version: [Version]
App Version: [Version]

Issue: QR code scanning not working
Steps Attempted:
- [ ] Camera permission verified ON
- [ ] Lighting conditions adequate
- [ ] Customer screen brightness maxed
- [ ] QR code confirmed not expired
- [ ] Color code manual verification attempted
- [ ] App force-restarted

Result: Still not scanning

Additional Notes:
[Any error messages, unusual behavior]
```

---

## Scenario 2: "The Map is Empty"

### The Scenario

A merchant opens their app, looks at the heatmap, and sees... nothing. No dots. No clusters. Just their store pin on an empty map. They're wondering if the app is broken.

### Root Causes

| Cause | Likelihood | Explanation |
|-------|------------|-------------|
| No users in area | 45% | Genuinely no one nearby with the app |
| User data expired | 30% | Data is >30 minutes old |
| Wrong time of day | 15% | Checking during off-peak hours |
| Geofence too small | 5% | Zone doesn't capture foot traffic |
| App data sync issue | 5% | Rare technical glitch |

### The Script

**Opening (Set Expectations):**

> "I hear you—an empty map can be confusing. Let me explain what you're seeing and then we'll figure out what's going on."

**Explain the 30-Minute Expiry:**

> "First, here's something important about how our map works: **we only show user locations from the last 30 minutes.** This is for two reasons:
>
> 1. **Privacy:** We don't track people indefinitely. That would be creepy.
> 2. **Accuracy:** If someone was near your store 2 hours ago, that information is useless to you. They're long gone.
>
> So when you see an empty map, it usually means there haven't been any Frictionless users walking through your zone in the last half hour. That's not a bug—that's the system working correctly."

**Step 1: Check the Time**

> "What time is it right now? And when do you typically get the most foot traffic?"

**If it's off-peak:**

> "That makes sense then. At [current time], there's probably less foot traffic in general. The best times to check the map are during your typical busy periods—lunch rush, after-work hours, weekends. Try checking again during [suggested peak time] and you should see more activity."

**Step 2: Explain User Adoption Context**

> "Here's something else to keep in mind: Frictionless is still growing in Morocco. Not everyone has the Buyer app yet. The dots you see represent people who:
>
> 1. Have downloaded the Frictionless Buyer app
> 2. Have location sharing turned on
> 3. Are currently within your zone
> 4. Have been active in the last 30 minutes
>
> As more people in Casablanca/[city] download the app, you'll see more dots. We're doing marketing campaigns to grow this—the map will get busier over time."

**Step 3: Review Geofence Size**

> "Let's also check your zone. Can you go to Settings → Store Location → View Zone? How big is the area you've drawn?"

**If geofence is very small (<50m):**

> "I see the issue—your zone is quite small. You're only seeing people who walk within about 50 meters of your store. Let's expand that a bit. Tap 'Edit Zone' and draw a larger circle—maybe 150 to 200 meters. That will capture more foot traffic from nearby streets and areas."

**If geofence seems reasonable, proceed to Step 4.**

**Step 4: Manual Refresh**

> "Let's make sure the data is current. Can you pull down on the map screen to refresh? You should see a loading indicator. What do you see now?"

**Step 5: Verify Location Services**

> "One more check—let's make sure your app can see location data. Go to Settings → Privacy → Location Services. Is Frictionless Seller set to 'While Using' or 'Always'?"

**If location is OFF or "Never":**

> "There's the problem! The app needs location permission to know where your store is and fetch nearby user data. Change that to 'While Using the App' and then restart the app."

### Setting Realistic Expectations

> "Let me set some realistic expectations. On a typical day:
>
> - **Early morning (6-9am):** Usually empty. People are commuting, not shopping.
> - **Mid-morning (9-11am):** Light activity starts.
> - **Lunch (11am-2pm):** Peak time in most areas. This is when you should see the most dots.
> - **Afternoon (2-5pm):** Moderate activity.
> - **Evening (5-8pm):** Another peak, especially near residential areas.
> - **Night (8pm+):** Activity drops significantly.
>
> The map isn't meant to always be full. It's meant to show you WHEN there's opportunity. An empty map at 7am is normal. An empty map at 12:30pm on a weekday might mean you need a bigger geofence, or we need more users in your area."

### The Silver Lining Script

> "Here's a way to think about it: when you DO see a red hotspot on an otherwise quiet map, that's a clear indicator. That's the moment to send a limited-time deal. The contrast matters. If the map was always full of dots, you wouldn't know when there's a real opportunity."

### Resolution Confirmation

> "Does that make sense? The map showing your actual surroundings—sometimes busy, sometimes quiet—is exactly what you want. Shall I show you the best times to check based on your store's location?"

### Escalation Criteria

Escalate to Tier 2 / Engineering if:

- [ ] Map is empty even during confirmed high-traffic times (merchant can visually see crowds outside)
- [ ] Map was showing data yesterday but is empty today (regression)
- [ ] Multiple merchants in the same neighborhood report empty maps simultaneously
- [ ] Refresh fails with an error message

### Escalation Template

```
ESCALATION: Empty Heatmap Investigation
Merchant: [Name]
Phone: [Number]
Location: [Address/Coordinates]
Geofence Size: [Approximate radius in meters]

Issue: Heatmap showing no users
Time of Report: [Time and Day]
Expected Activity Level: [Low/Medium/High based on location]

Steps Attempted:
- [ ] Explained 30-minute expiry
- [ ] Checked time of day vs. peak hours
- [ ] Reviewed geofence size
- [ ] Manual refresh attempted
- [ ] Location permissions verified ON

Observations:
- Is there visible foot traffic outside? [Yes/No]
- Was map working previously? [Yes/No/Don't know]
- Other merchants in area affected? [Yes/No/Unknown]

Additional Notes:
[Error messages, unusual behavior, merchant observations]
```

---

## Scenario 3: "The Deal Didn't Send" / "Nobody Received My Limited-Time Deal"

### The Scenario

Merchant sent a limited-time deal, saw the confirmation animation, but claims no customers showed up or that people they talked to said they didn't receive anything.

### Root Causes

| Cause | Likelihood | Explanation |
|-------|------------|-------------|
| No users in zone at that moment | 50% | Broadcast went out but nobody was there |
| Users had notifications off | 20% | They'll see it in-app but not as push |
| Broadcast worked, just low conversion | 15% | Normal behavior, not a bug |
| Geofence misconfigured | 10% | Deal sent to wrong area |
| Actual delivery failure | 5% | Rare technical issue |

### The Script

**Opening:**

> "Let's investigate this together. Can you open the app and go to the chart icon at the bottom? Then tap on 'Today's Broadcasts.' Find the broadcast you're asking about. What numbers do you see?"

**Interpreting the Stats:**

| Stat | What It Means |
|------|---------------|
| **Sent to: 0** | Nobody was in your zone when you sent the deal |
| **Sent to: 50, Viewed: 5** | 50 people got it, only 5 opened notification |
| **Sent to: 50, Viewed: 20, Redeemed: 2** | Working normally, 4% conversion is typical |

**If Sent = 0:**

> "I see the issue—it says 'Sent to: 0.' That means when you sent the deal, there were no Frictionless users in your zone at that exact moment. Remember, the map shows data from the last 30 minutes, but broadcasts only go to people who are there RIGHT NOW.
>
> Next time, wait until you see active dots on the map, then send it immediately. The best results come from sending when you see a fresh hotspot."

**If Sent > 0 but Viewed is low:**

> "Good news—the broadcast was sent to [X] people. That means it worked. But only [Y] viewed it. This usually means most users had their notifications turned off or were busy. They might still see your deal if they open the Frictionless app later. Some deals take a bit to convert."

**If Viewed is decent but Redeemed is 0:**

> "Your broadcast reached [X] people and [Y] actually saw it. That's the system working. The challenge now is conversion—getting people to actually come in. A few things affect this:
>
> - **Is the deal compelling?** '5% off' won't move people. '50% off' will.
> - **Is there friction?** 'Must buy 3 items to get discount' creates hesitation.
> - **Is timing right?** A lunch deal at 4pm won't work.
>
> Try a stronger offer during peak hours and see if conversion improves."

### Resolution Confirmation

> "So to recap: the broadcast did send—we can see it in your stats. The question is optimization. Want me to help you think through a higher-impact deal for your next limited-time deal?"

---

## Scenario 4: "My Location is Wrong on the Map"

### The Scenario

Merchant sees their store pin in the wrong location, or their geofence is covering the wrong area.

### The Script

**Step 1: Re-center the Store Location**

> "Let's fix that. Go to Settings → Store Location → tap 'Edit Location.'
>
> Now, you'll see the map. Don't trust the automatic GPS—it can be off by a block in dense urban areas. Instead:
>
> 1. Zoom in until you can see individual buildings
> 2. Find YOUR building visually
> 3. Drag the pin to your exact entrance
> 4. Tap 'Confirm Location'
>
> Now you may need to redraw your geofence around this new center point."

---

## Quick Reference: Support Decision Tree

```
CUSTOMER SAYS              →  FIRST CHECK
─────────────────────────────────────────────
"QR won't scan"            →  Camera permission
"Map is empty"             →  Time of day + 30min expiry
"Deal didn't send"         →  Check sent-to count in stats
"Location is wrong"        →  Manual pin adjustment
"App won't open"           →  Force close + restart
"Forgot password"          →  OTP reset via phone number
"Can't see my deals"       →  Check deal wasn't already expired
"Customer says code fake"  →  Verify code in system, check expiry
```

---

## Appendix: Frequently Asked Questions

### Q: "Why can't I see exactly where each person is?"

**A:** "For privacy reasons, we show heat clusters, not individual people. You don't need to know that 'Ahmed is at coordinates X'—you just need to know '10 people are near the food court.' This protects user privacy while still giving you actionable information."

### Q: "Can customers see my location?"

**A:** "Customers see your store on the map only after you send a limited-time deal to them. They see your store name, logo, and the deal—not your personal location. And only within your defined zone."

### Q: "What happens if I send a limited-time deal and then close the app?"

**A:** "The broadcast still works! Once you hit 'Send Deal,' the deal is broadcast from our servers. You can close the app. Customers will still receive it, and redemptions will still be tracked. When you open the app again, you'll see all the stats."

### Q: "Can I cancel a limited-time deal broadcast after sending?"

**A:** "Currently, no. Once a broadcast is sent, it's out there. Think of it like sending a text—you can't unsend it. That's why we show a preview before you confirm. Always double-check the deal details before tapping Send Deal."

---

*End of Document*

## Related Documents

**Dependencies**
- OPS-01: Section 2
- TECH-04: Section 2

**Related Specs**
- GUIDE-03: Section 4

**Implementation Guides**
- OPS-03: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Support Lead | Updated SafeColor and limited-time terminology |
| 1.0 | 2026-01-30 | Support Lead | Standardized metadata and cross-references |
