---
document_id: THREAD-02
version: 1.1
status: Final
priority: P0
last_updated: 2026-01-30
owner: Documentation Lead
dependencies:
  - PRD-01
  - PRD-02
  - TECH-04
  - TECH-06
  - TECH-01
  - DES-01
related_documents:
  - DATA-01
  - OPS-01
  - GUIDE-03
---

# THREAD-02: The Redemption Atomic Unit

## 1. Executive Summary

This thread maps the redemption flow across Buyer UI, Seller scanner UI, API verification, and Postgres constraints. It is the canonical sequence for the SafeColor Verification and should be consulted whenever redemption logic changes.

## 2. Thread Map (End-to-End)

```
Buyer QR Display
  -> Seller Scanner
    -> API Verify (/redeem)
      -> DB Transaction
        -> Success/Failure UI
```

## 3. State Mapping

| Component | Primary Source | Critical Spec |
| --- | --- | --- |
| QR display | PRD-01 | 60s JWT rotation |
| Color code | TECH-04 | Deterministic from token hash |
| Scanner UI | DES-01 | Square viewport + corner guides |
| API verify | TECH-06 | Errors: EXPIRED_TOKEN, ALREADY_REDEEMED |
| Transaction | TECH-01 | BEGIN/COMMIT, UNIQUE(user_id, deal_id) |
| Success UI | DES-01 | Checkmark + haptic + particle burst |

## 4. Security Checkpoints

- **Replay prevention**: `jti` tracked in `used_tokens` table (TECH-04).
- **Rate limit**: 10 verifications/min per seller (TECH-06).
- **Location constraint**: Redemption radius checked against store location (TECH-06).

## 5. Offline Fallback (Seller)

If connectivity fails, use the offline protocol defined in TECH-04:

- Signed offline token, 24h expiry
- Daily/hourly caps to limit fraud
- Deferred reconciliation via `/redeem/sync`

## 6. Observability Hooks

| Indicator | Source | Alert |
| --- | --- | --- |
| Redemption success rate | METRICS-01 | Alert if <60% in 30 min |
| EXPIRED_TOKEN rate | METRICS-03 | Alert if >5% of attempts |
| Duplicate redemption rate | TECH-01 | Investigate if >1% |

## 7. Related Documents

**Dependencies**
- PRD-01: Section 4
- PRD-02: Section 4
- TECH-04: Section 2
- TECH-06: Section 3
- TECH-01: Section 5
- DES-01: Section 2

**Related Specs**
- DATA-01: Section 2
- GUIDE-03: Section 2

**Implementation Guides**
- OPS-01: Section 4

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Documentation Lead | Updated SafeColor terminology |
| 1.0 | 2026-01-30 | Documentation Lead | Redemption flow thread |
