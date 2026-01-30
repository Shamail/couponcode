---
document_id: DES-16
version: 1.0
status: Final
priority: P2
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-12
related_documents:
  - PRD-01
  - PRD-02
  - DES-17
---

# DES-16: Navigation Patterns

## Executive Summary

Navigation patterns define end-to-end flows for discovery, claiming, and redemption, ensuring predictable movement across the product.

---

## 1. Buyer App Core Flows

### Discover → Claim → Redeem
1. Home map shows nearby deals
2. Tap marker → deal bottom sheet
3. Tap “Claim Deal” → wallet entry
4. Redeem in-store with QR + color code

### Wallet → History
- Active deals show first
- Redeemed and expired in history

---

## 2. Seller App Core Flows

### Create Deal → Publish
1. Deal form
2. Preview card
3. Publish confirmation

### Redemption Validation
1. Scan QR
2. Confirm color code
3. Success/failure feedback

---

## 3. Deep Linking

- Support links to deal detail (`/deal/:id`)
- Expired deals redirect to nearest active

---

## 4. Transition Guidance

- Map → Sheet: slide up
- Sheet → Full screen: fade + slide
- Success states: overlay with dimmed background

---

## Related Documents

**Dependencies**
- DES-12: Section 1

**Related Specs**
- PRD-01: Section 3
- PRD-02: Section 3
- DES-17: Section 2

**Implementation Guides**
- GUIDE-01: Section 2

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial navigation pattern definitions |
