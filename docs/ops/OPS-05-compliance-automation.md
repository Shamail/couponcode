---
document_id: OPS-05
version: 1.0
status: Final
priority: P1
last_updated: 2026-01-30
owner: Operations Lead
dependencies:
  - TECH-10
  - TECH-01
related_documents:
  - OPS-03
  - OPS-02
  - THREAD-04
---

# OPS-05: Compliance Automation (CNDP)

## 1. Executive Summary

This document defines automated compliance workflows for Morocco CNDP requirements, including right-to-be-forgotten, data subject requests (DSR), and breach notification timelines.

## 2. Right to be Forgotten (SQL Template)

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

## 3. DSR Workflow Timeline

- **Day 0**: Receive request, verify identity, acknowledge within 48h
- **Day 1-7**: Data discovery, identify data stores
- **Day 8-25**: Execute deletion or export
- **Day 26-30**: Confirm completion, archive record

## 4. CNDP Notification Deadlines

| Event | Deadline | Required Action |
| --- | --- | --- |
| High-risk data breach | 72 hours | Notify CNDP + affected users |
| New processing activity | Before processing | Register with CNDP |
| Cross-border transfer | Before transfer | Submit CNDP notice |

## 5. Audit Logging Requirements

| Event | Storage | Retention |
| --- | --- | --- |
| DSR request received | `dsr_requests` | 2 years |
| Deletion executed | `deletion_audit_log` | 2 years |
| Breach notification | `incident_log` | 5 years |

## 6. Related Documents

**Dependencies**
- TECH-10: Section 4
- TECH-01: Section 3

**Related Specs**
- OPS-03: Section 4
- OPS-02: Section 6
- THREAD-04: Section 3

**Implementation Guides**
- IMPL-02: Section 5

## 7. Document Control

| Version | Date | Author | Notes |
| --- | --- | --- | --- |
| 1.0 | 2026-01-30 | Operations Lead | CNDP automation playbook |
