---
document_id: TECH-10
version: 1.2
status: Final
priority: P1
last_updated: 2026-01-30
owner: Security Lead
dependencies:
  - TECH-00
related_documents:
  - OPS-03
  - APPENDIX-B
  - OPS-05
---

# TECH-10: Security & Compliance

## 1. Executive Summary

This document defines security controls and compliance considerations for Frictionless. It aligns data handling with Morocco's data protection framework and outlines privacy-by-design practices.

Compliance guidance is based on CNDP oversight and Law 09-08 on personal data processing. (See APPENDIX-B: S-03, S-04, S-05)

## 2. Data Classification

| Class | Examples | Handling |
| --- | --- | --- |
| Public | Marketing copy, store names | No restrictions |
| Internal | Aggregated metrics | Limited access |
| Personal | Phone number, precise location | Encryption + strict access |
| Sensitive | Authentication tokens, keys | Restricted + audited |

## 3. Privacy Principles

- **Minimization:** Collect only what is needed for location-driven deals
- **Purpose limitation:** Do not repurpose personal data without consent
- **Anonymization:** Hash user identifiers in analytics and heatmaps
- **Retention limits:** Delete or aggregate after defined periods

## 4. Consent & User Rights

- Location tracking requires explicit opt-in
- Users can request account deletion
- Provide clear in-app privacy disclosures

## 5. Security Controls

| Control | Implementation |
| --- | --- |
| Encryption at rest | Neon-managed encryption |
| Encryption in transit | TLS 1.2+ |
| Secrets management | SST secrets / SSM |
| Access control | Least privilege IAM |
| Audit logging | CloudWatch + app logs |

## 6. Data Retention (Proposed)

| Data Type | Retention | Reason |
| --- | --- | --- |
| Raw GPS check-ins | 30 days | Operational analytics |
| Aggregated heatmap | 12 months | Trend analysis |
| Redemptions | 24 months | Revenue and disputes |
| Support logs | 12 months | Incident analysis |

## 7. Vendor and Processor Review

- **AWS**: Infrastructure hosting
- **Neon**: Database
- **Mapbox**: Maps and tiles
- **Sentry**: Crash reporting

Each vendor should be reviewed for data processing terms and regional hosting implications.

## 8. Incident Response Integration

- Align with OPS-03 severity tiers
- Breach notification protocol should follow CNDP guidance

## 9. Related Documents

**Dependencies**
- TECH-00: Section 2

**Related Specs**
- OPS-03: Section 4
- APPENDIX-B: Section 2
- OPS-05: Section 3
- THREAD-04: Section 3

**Implementation Guides**
- IMPL-02: Section 5

## 10. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.2 | 2026-01-30 | Security Lead | Updated check-in terminology |
| 1.1 | 2026-01-30 | Security Lead | Added compliance automation reference |
| 1.0 | 2026-01-30 | Security Lead | Initial security and compliance guidelines |
