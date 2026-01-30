---
document_id: IMPL-02
version: 1.1
status: Final
priority: P1
last_updated: 2026-01-30
owner: Engineering Lead
dependencies:
  - IMPL-01
  - TECH-07
related_documents:
  - TECH-08
  - TECH-09
  - DATA-01
---

# IMPL-02: CI/CD Pipeline

## 1. Executive Summary

This document defines the CI/CD pipeline for Frictionless using GitHub Actions. It enforces quality gates, runs tests, and deploys to staging and production with repeatable steps.

The pipeline is designed to support rapid iteration while protecting production stability.

## 2. Pipeline Stages

1. **Lint & Type Check**
2. **Docs Validation** (links + refs)
3. **Unit Tests**
4. **Build**
5. **Deploy to Staging**
6. **Smoke Tests**
7. **Deploy to Prod** (manual approval)

## 3. Example Workflow

```yaml
name: ci
on:
  push:
    branches: [main]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build

  docs-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g markdown-link-check
      - run: markdown-link-check docs/**/*.md
      - run: node scripts/validate-doc-refs.js

  deploy-staging:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm sst deploy --stage staging
```

## 4. Quality Gates

- Lint must pass
- Documentation link checks must pass
- Cross-reference validation must pass
- All tests must pass
- Coverage threshold: 80% core packages
- Smoke tests in staging required for production deploy

## 5. Documentation & Schema Sync

- Generate `schema.json` from TECH-01 + DATA-01 in CI
- Validate API schemas against `schema.json`
- Fail build if schema drift is detected

## 6. Release Process

1. Merge to main
2. CI deploys to staging
3. Product/ops run smoke tests
4. Release manager approves prod deploy

## 7. Related Documents

**Dependencies**
- IMPL-01: Section 2
- TECH-07: Section 3

**Related Specs**
- TECH-08: Section 4
- TECH-09: Section 3
- DATA-01: Section 2

**Implementation Guides**
- OPS-03: Section 3

## 8. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.1 | 2026-01-30 | Engineering Lead | Added docs validation and schema sync gates |
| 1.0 | 2026-01-30 | Engineering Lead | Initial CI/CD pipeline |
