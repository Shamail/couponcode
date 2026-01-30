---
document_id: META-README
version: 1.3
status: Final
priority: P0
last_updated: 2026-01-30
owner: Documentation Lead
dependencies: []
related_documents:
  - META-TRACKER
  - META-GAP
---

# Documentation Hub (Frictionless)

## 1. Executive Summary

This repository contains the authoritative product, technical, business, and operational documentation for the Frictionless platform. It is designed to be navigable by role, task, and system area so teams can move from strategy to execution without hunting for context.

Use this README as the master index. Each document includes standardized metadata, numbered sections, and cross-references for fast discovery. If you are new to the product, start with STRAT-01 and the Buyer/Seller PRDs, then drill into technical and operational details as needed.

## 2. Directory Structure

```
/docs
  /appendix        Glossary and source citations
  /brand           Brand voice, tone, and membership
  /business        Monetization and unit economics
  /data            Canonical schema dictionary
  /design          Component system, accessibility, motion
  /guides          End-user and merchant quickstarts
  /implementation  Sprint plans, CI/CD, delivery
  /market          Market research and competitive analysis
  /metrics         KPIs, dashboards, analytics events
  /ops             Onboarding, support, incident response
  /prd             Product requirements (buyer & seller)
  /strategy        Vision, market thesis
  /tech            Architecture, data, security, deployment
  /threads         Cross-cutting system narratives
  /_templates      Standard document templates
```

## 3. Document Inventory

| ID | Title | Category | Priority | Status | Description |
| --- | --- | --- | --- | --- | --- |
| STRAT-01 | Executive Summary | strategy | P0 | Final | Strategic vision, product thesis, north star metrics |
| STRAT-02 | Market Analysis | strategy | P0 | Final | Morocco market narrative, simulation, GTM framing |
| PRD-01 | Buyer App | prd | P0 | Final | Buyer app requirements and user stories |
| PRD-02 | Seller App (Shadow) | prd | P0 | Final | Seller app requirements and workflows |
| TECH-00 | System Architecture | tech | P0 | Final | High-level infrastructure blueprint |
| ADR-001 | Stack Decisions | adr | P1 | Final | Postgres + PostGIS decision rationale |
| TECH-01 | Neon Database Schema | tech | P0 | Final | Postgres/PostGIS schema and modeling |
| DATA-01 | Schema Dictionary | data | P0 | Final | Canonical validation constraints and constants |
| TECH-02 | Location Ingestion | tech | P1 | Final | GPS pipeline, ingestion API, privacy rules |
| TECH-03 | Heatmap Generation | tech | P1 | Final | Spatial aggregation and heatmap API |
| TECH-04 | Redemption Security & QR Protocol | tech | P0 | Final | SafeColor Verification security model |
| TECH-05 | Mobile Stack Architecture | tech | P1 | Final | Expo/Mapbox/TanStack standards |
| TECH-06 | API Contract Specification | tech | P0 | Final | Endpoint contracts and schemas |
| TECH-07 | Deployment & Environments | tech | P0 | Final | SST deploy flow, AWS setup, secrets |
| TECH-08 | Monitoring & Observability | tech | P1 | Final | Logging, metrics, alerts, SLOs |
| TECH-09 | Database Migrations | tech | P1 | Final | Migration strategy and rollback |
| TECH-10 | Security & Compliance | tech | P1 | Final | CNDP alignment, privacy controls |
| TECH-11 | Failure Modes & Resilience | tech | P1 | Final | Runbooks for outages and latency |
| DES-01 | Component Specifications | design | P1 | Final | Core UI components and map system |
| DES-02 | Accessibility Guidelines | design | P0 | Final | WCAG-aligned mobile accessibility |
| DES-03 | Motion Guidelines | design | P1 | Final | Animation standards and transitions |
| DES-04 | Design Tokens | design | P0 | Final | Single source of truth for tokens |
| DES-05 | Design Principles | design | P1 | Final | Philosophy and decision framework |
| DES-06 | Layout System | design | P1 | Final | Grid, spacing, safe areas, RTL |
| DES-07 | Iconography | design | P1 | Final | UI icon standards and usage |
| DES-08 | Component States | design | P0 | Final | Interactive states across UI |
| DES-09 | Theming Architecture | design | P1 | Final | Dark mode and theme structure |
| DES-10 | Core Components | design | P0 | Final | Buttons, inputs, selection controls |
| DES-11 | Container Components | design | P1 | Final | Cards, sheets, modals |
| DES-12 | Navigation Components | design | P1 | Final | Tab bar, headers, search |
| DES-13 | Feedback Components | design | P0 | Final | Toasts, alerts, loading |
| DES-14 | Data Display Components | design | P1 | Final | Lists, badges, avatars |
| DES-15 | Form Patterns | design | P1 | Final | Validation and form flows |
| DES-16 | Navigation Patterns | design | P2 | Final | User flow patterns |
| DES-17 | Map Patterns | design | P1 | Final | Map interactions and overlays |
| DES-18 | Localization Patterns | design | P1 | Final | RTL and multilingual UI |
| DES-19 | Offline & Degraded Patterns | design | P2 | Final | Offline behavior and recovery |
| BRAND-00 | Brand Overview & Manifesto | brand | P0 | Final | Mission, vision, positioning |
| BRAND-01 | Voice & Tone Guide | brand | P1 | Final | Brand language and tone pillars |
| BRAND-02 | Membership & Rewards | brand | P1 | Final | Membership tiers and reward design |
| BRAND-03 | Logo & Marks | brand | P0 | Final | Logo system and usage |
| BRAND-04 | Color System | brand | P0 | Final | Full palette and accessibility |
| BRAND-05 | Typography | brand | P1 | Final | Type families and scale |
| BRAND-06 | Iconography | brand | P1 | Final | Brand icon system |
| BRAND-07 | Imagery & Photography | brand | P2 | Final | Photo direction and usage |
| BRAND-08 | Map Visual Identity | brand | P1 | Final | Map styling and marker identity |
| BRAND-09 | Localization Guidelines | brand | P1 | Final | Arabic/French/RTL guidance |
| BRAND-10 | Brand Applications | brand | P2 | Final | App store, social, merchant assets |
| BRAND-11 | Asset Library Index | brand | P2 | Final | Asset structure and governance |
| BIZ-01 | Monetization Model | business | P0 | Final | Revenue model and pricing |
| BIZ-02 | Unit Economics | business | P1 | Final | LTV, CAC, margin model |
| BIZ-03 | GTM Strategy | business | P1 | Final | Launch phases, channels, partnerships |
| BIZ-04 | Fraud Vectors | business | P1 | Final | Fraud detection and mitigation strategy |
| OPS-01 | Merchant Onboarding Playbook | ops | P0 | Final | Field onboarding scripts and flow |
| OPS-02 | Support Scripts & Troubleshooting | ops | P1 | Final | Support playbooks and escalation |
| OPS-03 | Incident Response | ops | P1 | Final | Severity model and runbooks |
| OPS-04 | Buyer Onboarding | ops | P2 | Final | Growth ops and activation |
| OPS-05 | Compliance Automation | ops | P1 | Final | CNDP workflows and DSR automation |
| IMPL-01 | Sprint Plan | implementation | P0 | Final | 6-sprint roadmap |
| IMPL-02 | CI/CD Pipeline | implementation | P1 | Final | GitHub Actions and release flow |
| MKT-01 | Competitive Analysis | market | P1 | Final | Competitor matrix and positioning |
| MKT-02 | User Research | market | P1 | Final | Personas, JTBD, research plan |
| MKT-03 | Morocco Retail Trends | market | P2 | Final | Market drivers and seasonal trends |
| METRICS-01 | KPI Framework | metrics | P1 | Final | North Star and KPI definitions |
| METRICS-02 | Dashboards | metrics | P2 | Final | Dashboard specs and alerting |
| METRICS-03 | Analytics Events | metrics | P1 | Final | Event taxonomy and properties |
| GUIDE-01 | Buyer App Quickstart | guides | P0 | Final | End-user onboarding and redemption |
| GUIDE-02 | Seller App Quickstart | guides | P0 | Final | Merchant onboarding and deal setup |
| GUIDE-03 | SafeColor Verification | guides | P1 | Final | QR redemption walkthrough |
| THREAD-01 | The Life of a Check-in | threads | P1 | Final | End-to-end data lifecycle |
| THREAD-02 | The Redemption Atomic Unit | threads | P0 | Final | Redemption flow mapping |
| THREAD-03 | Deal Lifecycle | threads | P1 | Final | Create -> discover -> redeem |
| THREAD-04 | Privacy & Compliance Lifecycle | threads | P1 | Final | Consent -> deletion lifecycle |
| THREAD-05 | Resilience -> Incident -> Recovery | threads | P1 | Final | Monitoring to recovery chain |
| APPENDIX-A | Glossary | appendix | P0 | Final | Frictionless-specific terminology |
| APPENDIX-B | Sources | appendix | P2 | Final | Research citations |
| META-README | Documentation Hub | meta | P0 | Final | Master navigation and standards |
| META-TRACKER | Document Tracker | meta | P1 | Final | Progress and ownership tracker |
| META-GAP | Gap Analysis | meta | P2 | Final | Coverage analysis and next gaps |

## 4. Role-Based Quick Reference

**Engineering**
- Start: TECH-00, TECH-06, TECH-07
- Data: TECH-01, TECH-02, TECH-03, TECH-09, DATA-01
- Security: TECH-04, TECH-10
- Quality: IMPL-02, TECH-08
- Threads: THREAD-01, THREAD-02, THREAD-05

**Product Management**
- Strategy: STRAT-01, STRAT-02
- Requirements: PRD-01, PRD-02
- Metrics: METRICS-01, METRICS-03
- GTM: BIZ-03, OPS-04
- Threads: THREAD-03, THREAD-04

**Design**
- System foundations: DES-04, DES-06, DES-08
- Components: DES-01, DES-10, DES-13, DES-14
- Patterns: DES-15, DES-17, DES-18, DES-19
- Accessibility & motion: DES-02, DES-03
- Brand: BRAND-00 to BRAND-11

**Operations**
- Merchant ops: OPS-01, OPS-02
- Incident response: OPS-03
- Buyer onboarding: OPS-04
- Support safety: TECH-10
- Compliance: OPS-05

**Leadership**
- Strategy: STRAT-01, STRAT-02
- Business: BIZ-01, BIZ-02, BIZ-03
- Metrics: METRICS-01

## 5. Task-Based Navigation

| Task | Start Here | Then | Output |
| --- | --- | --- | --- |
| Launch a new city | BIZ-03 | OPS-04, OPS-01 | City launch playbook |
| Implement heatmap | TECH-03 | TECH-01, TECH-02 | Heatmap API + map layer |
| Ship a new deal type | PRD-01 | TECH-06, METRICS-03 | Updated API + analytics |
| Handle incident | OPS-03 | TECH-08, TECH-10 | Restoration + postmortem |
| Investigate failure mode | TECH-11 | THREAD-05, TECH-08 | Runbook + mitigation |
| Improve onboarding | GUIDE-01 | METRICS-01, OPS-04 | Activation uplift plan |
| Process DSR request | OPS-05 | TECH-10, OPS-03 | CNDP-compliant deletion |

## 6. Finding Information

| Need | Document |
| --- | --- |
| Vision and narrative | STRAT-01 |
| Market landscape | STRAT-02, MKT-01, MKT-03 |
| Buyer app behavior | PRD-01, GUIDE-01 |
| Seller operations | PRD-02, OPS-01, GUIDE-02 |
| Security and privacy | TECH-04, TECH-10 |
| Data constraints | DATA-01 |
| System threads | THREAD-01 to THREAD-05 |
| Analytics events | METRICS-03 |
| Deployment & ops | TECH-07, TECH-08, OPS-03 |

## 7. Metadata Standards

All docs use YAML frontmatter with the following required fields:

```yaml
---
document_id: [CATEGORY]-[NUMBER]
version: 1.0
status: Final
priority: P0 | P1 | P2
last_updated: YYYY-MM-DD
owner: [Role Title]
dependencies: [list of document IDs]
related_documents: [list of document IDs]
---
```

## 8. Naming Conventions

- Format: `CATEGORY-XX-descriptive-title.md`
- Category prefixes: STRAT, PRD, TECH, DES, BRAND, BIZ, OPS, IMPL, MKT, METRICS, GUIDE, APPENDIX, META, DATA, THREAD, ADR
- Use two-digit numbering for ordering (except appendix letters and meta)

## 9. Maintenance Protocols

- **Monthly:** Metrics, dashboards, analytics events
- **Quarterly:** Market, business, strategy
- **Release-based:** Technical specs, PRDs, guides
- **Post-incident:** OPS-03 and TECH-08 updates

Ownership and review cadence should be captured in META-TRACKER.

## 10. Related Documents

**Dependencies**
- META-TRACKER: Section 1
- META-GAP: Section 1

**Related Specs**
- APPENDIX-A: Section 1
- APPENDIX-B: Section 1

**Implementation Guides**
- IMPL-02: Section 2

## 11. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.3 | 2026-01-30 | Documentation Lead | Added brand and design system expansion |
| 1.2 | 2026-01-30 | Documentation Lead | Updated rebrand terminology and titles |
| 1.1 | 2026-01-30 | Documentation Lead | Added data, ADRs, and thread inventory |
| 1.0 | 2026-01-30 | Documentation Lead | Initial navigation hub and standards |
