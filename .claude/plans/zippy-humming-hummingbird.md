# Documentation Enhancement Plan: Frictionless (Couponcode)

## Executive Summary

This plan enhances Frictionless documentation to match the enterprise-grade quality of the Aura docs. The Aura documentation demonstrates sophisticated patterns: 71 files across 12 categories with standardized metadata, comprehensive navigation, role-based discovery, and rich cross-referencing. Frictionless currently has 18 solid documents but lacks navigation infrastructure, metadata standards, and several content categories.

**Current State:** 18 documents, good content quality, missing navigation/discovery infrastructure
**Target State:** ~45+ documents with full navigation system, standardized metadata, and comprehensive coverage

---

## Phase 1: Navigation Infrastructure (Foundation)

### 1.1 Create Master README.md
**File:** `/docs/README.md`

Create comprehensive navigation hub including:
- Documentation overview and purpose
- Directory structure with descriptions
- Document inventory table (all files with descriptions)
- Role-based quick reference (Engineer, Product Manager, Designer, Operations, Leadership)
- Task-based navigation routes
- "Finding Information" lookup table
- Metadata standards documentation
- Naming conventions
- Maintenance protocols and update cycles

### 1.2 Create Document Tracker
**File:** `/docs/DOCUMENT-TRACKER.md`

- List all existing documents with status
- Track document generation progress
- Map documents to product areas
- Identify coverage gaps by category

### 1.3 Create Gap Analysis
**File:** `/docs/GAP-ANALYSIS.md`

- Coverage analysis by category
- Priority ranking for missing documents
- Recommendations for documentation maintenance

---

## Phase 2: Metadata Standardization

### 2.1 Define Frontmatter Standard
Add standardized YAML frontmatter to ALL 18 existing documents:

```yaml
---
document_id: [CATEGORY]-[NUMBER]
version: 1.0
status: Final
priority: P0 | P1 | P2
last_updated: YYYY-MM-DD
owner: [Role Title]
dependencies: [list of document IDs that should be read first]
related_documents: [list of complementary document IDs]
---
```

### 2.2 Documents to Update (18 files)
| File | Category | Priority |
|------|----------|----------|
| STRAT-01-executive-summary.md | strategy | P0 |
| STRAT-02-market-analysis.md | strategy | P0 |
| PRD-01-buyer-app.md | prd | P0 |
| PRD-02-seller-app.md | prd | P0 |
| TECH-00-architecture.md | tech | P0 |
| TECH-01-neon-schema.md | tech | P0 |
| TECH-02-location-ingestion.md | tech | P1 |
| TECH-03-heatmap.md | tech | P1 |
| TECH-04-redemption-security.md | tech | P0 |
| TECH-05-mobile-stack.md | tech | P1 |
| TECH-06-api-contract.md | tech | P0 |
| DES-01-components.md | design | P1 |
| BRAND-01-voice-tone.md | brand | P1 |
| BRAND-02-gamification.md | brand | P1 |
| BIZ-01-monetization.md | business | P0 |
| OPS-01-merchant-onboarding.md | ops | P0 |
| OPS-02-support-scripts.md | ops | P1 |
| IMPL-01-sprint-plan.md | implementation | P0 |

### 2.3 Add Cross-Reference Sections
Add "Related Documents" section to each document with:
- Dependencies (must-read-first documents)
- Related specs (complementary reading)
- Implementation guides (for specs)

---

## Phase 3: New Content Categories

### 3.1 Market Analysis Category
**Directory:** `/docs/market/`

| Document | Description | Priority |
|----------|-------------|----------|
| MKT-01-competitive-analysis.md | Competitive landscape, positioning matrix | P1 |
| MKT-02-user-research.md | User personas, Jobs-to-be-Done analysis | P1 |
| MKT-03-morocco-retail-trends.md | Market trends, growth opportunities | P2 |

### 3.2 Metrics & Analytics Category
**Directory:** `/docs/metrics/`

| Document | Description | Priority |
|----------|-------------|----------|
| METRICS-01-kpi-framework.md | North Star metric, KPI definitions, targets | P1 |
| METRICS-02-dashboards.md | Dashboard specs, alert definitions | P2 |
| METRICS-03-analytics-events.md | Event tracking plan, data schema | P1 |

### 3.3 Appendix/Reference Category
**Directory:** `/docs/appendix/`

| Document | Description | Priority |
|----------|-------------|----------|
| APPENDIX-A-glossary.md | 50+ Frictionless-specific terms | P0 |
| APPENDIX-B-sources.md | Research citations, references | P2 |

---

## Phase 4: Critical Content Gaps

### 4.1 User-Facing Documentation
**Directory:** `/docs/guides/`

| Document | Description | Priority |
|----------|-------------|----------|
| GUIDE-01-buyer-app-quickstart.md | How to use the buyer app (end users) | P0 |
| GUIDE-02-seller-app-quickstart.md | How to use the seller app (merchants) | P0 |
| GUIDE-03-visual-handshake.md | QR redemption process explained | P1 |

### 4.2 Technical Operations
**New documents in existing directories:**

| Document | Directory | Description | Priority |
|----------|-----------|-------------|----------|
| TECH-07-deployment.md | tech/ | SST deployment, AWS setup, secrets management | P0 |
| TECH-08-monitoring.md | tech/ | Observability, CloudWatch, alerts, on-call | P1 |
| TECH-09-database-migrations.md | tech/ | Schema versioning, migration strategy | P1 |
| IMPL-02-cicd-pipeline.md | implementation/ | GitHub Actions, testing, quality gates | P1 |

### 4.3 Security & Compliance
| Document | Directory | Description | Priority |
|----------|-----------|-------------|----------|
| TECH-10-security-compliance.md | tech/ | CNDP compliance, data retention, privacy policy | P1 |

### 4.4 Operations Expansion
| Document | Directory | Description | Priority |
|----------|-----------|-------------|----------|
| OPS-03-incident-response.md | ops/ | Runbooks, escalation, disaster recovery | P1 |
| OPS-04-buyer-onboarding.md | ops/ | Buyer acquisition, app store optimization | P2 |

### 4.5 Business Expansion
| Document | Directory | Description | Priority |
|----------|-----------|-------------|----------|
| BIZ-02-unit-economics.md | business/ | LTV:CAC calculations, financial projections | P1 |
| BIZ-03-gtm-strategy.md | business/ | Go-to-market plan, launch phases | P1 |

### 4.6 Design Expansion
| Document | Directory | Description | Priority |
|----------|-----------|-------------|----------|
| DES-02-accessibility.md | design/ | WCAG compliance, accessibility requirements | P2 |
| DES-03-motion-guidelines.md | design/ | Animation specs, transitions | P2 |

---

## Phase 5: Template Standardization

### 5.1 Document Templates to Create
Create templates in `/docs/_templates/`:

1. **PRD Template** - Product requirements with acceptance criteria
2. **TECH Template** - Technical specifications with architecture diagrams
3. **OPS Template** - Operational playbooks with step-by-step procedures
4. **GUIDE Template** - User-facing guides with screenshots/diagrams

### 5.2 Standard Section Patterns
All documents should follow:
1. YAML Frontmatter (metadata)
2. H1 Title
3. Executive Summary (2-4 paragraphs)
4. Detailed Sections (numbered H2/H3)
5. Related Documents section
6. Document Control (version history)

---

## Implementation Priority

### P0 - Critical (Week 1)
1. `/docs/README.md` - Navigation hub
2. Add frontmatter to all 18 existing docs
3. `/docs/appendix/APPENDIX-A-glossary.md` - Terminology foundation
4. `/docs/guides/GUIDE-01-buyer-app-quickstart.md`
5. `/docs/guides/GUIDE-02-seller-app-quickstart.md`
6. `/docs/tech/TECH-07-deployment.md`

### P1 - High (Week 2)
7. `/docs/DOCUMENT-TRACKER.md`
8. `/docs/market/MKT-01-competitive-analysis.md`
9. `/docs/market/MKT-02-user-research.md`
10. `/docs/metrics/METRICS-01-kpi-framework.md`
11. `/docs/metrics/METRICS-03-analytics-events.md`
12. `/docs/tech/TECH-08-monitoring.md`
13. `/docs/tech/TECH-09-database-migrations.md`
14. `/docs/tech/TECH-10-security-compliance.md`
15. `/docs/implementation/IMPL-02-cicd-pipeline.md`
16. `/docs/ops/OPS-03-incident-response.md`
17. `/docs/business/BIZ-02-unit-economics.md`
18. `/docs/business/BIZ-03-gtm-strategy.md`
19. `/docs/guides/GUIDE-03-visual-handshake.md`
20. Add cross-references to all documents

### P2 - Medium (Week 3)
21. `/docs/GAP-ANALYSIS.md`
22. `/docs/market/MKT-03-morocco-retail-trends.md`
23. `/docs/metrics/METRICS-02-dashboards.md`
24. `/docs/appendix/APPENDIX-B-sources.md`
25. `/docs/ops/OPS-04-buyer-onboarding.md`
26. `/docs/design/DES-02-accessibility.md`
27. `/docs/design/DES-03-motion-guidelines.md`
28. Create document templates

---

## Document Count Summary

| Category | Current | After Enhancement | New Docs |
|----------|---------|-------------------|----------|
| strategy/ | 2 | 2 | 0 |
| market/ | 0 | 3 | 3 |
| prd/ | 2 | 2 | 0 |
| tech/ | 7 | 11 | 4 |
| design/ | 1 | 3 | 2 |
| brand/ | 2 | 2 | 0 |
| business/ | 1 | 3 | 2 |
| ops/ | 2 | 4 | 2 |
| implementation/ | 1 | 2 | 1 |
| metrics/ | 0 | 3 | 3 |
| appendix/ | 0 | 2 | 2 |
| guides/ | 0 | 3 | 3 |
| Meta files | 0 | 3 | 3 |
| **TOTAL** | **18** | **43** | **25** |

---

## Key Patterns to Adopt from Aura

1. **Philosophy-Driven Headers**: Start major sections with guiding quotes/principles
2. **YAML for Structured Data**: Use YAML blocks for configs, specs, calculations
3. **ASCII Diagrams**: Include text-based architecture diagrams
4. **Tables for Comparisons**: Use markdown tables for matrices and comparisons
5. **Acceptance Criteria**: Use checkbox format `[ ]` for requirements
6. **Blockquotes for Vision**: Use `>` for key value propositions
7. **Numbered Sections**: H2 = 1, 2, 3; H3 = 1.1, 1.2, 2.1
8. **Cross-Reference Format**: "See [DOC-ID]: Section X.X"

---

## Verification Plan

After implementation:
1. Verify README.md provides clear navigation to all documents
2. Verify all 43 documents have standardized frontmatter
3. Verify all documents have Related Documents sections
4. Verify glossary covers all Frictionless-specific terminology
5. Verify user guides are actionable for end users
6. Test cross-references resolve correctly
7. Review documents match the quality bar set by Aura docs

---

## Files to Create (Complete List)

### New Directories
- `/docs/market/`
- `/docs/metrics/`
- `/docs/appendix/`
- `/docs/guides/`
- `/docs/_templates/`

### New Files (25 total)
1. `/docs/README.md`
2. `/docs/DOCUMENT-TRACKER.md`
3. `/docs/GAP-ANALYSIS.md`
4. `/docs/market/MKT-01-competitive-analysis.md`
5. `/docs/market/MKT-02-user-research.md`
6. `/docs/market/MKT-03-morocco-retail-trends.md`
7. `/docs/metrics/METRICS-01-kpi-framework.md`
8. `/docs/metrics/METRICS-02-dashboards.md`
9. `/docs/metrics/METRICS-03-analytics-events.md`
10. `/docs/appendix/APPENDIX-A-glossary.md`
11. `/docs/appendix/APPENDIX-B-sources.md`
12. `/docs/guides/GUIDE-01-buyer-app-quickstart.md`
13. `/docs/guides/GUIDE-02-seller-app-quickstart.md`
14. `/docs/guides/GUIDE-03-visual-handshake.md`
15. `/docs/tech/TECH-07-deployment.md`
16. `/docs/tech/TECH-08-monitoring.md`
17. `/docs/tech/TECH-09-database-migrations.md`
18. `/docs/tech/TECH-10-security-compliance.md`
19. `/docs/implementation/IMPL-02-cicd-pipeline.md`
20. `/docs/ops/OPS-03-incident-response.md`
21. `/docs/ops/OPS-04-buyer-onboarding.md`
22. `/docs/business/BIZ-02-unit-economics.md`
23. `/docs/business/BIZ-03-gtm-strategy.md`
24. `/docs/design/DES-02-accessibility.md`
25. `/docs/design/DES-03-motion-guidelines.md`

### Files to Modify (18 existing)
All existing documents need frontmatter standardization and cross-reference sections added.
