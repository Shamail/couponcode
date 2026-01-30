---
document_id: GUIDE-03
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Product Marketing Lead
dependencies:
  - TECH-04
related_documents:
  - GUIDE-01
  - GUIDE-02
  - OPS-02
---

# GUIDE-03: SafeColor Verification (QR Redemption)

## 1. Executive Summary

SafeColor Verification is Frictionless' secure, hardware-free redemption protocol. It combines rotating QR codes with a color-matching confirmation to ensure fast, reliable in-store verification.

This guide explains how the flow works for both buyers and merchants and how to handle common issues.

## 2. Buyer Flow

1. Tap **Claim Deal** in the Buyer app
2. A QR code appears with a matching SafeColor indicator
3. Present the QR to the merchant within the countdown window

## 3. Merchant Flow

1. Open **Redeem** in the Seller app
2. Scan the buyer's QR code
3. Confirm the color match on both screens
4. Tap **Confirm** to complete redemption

## 4. Why It Is Secure

- QR tokens rotate every 60 seconds
- Tokens are signed server-side
- Color confirmation blocks replay attacks
- Redemptions are verified in real time

## 5. Troubleshooting

| Issue | Likely Cause | Fix |
| --- | --- | --- |
| Scan fails | Low brightness or glare | Increase screen brightness |
| Color mismatch | Expired QR | Ask buyer to refresh QR |
| Delay after scan | Poor network | Retry on stable connection |

## 6. Related Documents

**Dependencies**
- TECH-04: Section 2

**Related Specs**
- GUIDE-01: Section 6
- GUIDE-02: Section 5

**Implementation Guides**
- OPS-02: Section 2

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Product Marketing Lead | Renamed to SafeColor Verification |
| 1.0 | 2026-01-30 | Product Marketing Lead | Initial redemption guide |
