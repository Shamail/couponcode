# Comprehensive Documentation Enhancement Plan
## From "Descriptive Documentation" to "Prescriptive & Resilient Documentation"

---

## Executive Summary

This plan transforms the Frictionless documentation suite from describing "how things work when they work" to governing "how the platform survives, scales, and handles failure." The enhancement addresses 5 key gaps across 5 phases, resulting in 6 new documents and 4 major document enhancements.

**Current State:** 47 well-structured documents covering strategy, product, tech, design, ops, and business—but siloed with "happy path" bias.

**Target State:** A unified knowledge graph with cross-cutting threads, failure runbooks, fraud detection, and Morocco-specific operational automation.

---

## Phase 1: Structural Metamorphosis (Unify Siloed Documents)

### Deliverable 1.1: `threads/THREAD-01-data-lifecycle.md`
**"The Life of a Ping"**

Traces GPS coordinate flow end-to-end:
```
Expo Location → Privacy Fuzzing (3 decimals) → API Gateway → Lambda
→ PostGIS Ingestion → Heatmap Aggregation → Seller App → Mapbox Render
```

| Stage | Primary Source | Key Details |
|-------|---------------|-------------|
| Expo Location | PRD-01 §F2 | 30s interval, 50m distance threshold |
| Privacy Fuzzing | TECH-02 §4 | ~111m precision, client-side truncation |
| API Validation | TECH-06 §POST /ping | Zod schema, rate limit 60/min |
| PostGIS Write | TECH-01 §user_locations | SP-GiST index, UPSERT on user_id |
| TTL Cleanup | TECH-02 §cron | 30-min expiration, 5-min cron |
| Heatmap Query | TECH-03 | ST_DWithin, k-anonymity k=3 |
| Mapbox Render | DES-01 §HeatmapLayer | Color gradient, 30s polling |

---

### Deliverable 1.2: `threads/THREAD-02-redemption-flow.md`
**"The Redemption Atomic Unit"**

Maps UI states ↔ API codes ↔ DB constraints:

| Component | Document | Critical Spec |
|-----------|----------|---------------|
| QR Display | PRD-01 §F4 | 60s JWT rotation, RS256 signed |
| Color Code | TECH-04 §2.3 | Deterministic from token hash |
| Scanner UI | DES-01 §2.1 | Square viewport, corner indicators |
| API Verify | TECH-06 §POST /redeem | Error codes: EXPIRED_TOKEN, ALREADY_REDEEMED |
| Transaction | TECH-04 §4 | BEGIN/COMMIT, UNIQUE(user_id, deal_id) |
| Success UI | DES-01 §3.1 | Checkmark + confetti + haptic |

---

### Deliverable 1.3: `data/DATA-01-schema-dictionary.md`
**Single Source of Validation Truth**

Centralizes rules scattered across PRD-01, PRD-02, TECH-01, TECH-06:

| Field | Constraint | Source | Enforced At |
|-------|-----------|--------|-------------|
| lat | -90 to 90 | TECH-06 | Zod + CHECK |
| deal.title | max 50 chars | PRD-02 | Zod + VARCHAR |
| discount_value | 1-100 if % | TECH-06 | Zod |
| qr_code | 64 chars max | TECH-01 | VARCHAR |
| (user_id, deal_id) | unique | TECH-01 | UNIQUE constraint |

**Time Constants:**
- Location TTL: 30 min
- QR Token TTL: 60 sec
- Heatmap Cache: 25 sec
- Polling Interval: 30 sec

---

### Deliverable 1.4: `tech/ADR-001-stack-decisions.md`
**Why Postgres+PostGIS Over DynamoDB**

| Requirement | DynamoDB | PostGIS | Decision |
|-------------|----------|---------|----------|
| Spatial queries | Geohash (manual) | ST_DWithin (native) | PostGIS |
| Aggregation | Client-side | SQL GROUP BY | PostGIS |
| ACID transactions | Limited | Full | PostGIS |
| Developer familiarity | Low | High (SQL) | PostGIS |

**Consequences:**
- ✅ Single query for "users within 500m"
- ✅ Heatmap aggregation ~50ms
- ✅ Reliable redemption transactions
- ⚠️ Vendor awareness (mitigated: standard SQL)

---

## Phase 2: Technical Hardening (Anti-Fragile Layer)

### Deliverable 2.1: Enhance `TECH-01` with Partitioning Strategy

**Problem:** `user_locations` table will explode with 30s pings from active users.

**Solution: Daily Partitioning**
```sql
CREATE TABLE user_locations_partitioned (
    id UUID DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    geom GEOMETRY(Point, 4326) NOT NULL,
    recorded_at TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (id, recorded_at)
) PARTITION BY RANGE (recorded_at);
```

**Partition Manager Cron (daily at midnight):**
- Create partitions 2 days ahead
- Drop partitions older than 3 days (O(1) vs DELETE O(N))

**Read/Write Splitting Thresholds:**
| Metric | Threshold | Action |
|--------|-----------|--------|
| Read RPM | >10,000 | Route heatmap to replica |
| Write RPM | >5,000 | Consider batching |
| p95 latency | >500ms | Enable replica failover |

---

### Deliverable 2.2: Enhance `TECH-04` with Offline Redemption Protocol

**Problem:** GUIDE-03 says "Retry on stable connection"—unacceptable for frictionless payments.

**Solution: Signed Offline Tokens**
```typescript
interface OfflineTokenPayload {
  offline: true;
  deal_id: string;
  deal_snapshot: { title, discount_type, discount_value };
  device_id: string;
  sequence: number;  // Fraud detection
  exp: number;       // 24-hour validity (vs 60s online)
}
```

**Fraud Risk Caps:**
| Cap | Limit | Rationale |
|-----|-------|-----------|
| Per Hour | 5 offline redemptions | Prevent abuse bursts |
| Per Day | 15 offline redemptions | Daily ceiling |
| Max Value | 500 MAD | Limit exposure |
| Sequence Gap | Alert if >3 skipped | Detect manipulation |

**Deferred Reconciliation:**
- Seller stores redemptions in offline ledger (AsyncStorage)
- On connectivity, POST /redeem/sync with batch upload
- Server handles: accepted, duplicate, conflict, rejected

---

### Deliverable 2.3: Create `TECH-11-failure-modes.md`

**Scenario A: Neon Cold Start Latency > 3s**
- **Detection:** CloudWatch alarm on Lambda p99 > 3000ms
- **Mitigation 1:** Keep-warm Lambda (`rate(4 minutes)`)
- **Mitigation 2:** Skeleton loader UI while DB warms

**Scenario B: Mapbox API Outage**
- **Detection:** HEAD request to tile endpoint
- **Mitigation 1:** Pre-cached vector tiles for Morocco cities
- **Mitigation 2:** Abstract "Radar View" (concentric circles, activity dots)

**Scenario C: PostGIS Query Timeout**
- **Detection:** Lambda timeout after 8s (10s limit)
- **Mitigation:** Failover to read replica
- **Recovery:** REINDEX CONCURRENTLY during low traffic

**Scenario D: Lambda Cold Start > 1s**
- **Detection:** CloudWatch Logs Insights on @initDuration
- **Mitigation:** Connection pre-warming in module scope
- **Critical paths:** Provisioned concurrency for /redeem

---

## Phase 3: Business & Economic Simulation

### Deliverable 3.1: `biz/BIZ-04-fraud-vectors.md`

**GPS Spoofing Detection:**
| Method | Threshold | Action |
|--------|-----------|--------|
| Altitude check | >50m from terrain | Flag |
| Speed variance | >120 km/h | Block |
| Location jumping | >5km in <5min | Suspend |
| Mock Location API | Any detection | Warn + log |

**Gamification Abuse Detection:**
| Abuse | Pattern | Mitigation |
|-------|---------|------------|
| Streak manipulation | Timezone switching | Lock after 2 changes |
| Point farming | >10 pings, no movement | Cap daily points |
| Bot detection | Identical timing | CAPTCHA challenge |
| Multi-account | Same device fingerprint | Link + merge |

**Merchant Collusion Detection:**
- Device fingerprint clustering
- Graph analysis of redemption networks
- Z-score on daily redemption counts

**Sensitivity Matrix Enhancement:**
- Add fraud_loss_rate (2-5% of GMV) to BIZ-02 model
- Add seasonal multipliers (Ramadan 1.8x, Summer 1.4x)
- Add cohort economics (Free vs Pro vs Enterprise LTV)

---

## Phase 4: Morocco Context Operationalization

### Deliverable 4.1: `ops/OPS-05-compliance-automation.md`

**Right to be Forgotten SQL Template:**
```sql
CREATE FUNCTION anonymize_user(p_user_id UUID) RETURNS JSONB AS $$
BEGIN
  DELETE FROM user_locations WHERE user_id = p_user_id;
  UPDATE redemptions SET user_id = gen_random_uuid() WHERE user_id = p_user_id;
  DELETE FROM users WHERE id = p_user_id;
  INSERT INTO deletion_audit_log (original_user_id, deleted_at, reason)
  VALUES (p_user_id, NOW(), 'CNDP Right to be Forgotten');
  RETURN jsonb_build_object('success', true);
END;
$$ LANGUAGE plpgsql;
```

**DSR Workflow Timeline:**
- Day 0: Receive, verify identity, acknowledge (48h)
- Day 1-7: Data gathering, identify all stores
- Day 8-25: Execute deletion/export
- Day 26-30: Confirm to requester, archive

**CNDP Notification Deadlines:**
- Data breach (high risk): 72 hours to CNDP + users
- New processing activity: Before processing
- Cross-border transfer: Before transfer

---

### Deliverable 4.2: Enhance `DES-01` with Lite Mode

**Activation Criteria:**
- Network RTT > 500ms
- Save-Data header present
- effectiveType === '2g' or 'slow-2g'

**Lite Mode Constraints:**
| Asset | Normal | Lite |
|-------|--------|------|
| Map tiles | Mapbox vector | Disabled (list view) |
| Deal images | 200KB | 50KB max |
| Animations | Enabled | Disabled |
| Polling | 30s | 60s |

---

### Deliverable 4.3: Enhance `TECH-05` with Android Fragmentation

**Minimum API Level:** 23 (Android 6.0) — captures 95%+ Morocco market

**Device Testing Matrix:**
| Priority | Device | Test Focus |
|----------|--------|------------|
| P0 | Samsung Galaxy A series | Full regression |
| P0 | Xiaomi Redmi series | Battery + location |
| P1 | Infinix Hot/Note | Memory constraints |
| P1 | Tecno Spark | Low RAM (<3GB) |
| P2 | iPhone SE/XR | iOS baseline |

**RAM-Based Feature Gating:**
- <2GB: 60s polling, 20 cached deals
- 2-4GB: 30s polling, 50 cached deals
- >4GB: 20s polling, 100 cached deals

---

## Phase 5: Living Document Governance

### Deliverable 5.1: Documentation Tests in CI/CD

Add to `IMPL-02`:
```yaml
# .github/workflows/docs-lint.yml
- name: Link Check
  run: markdown-link-check docs/**/*.md

- name: Cross-Reference Validation
  run: node scripts/validate-doc-refs.js
```

### Deliverable 5.2: Schema Sync Enforcement

Generate `schema.json` from TECH-01 and validate code against DATA-01 definitions.

---

## Summary: New & Modified Files

| File | Type | Priority | Effort |
|------|------|----------|--------|
| `threads/THREAD-01-data-lifecycle.md` | New | P1 | 1 day |
| `threads/THREAD-02-redemption-flow.md` | New | P0 | 1 day |
| `data/DATA-01-schema-dictionary.md` | New | P0 | 1 day |
| `tech/ADR-001-stack-decisions.md` | New | P1 | 0.5 day |
| `tech/TECH-01-neon-schema.md` | Enhance | P0 | 1 day |
| `tech/TECH-04-redemption-security.md` | Enhance | P0 | 1.5 days |
| `tech/TECH-11-failure-modes.md` | New | P1 | 2 days |
| `biz/BIZ-04-fraud-vectors.md` | New | P1 | 2 days |
| `ops/OPS-05-compliance-automation.md` | New | P1 | 1 day |
| `design/DES-01-component-spec.md` | Enhance | P2 | 0.5 day |
| `tech/TECH-05-mobile-stack.md` | Enhance | P2 | 0.5 day |

**Total Estimated Effort:** 12-14 days

---

## Implementation Sequence

```
Week 1: Foundation
├── DATA-01 (schema dictionary) — enables all other work
├── ADR-001 (stack decisions)
└── THREAD-01 (data lifecycle)

Week 2: Critical Paths
├── THREAD-02 (redemption flow)
├── TECH-01 enhancement (partitioning)
└── TECH-04 enhancement (offline protocol)

Week 3: Resilience
├── TECH-11 (failure modes)
└── BIZ-04 (fraud vectors)

Week 4: Operationalization
├── OPS-05 (CNDP automation)
├── DES-01 enhancement (Lite Mode)
└── TECH-05 enhancement (fragmentation)
```

---

## Verification Plan

After implementation, verify each deliverable:

1. **Threads:** Walk through with engineering team, confirm all stages mapped
2. **DATA-01:** Generate Zod schemas from document, compare to codebase
3. **ADR-001:** Review with tech lead, confirm rationale is complete
4. **TECH-01:** Test partitioning migration in staging
5. **TECH-04:** Simulate offline redemption flow on device in airplane mode
6. **TECH-11:** Trigger each failure mode in staging, validate runbook
7. **BIZ-04:** Run fraud detection queries against sample data
8. **OPS-05:** Execute anonymize_user() in staging, verify audit log
9. **DES-01 Lite:** Test on throttled connection (Chrome DevTools)
10. **TECH-05:** Run test suite on Infinix Hot device

---

## Success Criteria

- [ ] All new documents pass markdown-link-check
- [ ] Cross-references are bidirectional
- [ ] DOCUMENT-TRACKER.md updated with new entries
- [ ] GAP-ANALYSIS.md shows resolved gaps
- [ ] Engineering team can trace any feature through threads
- [ ] Ops team can execute DSR in <30 days using OPS-05
- [ ] Failure runbooks tested in staging environment
