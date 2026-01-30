---
document_id: METRICS-03
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Data Lead
dependencies:
  - METRICS-01
  - TECH-06
related_documents:
  - PRD-01
  - PRD-02
  - TECH-02
  - THREAD-03
---

# METRICS-03: Analytics Events

## 1. Executive Summary

This document defines the analytics event taxonomy for Frictionless. It standardizes naming, required properties, and event sources to ensure consistent reporting across buyer and seller apps.

All events are JSON payloads emitted by client apps or server-side endpoints and ingested into the analytics pipeline.

## 2. Event Naming Standards

- Format: `domain.action`
- Examples: `buyer.opened_app`, `deal.claimed`, `redemption.success`
- All events require: `event_id`, `user_id`, `timestamp`, `app_version`, `device_os`

## 3. Core Events (Buyer)

| Event | Trigger | Required Properties |
| --- | --- | --- |
| buyer.opened_app | App launch | session_id, location_accuracy |
| buyer.opted_in_location | Location permission granted | permission_state |
| deal.viewed | Deal detail opened | deal_id, distance_meters |
| deal.claimed | Buyer taps Claim | deal_id, ttl_seconds |
| redemption.presented | QR shown | deal_id, redemption_id |
| redemption.success | Redemption verified | redemption_id, merchant_id |

## 4. Core Events (Seller)

| Event | Trigger | Required Properties |
| --- | --- | --- |
| seller.opened_app | App launch | session_id |
| deal.created | Deal posted | deal_id, deal_type, radius_meters |
| deal.broadcast | Broadcast initiated | deal_id, price_mad |
| redemption.verified | QR matched | redemption_id, buyer_id |
| heatmap.viewed | Heatmap screen opened | radius_meters |

## 5. Event Payload Example

```json
{
  "event_id": "evt_01HXYZ...",
  "event_name": "deal.claimed",
  "timestamp": "2026-01-30T15:08:22Z",
  "user_id": "anon_123",
  "app_version": "1.4.0",
  "device_os": "ios",
  "properties": {
    "deal_id": "deal_456",
    "distance_meters": 120,
    "ttl_seconds": 60,
    "city": "casablanca"
  }
}
```

## 6. Data Quality Rules

- Reject events missing `event_id` or `timestamp`
- Coerce timestamps to UTC
- Normalize city names to canonical list
- Redact PII before ingestion

## 7. Related Documents

**Dependencies**
- METRICS-01: Section 4
- TECH-06: Section 2

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 4
- TECH-02: Section 5
- THREAD-03: Section 3

**Implementation Guides**
- IMPL-02: Section 3

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Data Lead | Added deal lifecycle thread reference |
| 1.0 | 2026-01-30 | Data Lead | Initial analytics taxonomy |
