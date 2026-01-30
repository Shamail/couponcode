---
document_id: APPENDIX-A
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Documentation Lead
dependencies: []
related_documents:
  - APPENDIX-B
---

# APPENDIX-A: Glossary

## 1. Executive Summary

This glossary defines Frictionless-specific terminology used across strategy, product, technical, and operational documentation. Use it as the authoritative reference for shared language across teams.

## 2. Glossary Terms

| Term | Definition |
| --- | --- |
| Activity Points | Points awarded to buyers for actions such as claiming or redeeming deals |
| Activation | A buyer completes onboarding and views the first deal |
| Active Deal | A deal currently visible and redeemable in the Buyer app |
| Active Merchant | A merchant that has broadcast or redeemed a deal in the past 30 days |
| Anon Hash | An anonymized user identifier used in location events |
| API Contract | The documented input/output format for backend endpoints |
| Base Radius | Default discovery radius in the Buyer app (typically 500m) |
| Beaconless | No hardware required for redemption or discovery |
| Buyer | End user who browses and redeems deals |
| Buyer App | Consumer-facing mobile application |
| Catchment Area | The effective trading radius around a merchant |
| Claim | The buyer action that reserves a deal for redemption |
| Cold Start | Initial city launch challenge with no users or merchants |
| Conversion | A deal claim that results in a successful redemption |
| New Deal Alert | A deal broadcast sent to nearby buyers |
| Deal Pulse | Animation indicating deal presence on the map |
| Deal TTL | Time-to-live for an active deal |
| Density Cell | A heatmap cell representing aggregated foot traffic |
| Discovery Loop | The repeated flow: open app → discover → claim → redeem |
| FOMO | Fear of missing out used to increase urgency |
| Limited-Time Deal | Time-limited promotion broadcast to nearby users |
| Foot Traffic | Aggregate movement of buyers in a defined area |
| Geo-fence | A geographic boundary used for eligibility or visibility |
| Geo-hash | Encoded location value used for indexing |
| Location Check-in | A location update sent from the Buyer app |
| Heatmap | Visual overlay showing real-time activity density |
| Heatmap Zone | A colored region derived from density cells |
| Hono | Lightweight web framework used for API services |
| Incentive Loop | Reward cycle that motivates repeated usage |
| Deal | A deal or offer available in the Buyer app |
| Deal Redeemed | UI state after successful redemption |
| Mapbox | Mapping SDK used in the Buyer and Seller apps |
| MAU | Monthly Active Users |
| Merchant | Store owner or operator using the Seller app |
| Merchant Success | Operations function focused on merchant outcomes |
| Net Retention | Revenue retained from merchants after upgrades/churn |
| North Star Metric | Primary metric indicating platform success |
| OTP | One-time password used for login or verification |
| Pay-per-Result | Monetization model charging only upon redemption |
| Personal Data | Any data that can identify a person (directly or indirectly) |
| PostGIS | PostgreSQL extension enabling spatial queries |
| Product-Market Fit | Evidence of strong demand and retention |
| Pro Tier | Paid merchant subscription tier with premium features |
| Pulse Radius | Visual ring effect around an active deal |
| QR Rotation | Security measure with rotating QR codes |
| Redemption | Successful verification of a claimed deal |
| Redemption Rate | Redemptions divided by claims |
| Referral Loop | User acquisition via sharing and rewards |
| Retention | Continued usage over time (D7, D30, etc.) |
| Seller App | Merchant-facing mobile application (Shadow) |
| Shadow | Codename for Seller app |
| Proximity Indicator | Visual indicator of proximity to a deal |
| Spatial Index | PostGIS index for geospatial lookup performance |
| SST | Serverless Stack for infrastructure as code |
| Store Pin | Map marker representing a merchant location |
| Take Rate | Percentage of GMV captured as revenue |
| Urban Corridor | Dense shopping area targeted for launch |
| SafeColor Verification | The QR + color code redemption protocol |
| Wallet Share | Percentage of a buyer's spend captured by merchants |

## 3. Related Documents

**Dependencies**
- None

**Related Specs**
- APPENDIX-B: Section 2

**Implementation Guides**
- GUIDE-01: Section 2
- GUIDE-02: Section 2

## 4. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Documentation Lead | Updated terminology for rebrand |
| 1.0 | 2026-01-30 | Documentation Lead | Initial glossary |
