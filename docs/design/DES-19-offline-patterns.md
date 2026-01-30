---
document_id: DES-19
version: 1.0
status: Final
priority: P2
last_updated: 2026-01-30
owner: Design Lead
dependencies:
  - DES-13
related_documents:
  - DES-01
  - TECH-11
  - PRD-01
---

# DES-19: Offline & Degraded Patterns

## Executive Summary

Offline and degraded patterns ensure users can still understand their state and recover gracefully when connectivity is poor. The focus is transparency and clear next steps.

---

## 1. Connectivity States

| State | Criteria | UI Response |
| --- | --- | --- |
| Online | Normal | Full experience |
| Degraded | High RTT / low bandwidth | Lite mode surfaces |
| Offline | No connectivity | Cached list view |

---

## 2. Map Fallback

- Switch to **list-only view** when tiles fail
- Show “Last updated” timestamp
- Allow manual refresh

---

## 3. Content Caching

- Cache last 50 deals with distance
- Cache wallet and active claims
- Expired items marked clearly

---

## 4. Error Recovery

- Provide “Try again” for fetch failures
- Persist user actions locally until sync

---

## Related Documents

**Dependencies**
- DES-13: Section 3

**Related Specs**
- DES-01: Section 1
- TECH-11: Section 2
- PRD-01: Section 4

**Implementation Guides**
- TECH-05: Section 4

## Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Design Lead | Initial offline and degraded patterns |
