# PRD: [Project / Feature Name]

> **Fork this template** — copy ทั้งไฟล์ → rename เป็น `PRD-{ID}-{slug}.md` → เริ่มกรอก

| Field | Value |
|---|---|
| PRD ID | `{PROJECT}-{MODULE}-{NNN}` (e.g. TEIBTO-IF-001) |
| Version | 0.1 (Draft) |
| Date | YYYY-MM-DD |
| Author | Wichit Wongta |
| Reviewers | [business owner], [tech lead], [end-user rep] |
| Status | Draft \| In Review \| Approved \| Implemented \| Deprecated |
| Related Docs | [link 1], [link 2] |

---

## 1. Executive Summary
- **Problem:** [1-2 sentences ของปัญหา]
- **Solution:** [1 sentence ของวิธีแก้]
- **Impact:** [ใครได้, ขนาดเท่าไหร่]
- **Effort:** S \| M \| L
- **Target release:** YYYY-MM-DD or Sprint N

## 2. Business Context

### 2.1 Current State
[อธิบาย flow ปัจจุบัน — ใส่ diagram ถ้าซับซ้อน]

### 2.2 Pain Points
- [ปัญหา 1] — quantify: เสีย X ชม./วัน × Y คน = Z ชม./สัปดาห์
- [ปัญหา 2]
- [ปัญหา 3]

### 2.3 Stakeholders
| Role | Name | Responsibility |
|---|---|---|
| Business Owner | | อนุมัติ scope, สถานะ go/no-go |
| End Users | | ใช้งาน feature นี้ (จำนวน N คน) |
| System Admin | | maintenance, support |
| Tech Lead | | review tech design |
| Developer | | implement |

### 2.4 Success Metrics
| Metric | Baseline | Target | Measurement |
|---|---|---|---|
| | | | |

## 3. Scope

### 3.1 In Scope
- [item 1]
- [item 2]

### 3.2 Out of Scope
- [item 1] — เหตุผล: ...
- [item 2] — เหตุผล: ...

### 3.3 Phasing
- **Phase 1 (MVP):** ...
- **Phase 2:** ...
- **Future:** ...

## 4. User Stories

### Story [ID-001]
**As a** [role]
**I want** [action]
**So that** [benefit]

**Acceptance Criteria:**
- **AC1:** Given [precondition], When [action], Then [expected outcome]
- **AC2:** Given [precondition], When [action], Then [expected outcome]
- **AC3:** Given [precondition], When [action], Then [expected outcome]

### Story [ID-002]
...

## 5. Functional Requirements

### 5.1 Business Rules
| ID | Rule | Source |
|---|---|---|
| FR-01 | [declarative rule] | [stakeholder/doc] |
| FR-02 | | |

### 5.2 Validation Rules
| ID | Rule | Severity | Error Message |
|---|---|---|---|
| VR-01 | | Error \| Warning | |

### 5.3 Calculation Rules
```
[formula 1]
[formula 2]
```

## 6. Data Model

> **🔴 Naming Convention (CRITICAL):** ทุก Custom Record / Custom List / Custom Transaction ต้องมี **3-letter topic prefix** เช่น `customrecord_lot_*`, `customlist_apr_*`, `customtransaction_inv_*` (XXX = ตัวย่อ 3 ตัวของ topic ไม่ใช่ project)
> ระบุ prefix ที่จะใช้ในเอกสารนี้: **`___`** (เช่น `lot` สำหรับ lot tracking, `apr` สำหรับ approval, `bom` สำหรับ BOM)

### 6.1 Native Records Affected
| Record | Operations | Notes |
|---|---|---|
| `itemfulfillment` | Read, Update sublist | only when status = Picked |
| | | |

### 6.2 Custom Fields
| Field ID | Label | Type | Record | Required | Default | Notes |
|---|---|---|---|---|---|---|
| `custbody_xxx` | | Free-Form Text | IF | N | | |

### 6.3 Custom Records *(3-letter topic prefix required)*
**Custom Record:** `customrecord_XXX_<name>` (เช่น `customrecord_lot_scan_log`)
| Field | Type | Required | Default | Notes |
|---|---|---|---|---|
| `name` | Auto-Number | | `XX-{####}` | |
| `custrecord_XXX_<field>` | List/Record | Y | | → Reference to `xxx` |

**Indexes:** [list]
**Permissions:**
- Role X: [perms]
- Role Y: [perms]

### 6.4 Custom Lists *(3-letter topic prefix required)*
| Script ID | Label | Values | Inactive Allowed |
|---|---|---|---|
| `customlist_XXX_<name>` | | val1, val2, val3 | Y/N |

### 6.5 Custom Transactions *(ถ้ามี — 3-letter topic prefix required)*
**Custom Transaction:** `customtransaction_XXX_<name>` (เช่น `customtransaction_lot_adjustment`)
- Numbering format: `XXXADJ-{####}`
- GL Impact: Y/N
- Subsidiaries: [scope]
- Permissions: [matrix]

### 6.6 Relationship Diagram
```
[ASCII or Mermaid diagram]
```

## 7. Technical Design (High-Level)

### 7.1 Script Architecture
| Script | Type | API Version | Purpose | Why this type |
|---|---|---|---|---|
| `cs_xxx` | Client Script | 2.1 | realtime validation | needs immediate feedback |
| `ue_xxx` | User Event afterSubmit | 2.1 | log create | trigger on save |
| `mr_xxx` | Map/Reduce | 2.1 | reconcile | high volume batch |

### 7.2 File Structure
```
/SuiteScripts/{project}/
  lib_xxx.js
  cs_xxx.js
  ue_xxx.js
  mr_xxx.js
  /html/
    xxx.html
```

### 7.3 Dependencies
- Shared libraries: ...
- Other custom scripts: ...
- External services: ...
- Bundles: ... (check naming conflicts)

> **🗄️ ถ้ามีการ Query ข้อมูลจาก NetSuite** — ต้องใช้ SuiteQL skills เสมอ
> - `Suiteql/netsuite-suiteql.md` — syntax reference, Oracle SQL rules, BUILTIN.*, performance
> - `Suiteql/ns-suiteql.md` — generate SuiteQL query พร้อม deny-list guard (schema 2026.1)
> ห้ามเขียน SuiteQL โดยไม่ผ่าน skill ทั้งสองนี้

## 8. UI / UX

### 8.1 Mockup / Wireframe
[link to Figma, image, หรือ ASCII wireframe]

### 8.2 Native vs INLINEHTML Decision
**Choice:** [native serverWidget | INLINEHTML | Custom HTML Suitelet]
**Reason:** ...

> **🎨 ถ้าเลือก Custom HTML Suitelet** — ต้องใช้ Teibto Design System (TBT-DS) เท่านั้น
> อ่าน `UI/teibto-ui-component.md` ก่อนเขียน UI ทุกครั้ง
> ห้ามใช้ raw HTML primitives เมื่อมี `tbt-*` component แล้ว

### 8.3 User Flow
```
[Start] → [Action 1] → [Action 2] → [End]
              ↓
          [Error path]
```

### 8.4 Error Handling UX
| Scenario | UX |
|---|---|
| Success | Green toast 2s |
| Validation error | Red banner + sound |
| System error | Modal with error code + "Contact admin" |

## 9. Integration

### 9.1 External Systems
| System | Direction | Protocol | Auth | Frequency | Volume |
|---|---|---|---|---|---|
| | push/pull | REST/SOAP/SFTP | OAuth2/token | realtime/batch | |

### 9.2 Internal Modules
- Affects: [list of NS modules]
- Triggers: [downstream scripts]

## 10. Performance & Governance

### 10.1 Volume Baseline
- Records/day: ~N
- Lines per record: avg N, max N
- Concurrent users: N
- Peak time: HH:MM-HH:MM (% of daily volume)
- Growth rate: N%/year

### 10.2 SLA Target
| Metric | Target |
|---|---|
| Response time (p50) | < N ms |
| Response time (p95) | < N ms |
| Throughput | N records/min |
| Availability | 99.x% |

### 10.3 Governance Budget
| Script | Limit | Est. Use | Margin | Optimization If Needed |
|---|---|---|---|---|
| UE | 1000 | | | |
| Suitelet | 1000 | | | |
| MR map | 1000 | | | |

## 11. Security & Permissions

### 11.1 Role Matrix
| Role | Read | Create | Edit | Delete | Run Script |
|---|---|---|---|---|---|
| | | | | | |

### 11.2 Sensitive Data
- PII fields: [list]
- Encryption: at rest / in transit
- Audit log: [requirement]

### 11.3 Available Without Login
- Y / N
- If Y: rate limiting, IP whitelist, etc.

## 12. Subsidiary & Multi-Book Considerations
- Subsidiary scope: [single | multi | all]
- Cross-subsidiary scenario: [allow | block | warning]
- Currency conversion: [if applicable]
- Multi-book: [primary book only | all books]

## 13. Migration & Deployment

### 13.1 SDF Bundle Composition
- Scripts: N files
- Custom records: N
- Custom fields: N (across X records)
- Roles: [create new / modify existing]
- Saved searches: N
- Workflows: N

### 13.2 Data Migration (ถ้ามี)
- Source: ...
- Target: ...
- Volume: N records
- Mapping: [link to mapping doc]
- Validation: [criteria]
- Cutover window: HH:MM-HH:MM

### 13.3 Deployment Plan
| Step | Date/Time | Owner | Duration | Notes |
|---|---|---|---|---|
| Deploy to Sandbox | | | | |
| Smoke test (Sandbox) | | | | |
| UAT | | | | |
| Production deploy | | | | |
| Smoke test (Prod) | | | | |
| Go/No-go | | | | |

### 13.4 Rollback Plan
- **Trigger:** [conditions ที่ต้อง rollback]
- **Actions:** 
  1. ...
  2. ...
- **Data restore:** [strategy ถ้าจำเป็น]
- **Communication:** [ใครต้องแจ้ง]

## 14. Testing Strategy

### 14.1 Test Scenarios
- [ ] Happy path: ทุก story
- [ ] Validation error: ทุก rule
- [ ] Permission denial
- [ ] Concurrency (2+ users)
- [ ] Volume (max boundary)
- [ ] Edge cases: null, empty, max length, Thai chars, special chars
- [ ] Mobile / tablet (ถ้าเกี่ยวข้อง)
- [ ] Browser compatibility (Chrome, Edge, Safari)

### 14.2 Acceptance Tests (UAT)
| Test ID | Story | Steps | Expected | Tester | Pass/Fail |
|---|---|---|---|---|---|
| AT-01 | | | | | |

### 14.3 Performance Benchmark
- Dataset: ...
- Tool: [Playwright / manual / load test]
- Compare: before vs after
- Pass criteria: ...

## 15. Risks & Assumptions

### 15.1 Risks
| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-01 | | L/M/H | L/M/H | |

### 15.2 Assumptions
| ID | Assumption | Validation | If Wrong |
|---|---|---|---|
| A-01 | | how to validate | impact + plan B |

## 16. Open Questions & Decision Log

### 16.1 Open Questions
| ID | Question | Owner | Due | Status |
|---|---|---|---|---|
| Q-01 | | | | Open/Resolved |

### 16.2 Decision Log
| Date | Decision | Reason | Decided By |
|---|---|---|---|
| | | | |

---

## Appendix

### A. Glossary
| Term | Definition |
|---|---|
| | |

### B. References
- [link 1]
- [link 2]

### C. Change Log
| Version | Date | Author | Changes |
|---|---|---|---|
| 0.1 | YYYY-MM-DD | | Initial draft |
| 1.0 | YYYY-MM-DD | | Approved |

### D. Sign-off
| Role | Name | Date | Signature |
|---|---|---|---|
| Business Owner | | | |
| Tech Lead | | | |
| PM | | | |
