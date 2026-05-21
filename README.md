# netsuite-skill

Claude Code skill library สำหรับ NetSuite development ของ Teibto

Skills, templates, และ UI component reference ที่ใช้ร่วมกันในทีม — ให้ Claude Code เข้าใจ pattern, convention, และ tooling ของ Teibto โดยไม่ต้องอธิบายซ้ำทุกครั้ง

---

## โครงสร้าง

```
netsuite-skill/
├── CLAUDE.md                        ← Claude Code reads this first
├── Suiteql/
│   ├── netsuite-suiteql.md          ← SuiteQL syntax reference
│   └── ns-suiteql.md                ← SuiteQL query generator
├── UI/
│   └── teibto-ui-component.md       ← Teibto Design System (TBT-DS)
└── prd/
    └── PRD_TEMPLATE.md              ← Project PRD template
```

---

## Skills

### `Suiteql/netsuite-suiteql.md` — SuiteQL Syntax Reference

**ใช้เมื่อ:** เขียน, debug, หรือ optimize SuiteQL query ใดก็ตาม

Skill นี้เป็น complete reference ของ SuiteQL syntax ที่ NetSuite รองรับ ครอบคลุม:

- **Core rules** — Oracle SQL vs SQL-92, ห้ามผสมใน query เดียว, แนะนำ Oracle SQL
- **Syntax ที่ต้องรู้** — `||` แทน `+`, `TO_DATE()` แทน date literal, `SUBSTR` แทน `SUBSTRING`
- **Join types** — Cross, Inner, Left/Right/Full Outer (ANSI only สำหรับ Right)
- **Functions ที่รองรับ** — String, Numeric, Date, NULL handling, Aggregate, Window/Analytic, CONNECT BY
- **Functions ที่ไม่รองรับ** — `CEILING`, `DATEDIFF`, `LEFT`, `RIGHT`, `SUBSTRING` พร้อม alternative
- **BUILTIN.\* catalog** — `BUILTIN.DF`, `BUILTIN.CF`, `BUILTIN.CONSOLIDATE`, `BUILTIN.HIERARCHY`, `BUILTIN.PERIOD`
- **Pagination patterns** — `runSuiteQLPaged`, ROWNUM double-subquery, ID-range cursor, `lastmodifieddate` window (ห้ามใช้ `OFFSET`)
- **Governance limits** — unit budgets ต่อ script type, row caps ต่อ API
- **Performance best practices** — indexed filters, batch query, ห้าม `SELECT *`, ห้าม calculated fields
- **Anti-patterns** — ตาราง ❌/✅ ที่เจอบ่อย
- **Volume risk tables** — `transaction`, `inventoryassignment`, `systemnote` และวิธี filter ที่ถูกต้อง

---

### `Suiteql/ns-suiteql.md` — SuiteQL Query Generator

**ใช้เมื่อ:** ต้องการสร้าง SuiteQL query ใหม่สำหรับ NetSuite 2026.1

Skill นี้ generate SuiteQL query โดยอัตโนมัติ พร้อม guard ที่ป้องกันการใช้ field ที่ tenant ไม่รองรับ:

1. **โหลด deny-list** — อ่าน field ที่ถูก gate โดย feature flag (License Plate, WMS, OneWorld) จาก memory
2. **Resolve columns จาก schema** — ตรวจ NetSuite 2026.1 schema export (~164 MB) ว่า field มีจริงไหม ก่อน emit
3. **Apply guard** — ตัด field ที่อยู่ใน deny-list ออกจาก SELECT/JOIN/WHERE อัตโนมัติ รายงาน `OMITTED:` กลับมา
4. **Apply conventions** — FK columns ได้ `BUILTIN.DF()` อัตโนมัติ, date ใช้ `TRUNC(SYSDATE)`, quantity filter `<> 0`
5. **Format + เขียน file** — บันทึก `.sql` ที่ `Schema/` พร้อม header comment และ section comments

**Output:** ไฟล์ `.sql` พร้อมใช้ + summary ของ tables, omitted columns, และ caveats

> ใช้ร่วมกันเสมอ: `ns-suiteql.md` generate → `netsuite-suiteql.md` ตรวจ syntax

---

### `UI/teibto-ui-component.md` — Teibto Design System (TBT-DS)

**ใช้เมื่อ:** สร้างหรือแก้ไข Suitelet ที่ใช้ custom HTML (ไม่ใช่ serverWidget มาตรฐาน)

Skill นี้เป็น reference ของ **Teibto Design System** — Lit Web Components library สำหรับ NetSuite Suitelet UI ของ Teibto:

**Brand**
- Primary: Navy Blue `#0D1171` (จาก logo TEIBT)
- Accent: Gradient `#8B35C8` → `#59BBF6` (จาก logo O)
- Font: Inter + IBM Plex Sans Thai

**Components (tbt-\* prefix)**

| Component | Tag | หน้าที่ |
|---|---|---|
| App Shell | `<tbt-app-shell>` | Page wrapper หลัก |
| Menubar | `<tbt-menubar>` | Top navigation |
| Sidebar | `<tbt-sidebar>` | Side navigation แบบ collapsible |
| Subtab | `<tbt-subtab>` | Tab ภายในหน้า |
| Button | `<tbt-button>` | primary / secondary / danger / ghost / accent |
| Badge | `<tbt-badge>` | Status indicator |
| Field | `<tbt-field>` | Label + value pair |
| Field Grid | `<tbt-field-grid>` | Responsive grid ของ fields |
| Form | `<tbt-form>` | Form wrapper |
| Search | `<tbt-search>` | Search input + debounce |
| Dropdown | `<tbt-dropdown>` | Select list |
| Multi-select | `<tbt-multiselect>` | Multiple selection |
| Date Picker | `<tbt-datepicker>` | Calendar popup |
| Table | `<tbt-table>` | Data table + sort + pagination |
| Section | `<tbt-section>` | Content block |
| Summary | `<tbt-summary>` | KPI summary row |

**Architecture**
- Lit 3 Web Components (CDN หรือ bundle ใน File Cabinet)
- ไม่ต้อง build step ในช่วง dev
- Deploy เป็น `tbt-ds.min.js` + `tbt-theme.css` ที่ `/SuiteScripts/Teibto/ds/v{X.Y.Z}/`
- Design tokens ทั้งหมดอยู่ใน `tbt-theme.css` — ห้ามใส่ hex color ใน component โดยตรง

**Governance rules**
- ห้ามใช้ raw HTML primitive (`<button>`, `<select>`) เมื่อมี `tbt-*` component
- ห้ามใส่ `<style>` block ใน consumer page
- Sentence case ทุก label
- ระบุ version ชัดเจนใน path เสมอ

---

### `prd/PRD_TEMPLATE.md` — PRD Template

**ใช้เมื่อ:** เริ่มโปรเจกต์หรือ feature ใหม่ใน NetSuite

Template มาตรฐานสำหรับเขียน Product Requirements Document ของ Teibto:

- Fork → rename เป็น `PRD-{PROJECT}-{MODULE}-{NNN}.md`
- ครอบคลุม 16 sections: Executive Summary → Business Context → Scope → User Stories → Functional Requirements → Data Model → Technical Design → UI/UX → Integration → Performance → Security → Subsidiary → Deployment → Testing → Risks → Open Questions
- **Naming convention** — Custom Record/List/Transaction ต้องมี 3-letter topic prefix (`customrecord_lot_*`)
- **Data Model** — Native records, Custom fields, Custom records, Custom lists, Custom transactions
- **UI/UX** — เมื่อเลือก Custom HTML Suitelet → บังคับใช้ `UI/teibto-ui-component.md`
- **Dependencies** — เมื่อมี Query ข้อมูล → บังคับใช้ `Suiteql/netsuite-suiteql.md` + `Suiteql/ns-suiteql.md`

---

## GitHub Repository Checklist

ทุก project ที่ push ขึ้น GitHub **ต้องมี `README.md`** ที่หน้าแรก repo อ่านแล้วเข้าใจได้ทันที

| ส่วน | เนื้อหา |
|---|---|
| ชื่อ + คำอธิบาย | project ทำอะไร แก้ปัญหาอะไร |
| ลิงก์ PRD | ตาราง PRD ID / version / status พร้อม link |
| ขอบเขต Phase ปัจจุบัน | bullet list In Scope |
| Tech Stack | ตาราง Layer / Technology |
| Skills ที่ใช้ | ลิงก์ไป `kingcomen/netsuite-skill` + อธิบาย skill ไหนใช้ทำอะไร |

> ห้าม push ขึ้น GitHub โดยไม่มี README.md

---

## การใช้งานร่วมกัน

```
PRD ใหม่
  ├── มี Query?     → ใช้ ns-suiteql.md (generate) + netsuite-suiteql.md (verify)
  └── มี Custom UI? → ใช้ teibto-ui-component.md (components + patterns)

Push ขึ้น GitHub
  └── ต้องมี README.md ครบ 5 ส่วน (ดู GitHub Repository Checklist)
```

---

## Author

Wichit Wongta — Teibto
