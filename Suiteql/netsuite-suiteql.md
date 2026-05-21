---
name: netsuite-suiteql
description: SuiteQL syntax reference for NetSuite — Oracle SQL vs SQL-92 rules, join types, supported/unsupported functions, BUILTIN.* catalog, governance limits, pagination patterns, and performance best practices. Use when writing, debugging, or optimizing SuiteQL via N/query (query.runSuiteQL), SuiteAnalytics Connect (ODBC/JDBC), REST /query/v1/suiteql, or Saved Search SQL formulas. Triggers on errors like INVALID_SUITEQL or syntax issues with TO_DATE, BUILTIN., CONSOLIDATE, HIERARCHY, CTEs, joins, and || vs + concatenation.
---

# SuiteQL — Complete Reference Skill

> **Source:** Oracle NetSuite Applications Suite Help (SuiteAnalytics → Workbook → SuiteQL)  
> **Authority:** ใช้ SKILL นี้แทนการเดา syntax — ทุกอย่างใน skill นี้ตรงกับ Oracle docs

---

## 1. Core Concept

- **SuiteQL = SQL ที่ NetSuite รองรับ** based on **SQL-92** + **Oracle SQL** syntax
- ใช้ผ่าน 3 ทาง:
  1. `N/query` module ใน SuiteScript (`query.runSuiteQL`, `query.create`)
  2. **SuiteAnalytics Connect** (ODBC/JDBC)
  3. **SuiteTalk REST Web Services**
- Query บน **analytics data source** — สอดคล้องกับ SuiteAnalytics Workbook ไม่ใช่ saved-search engine แบบเก่า
- Enforce **role-based access** เหมือน Workbook → ดึงได้เฉพาะ data ที่ role มีสิทธิ์เห็น

### ⚠️ กฎเหล็ก #1: เลือก syntax อย่างใดอย่างหนึ่งใน query เดียว
- รองรับทั้ง **SQL-92** และ **Oracle SQL** แต่ **ห้ามผสมในคำสั่งเดียวกัน**
- **แนะนำใช้ Oracle SQL** เพราะ SQL-92 อาจถูก convert internally → เกิด timeout ที่แก้ไม่ได้

---

## 2. Syntax Requirements (สำคัญ — ทำผิดคือ error ทันที)

| Rule | ✅ ถูก | ❌ ผิด |
|---|---|---|
| String concat | `a \|\| b` | `a + b` |
| Date literal | `TO_DATE('2024-01-01','YYYY-MM-DD')` | `'2024-01-01'` (ใช้ตรง ๆ ไม่ได้) |
| Right outer join (Oracle non-ANSI) | ใช้ ANSI: `RIGHT JOIN ... ON` | `where a.id (+) = b.id` ที่หมายถึง right join |
| `IN` clause | ≤ 1000 args | > 1000 args |
| `WITH` clause (CTE) | ✅ ใน **N/query + REST SuiteQL** | ❌ ใน **SuiteAnalytics Connect** (Oracle docs) — ถ้าต้อง portable ทั้ง 2 ฝั่ง ใช้ inline subquery แทน |
| `OFFSET` (pagination) | ใช้ **ROWNUM double-subquery** หรือ id-range cursor | `OFFSET 1000 ROWS` → **silently ignored** (ไม่ error แต่ไม่ทำงาน) |
| Subselect column alias | `SELECT a.externalid AccountId ...` | `SELECT a.externalid "AccountId" ...` (ใส่ quote) |
| Square brackets `[ ]` | — | **ไม่รองรับเลย** |
| Mix ANSI + non-ANSI joins | เลือกอย่างเดียว | — |
| Mix `TO_DATE` กับ `TO_TIMESTAMP` ใน predicate เดียว | ใช้ type เดียว | forced conversion → ช้า |
| `SYS_CONNECT_BY_PATH(col, sep)` | — | ❌ **`SSS_SEARCH_ERROR_OCCURRED`** ใน tenant นี้ → return parent_id+child_id pairs จาก CONNECT BY แล้วต่อ path ใน client |
| `GROUP BY BUILTIN.DF(<int_fk>)` | — | ❌ Same error → ใช้ `SELECT DISTINCT` หรือ group by raw FK + lookup display แยก |
| `OFFSET n ROWS` | — | **silently ignored** → ใช้ ROWNUM double-subquery หรือ id-range cursor |
| `tl.isinventoryaffecting = 'T'` (กรอง movement) | — | ❌ ตัด ItemShip ทิ้ง (carries `'F'`) → ใช้ `t.posting='T'` อย่างเดียว — ดู memory `project_isinventoryaffecting_unreliable` |

---

## 3. Join Types

NetSuite default ใน Workbook = **left outer join** แต่ใน SuiteQL กำหนดเองได้ทุกแบบ

### Cross Join — Cartesian product
```sql
SELECT c.entityid, e.entityid
FROM customer c, employee e
-- หรือ ANSI: FROM customer c CROSS JOIN employee e
```
ทุก row ของ customer × ทุก row ของ employee

### Inner Join — เฉพาะ row ที่ match
```sql
-- Oracle (non-ANSI)
SELECT c.entityid, e.entityid
FROM customer c, employee e
WHERE c.salesrep = e.id

-- ANSI
SELECT c.entityid, e.entityid
FROM customer c INNER JOIN employee e ON c.salesrep = e.id
```

### Left Outer Join — ทุก row จาก left + match จาก right
```sql
SELECT c.entityid, e.entityid
FROM customer c LEFT OUTER JOIN employee e ON c.salesrep = e.id
```
Row ที่ไม่ match → field จาก right table = NULL

### Right Outer Join — ใช้ได้เฉพาะ ANSI เท่านั้น
```sql
-- ✅ ANSI ใช้ได้
SELECT c.entityid, e.entityid
FROM customer c RIGHT OUTER JOIN employee e ON c.salesrep = e.id

-- ❌ Oracle non-ANSI ใช้ไม่ได้ใน SuiteQL
-- SELECT ... WHERE c.salesrep (+) = e.id
```

### Full Outer Join — รวมทั้งสองฝั่ง
```sql
SELECT c.entityid, e.entityid
FROM customer c FULL OUTER JOIN employee e ON c.salesrep = e.id
```

---

## 4. Supported Functions (ที่ใช้บ่อย)

### String
- `CONCAT(a,b)` หรือ `a || b`, `LOWER`, `UPPER`, `INITCAP`
- `SUBSTR(str, pos, len)` — **ไม่ใช่ `SUBSTRING`**
- `INSTR(str, substr)` — **ไม่ใช่ `LOCATE`/`CHARINDEX`/`POSITION`**
- `LENGTH` (ไม่ใช่ `CHAR_LENGTH`)
- `LPAD`, `RPAD`, `LTRIM`, `RTRIM`, `REPLACE`, `TRANSLATE`
- `REGEXP_INSTR`, `REGEXP_REPLACE`, `REGEXP_SUBSTR`
- `ASCII`, `CHR`, `SOUNDEX`

### Numeric
- `ABS`, `CEIL` (**ไม่ใช่ `CEILING`**), `FLOOR`, `ROUND`, `TRUNC`, `SIGN`, `MOD`, `REMAINDER`
- `POWER`, `SQRT`, `EXP`, `LN`, `LOG`
- `GREATEST`, `LEAST`
- Trig: `SIN`, `COS`, `TAN`, `ASIN`, `ACOS`, `ATAN`, `ATAN2`, `SINH`, `COSH`, `TANH`

### Date / Time
- `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `LOCALTIMESTAMP`, `SYSDATE`
- `TO_DATE(str, fmt)`, `TO_TIMESTAMP`, `TO_TIMESTAMP_TZ`, `FROM_TZ`
- `TO_CHAR(date, fmt)` — สำหรับ format date เป็น string
- `ADD_MONTHS(date, n)`, `MONTHS_BETWEEN(d1, d2)`
- `LAST_DAY`, `NEXT_DAY`, `NEW_TIME`, `SYS_EXTRACT_UTC`, `TZ_OFFSET`

### Conversion
- `TO_NUMBER`, `TO_CHAR`, `TO_DATE`, `TO_CLOB`, `TO_NCLOB`
- `TO_BINARY_FLOAT`, `TO_BINARY_DOUBLE`
- `TO_MULTI_BYTE`, `TO_SINGLE_BYTE`, `TO_NCHAR`
- `CHARTOROWID`, `ASCIISTR`, `UNISTR`, `COMPOSE`, `DECOMPOSE`

### NULL handling
- `NVL(expr, replacement)` — replace NULL
- `NVL2(expr, ifNotNull, ifNull)`
- `COALESCE(a, b, c, ...)` — first non-null
- `NULLIF(a, b)` — NULL ถ้าเท่ากัน
- `NANVL` — สำหรับ binary float/double

### Aggregate / Analytic
- `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `MEDIAN`
- `APPROX_COUNT_DISTINCT` — เร็วกว่า `COUNT(DISTINCT ...)` สำหรับ data ใหญ่
- `CORR`, `CORR_K`, `CORR_S`, `COVAR_POP`, `COVAR_SAMP`
- `WIDTH_BUCKET` — สร้าง histogram bins

### Window / Analytic Functions (ทำงานเต็มรูปแบบ)
- `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)`
- `RANK()`, `DENSE_RANK()` — ranking (มีและไม่มี gap)
- `SUM/AVG/MAX/MIN/COUNT(...) OVER (PARTITION BY ... ORDER BY ...)` — running totals / moving aggregates
- `MEDIAN()` — สามารถใช้กับ window
- `LAG`/`LEAD` — **ไม่อยู่ใน Oracle official supported list** → ใช้แบบ test-then-trust

```sql
-- ตัวอย่าง: running total ของยอดขายต่อ customer
SELECT
  t.id, t.entity, t.tranDate, t.foreignTotal,
  SUM(t.foreignTotal) OVER (PARTITION BY t.entity ORDER BY t.tranDate, t.id) AS running_total,
  ROW_NUMBER()        OVER (PARTITION BY t.entity ORDER BY t.tranDate DESC)  AS rn_recent
FROM transaction t
WHERE t.type='SalesOrd'
```

### Hierarchical (CONNECT BY)
- `START WITH ... CONNECT BY [PRIOR ...]` — รองรับ
- `LEVEL`, `ORDER SIBLINGS BY` — รองรับ
- ⚠ **Performance**: NSP archives รายงาน CONNECT BY ช้าลงในบาง release โดยเฉพาะเมื่อ wrap ด้วย `BUILTIN.DF` — ถ้าเป็น chart-of-accounts/subsidiary/location ใช้ `BUILTIN.HIERARCHY` แทน

### Logic
- `DECODE(expr, search1, result1, search2, result2, ..., default)`
- `CASE WHEN ... THEN ... ELSE ... END`
- `COALESCE`, `NULLIF`, `NVL`, `NVL2`
- `BITAND`, `ORA_HASH`, `VSIZE`

---

## 5. ❌ Unsupported Functions (และตัวที่ใช้แทน)

| ❌ Unsupported | ✅ ใช้แทน |
|---|---|
| `CEILING` | `CEIL` |
| `CHAR_LENGTH`, `CHARACTER_LENGTH` | `LENGTH` |
| `CHARINDEX`, `LOCATE`, `POSITION` | `INSTR` |
| `LCASE` | `LOWER` |
| `UCASE` | `UPPER` |
| `LEFT`, `RIGHT` | `SUBSTR` |
| `SUBSTRING` | `SUBSTR` |
| `DATEDIFF` | คำนวณเอง: `(d1 - d2)` หรือ `MONTHS_BETWEEN` |
| `LISTAGG` | **Officially unsupported** แต่ **ใช้งานได้จริงใน N/query + REST SuiteQL** (Tim Dietrich + NSP archives ยืนยัน) — มี 4000-char per-result limit / อาจ break ใน release ถัดไป → ถ้าใช้ ต้อง fallback plan / สำหรับ Connect: ใช้ XMLAGG หรือ aggregate ใน app layer |
| `CONVERT`, `CHAR`, `COT`, `REPEAT`, `BIT_LENGTH`, `BIT_XOR_AGG` | ไม่มีตัวแทน |

---

## 6. BUILTIN.* Functions (เฉพาะ SuiteQL)

ต้อง prefix `BUILTIN.` เสมอ เช่น `BUILTIN.CONSOLIDATE(...)`

| Function | จุดประสงค์ |
|---|---|
| `BUILTIN.CF` | Format field value as **TEXT** context (เหมือนที่เห็นใน UI) — ใช้กับ select/list/checkbox/date fields |
| `BUILTIN.DF` | Display Format — คล้าย CF แต่สำหรับ display value ของ list/record fields |
| `BUILTIN.CONSOLIDATE` | แปลง currency amount → target currency ของ subsidiary parent |
| `BUILTIN.CURRENCY` | คืน currency ID/info ตาม subsidiary context |
| `BUILTIN.CURRENCY_CONVERT` | แปลง amount ระหว่าง currency 2 ตัว ตาม exchange rate |
| `BUILTIN.HIERARCHY` | คืน parent / level / full path ของ hierarchical record (account, location, dept) |
| `BUILTIN.MNFILTER` | Multi-currency filter (consolidated reporting) |
| `BUILTIN.NAMED_GROUP` | Group by named bucket (rich grouping logic) |
| `BUILTIN.PERIOD` | คืน period info (start/end/relative period) |
| `BUILTIN.RELATIVE_RANGES` | Date range filter เช่น "last 30 days", "this month" — มาเป็น helper ของ `WHERE` clause |

### ตัวอย่าง CF / DF
```sql
SELECT
  t.id,
  BUILTIN.DF(t.entity)    AS customer_name,   -- display name แทน internal id
  BUILTIN.CF(t.status)    AS status_text      -- "Pending Approval" แทน "A"
FROM transaction t
```

### ⚠️ HIERARCHY caveat (จาก official docs)
`BUILTIN.HIERARCHY(PARENT, 'LEVEL')` อาจคืนค่า level เป็น `n-1` แทน `n` ถ้า depth ของ parent ผูกกับ base order ตรง ๆ → **ทดสอบกับ data จริงเสมอ**

---

## 7. Limitations & Exceptions (ที่ Oracle ระบุชัด)

1. **Casing ของ record / field names อาจเปลี่ยน** หลัง release — `account` อาจกลายเป็น `Account`/`ACCOUNT` → **อย่าพึ่ง case-sensitive logic** ใน application code
2. **Aliases (`AS`) ใน mapped results** จะกลายเป็น **lowercase** เสมอ — `SELECT id AS myID` → key เป็น `myid`
3. **CSV injection mitigation** — text values ทุกตัวจะถูก wrap ด้วย `"..."`, และค่าที่ขึ้นต้นด้วย `-`, `+`, `=`, `@`, tab, EOL → จะถูกใส่ `'` นำหน้า
4. **Custom transaction body fields** — ไม่ดึง subtab data ออกมาผ่าน Analytic API
5. **Sorting CLOB fields** — ประเมินแค่ 250 ตัวแรก
6. **TransactionStatus + TranType bug** — `WHERE TranType = 'CuTrSale'` อาจ return ผลแปลก → ใช้ `LIKE 'CuTrSale%'` แทน
7. **Square brackets `[ ]`** — ห้ามใช้
8. **Listing application data จาก custom field** — ไม่รองรับใน Analytic API

---

## 8. Performance Best Practices (สำคัญที่สุดสำหรับ data ใหญ่)

### ✅ DO
- ใช้ **Oracle SQL syntax** (ไม่ใช่ SQL-92) เสมอ
- **เจาะจง column** ที่ select — `SELECT id, tranid, trandate FROM transaction` ไม่ใช่ `SELECT *`
- **Filter ด้วย indexed field**: `id`, `lastmodifieddate`, primary keys → query plan optimal
- **Incremental load** ด้วย `lastmodifieddate >= TO_DATE(...)`
- **Batch query** เมื่อคาดว่า return rows มาก:
  ```sql
  WHERE id < 10000000
  WHERE id >= 10000000 AND id < 20000000
  WHERE id >= 20000000 AND id < 30000000
  ```
- ใช้ **filter type ตรงกัน** — อย่าผสม `TO_TIMESTAMP` กับ `TO_DATE` ใน filter เดียว → forced conversion ทำให้ช้า
- ใช้ **inner join** เมื่อทำได้ — เร็วกว่า outer join

### ❌ DON'T
- หลีกเลี่ยง **calculated fields** — Oracle published list:
  - `customer.oncredithold`
  - `transaction.fxamount`
  - `transaction.daysoverdue`
  - `entity.balance`
  - ตรวจ list ของ tenant ตัวเอง: `oa_columns` (Connect) → `oa_userdata` ตำแหน่ง 6 = `'C'`
- หลีกเลี่ยง **WLONGVARCHAR fields** (rich-text/HTML cols):
  - `item.featureddescription`
  - `item.storedetaileddescription`
  - `item.metataghtml`
  - **อย่า** SELECT, **อย่า** JOIN, **อย่า** filter
- หลีกเลี่ยง **function on indexed column ใน WHERE** (e.g., `UPPER(tranid) = 'SO123'`) → defeats index
- หลีกเลี่ยง **nested SELECT ลึก ๆ** — flatten ให้มากที่สุด
- หลีกเลี่ยง **`OR` ใน predicate** (logical disjunction) — แตกเป็นหลาย query แล้ว `UNION ALL` เร็วกว่า
- หลีกเลี่ยง **join ตารางเดิมซ้ำหลายครั้ง** (Oracle's explicit warning)
- หลีกเลี่ยง **correlated subqueries** ที่ re-execute ทุก outer row — rewrite เป็น `JOIN ... GROUP BY`
- หลีกเลี่ยง **`SELECT TOP n`** — รัน eval ทุก row ก่อน → ใช้ `WHERE id <= n` หรือ `ROWNUM` แทน
- หลีกเลี่ยง **`OFFSET`** — silently ignored (ดู §10 Pagination Patterns)
- หลีกเลี่ยง **systemnote/systemnoteline join transaction** ถ้าไม่ filter `recordtypeid` + `recordid` ก่อน → table มหาศาล

### Pattern: เปรียบเทียบ TOP vs WHERE
```sql
-- ❌ ช้า: ประเมินทุก row แล้วค่อยตัด
SELECT TOP 100 id, lastmodifieddate, tranid FROM transaction

-- ✅ เร็ว: index-friendly
SELECT id, lastmodifieddate, tranid FROM transaction WHERE id <= 100
```

### Pattern: Oracle ROWNUM (สำหรับ pagination)
```sql
-- จำกัด N rows + sort
SELECT *
FROM (
  SELECT id, tranid, trandate
  FROM transaction
  WHERE trandate >= TO_DATE('2026-01-01','YYYY-MM-DD')
  ORDER BY trandate DESC
)
WHERE ROWNUM <= 1000
```

---

## 9. Common Recipes

### Date filter (Oracle syntax)
```sql
SELECT id, tranid, trandate
FROM transaction
WHERE trandate >= TO_DATE('2026-04-01','YYYY-MM-DD')
  AND trandate <  TO_DATE('2026-05-01','YYYY-MM-DD')
```

### Date display in YYYY-MM-DD (สำหรับ output ไป client)
```sql
SELECT
  id,
  TO_CHAR(trandate, 'YYYY-MM-DD')        AS trandate_iso,
  TO_CHAR(createddate, 'YYYY-MM-DD HH24:MI:SS') AS created_iso
FROM transaction
```

### Aggregation + filter (HAVING)
```sql
SELECT
  email,
  COUNT(*)              AS cnt,
  MAX(createddate)      AS last_seen
FROM transaction
GROUP BY email
HAVING COUNT(*) > 2
```

### Subquery ใน WHERE
```sql
SELECT *
FROM transaction t
WHERE EXISTS (
  SELECT 1
  FROM transactionLine tl
  WHERE tl.transaction = t.id
    AND tl.foreignamount > 1000
)
```

### Set operations
```sql
SELECT id FROM transaction WHERE type='SalesOrd'
UNION
SELECT id FROM transaction WHERE type='Invoice'
```

### CASE expression
```sql
SELECT
  id, tranid,
  CASE
    WHEN status = 'A' THEN 'Approved'
    WHEN status = 'P' THEN 'Pending'
    ELSE 'Other'
  END AS status_label
FROM transaction
```

### Multi-criteria with COALESCE (avoid OR when possible)
```sql
-- Instead of: WHERE a=1 OR b=1 OR c=1
SELECT * FROM t WHERE COALESCE(a, b, c) = 1
```

---

## 10. SuiteQL ใน N/query (SuiteScript)

### `query.runSuiteQL(options)` — quick path
```javascript
// SuiteScript 2.x
const sql = `
  SELECT id, tranid, TO_CHAR(trandate,'YYYY-MM-DD') AS trandate
  FROM transaction
  WHERE type = ?
    AND trandate >= TO_DATE(?, 'YYYY-MM-DD')
`;
const rs = query.runSuiteQL({
    query : sql,
    params: ['SalesOrd', '2026-01-01']
}).asMappedResults();
// rs[0].trandate, rs[0].tranid (lowercase keys เสมอ)
```

### `query.runSuiteQLPaged` — สำหรับ result เกิน 5,000 rows
```javascript
const paged = query.runSuiteQLPaged({
  query: sql, params: [...], pageSize: 1000
});
paged.iterator().each(page => {
  page.value.data.iterator().each(row => { /* ... */ ; return true; });
  return true;
});
```

### Casing reminder (จาก Oracle docs)
- `mapped results` keys → **lowercase always** (`SELECT id AS myID` → `row.myid`)
- `query.create({type: query.Type.TRANSACTION})` → enum-based (case insensitive ปลอดภัย)
- ห้ามเขียน app code ที่พึ่ง casing ของ field/record names

---

## 11. Records Catalog & Field Discovery

วิธีค้นหา record type / field name ที่ใช้ใน SuiteQL:
- **Records Catalog UI**: Setup → Records Catalog (มี analytics view)
- **`oa_columns` system table** (Connect): list ทุก column + datatype + flags
- **`oa_tables`**: list ทุก record type ที่ query ได้

```sql
-- หา column ของ record type
SELECT * FROM oa_columns WHERE table_name = 'TRANSACTION'
```

**Tip:** Calculated field detection → check `oa_userdata` column ตำแหน่งที่ 6 = `'C'`

---

## 12. Quick Decision Tree

```
ต้องเขียน query ดึง NetSuite data?
│
├─ Static schema, ใหญ่, ใช้ filter เยอะ → SuiteQL (Oracle syntax)
├─ Need analytic data (joins workbook-style) → SuiteQL
├─ Search-style record API ที่มี subrecord เยอะ → N/search (อาจเร็วกว่าในบางกรณี)
└─ One-shot lookup field เดียว → search.lookupFields (ไม่ต้อง SuiteQL)

เขียน SuiteQL แล้ว...
│
├─ Performance ห่วย → ดู section 8 → batch / indexed filter / no SELECT *
├─ Syntax error → ดู section 2 (|| not +, TO_DATE not literal)
├─ Function not found → ดู section 5 (substitute table)
├─ ต้อง currency conversion → BUILTIN.CONSOLIDATE / CURRENCY_CONVERT
├─ ต้อง period/range filter → BUILTIN.RELATIVE_RANGES / PERIOD
└─ ต้อง display value แทน internal id → BUILTIN.DF / BUILTIN.CF
```

---

## 13. Anti-patterns ที่เจอบ่อย

| ❌ ผิด | ✅ ถูก |
|---|---|
| `WHERE trandate = '2026-04-01'` | `WHERE trandate = TO_DATE('2026-04-01','YYYY-MM-DD')` |
| `SELECT 'A=' + tranid FROM ...` | `SELECT 'A=' \|\| tranid FROM ...` |
| `WHERE id IN (1,2,3,...,5000)` | แบ่ง batch ≤ 1000 ต่อ IN |
| `WITH cte AS (...) SELECT ...` (ใน Connect) | flatten เป็น subquery — N/query + REST ใช้ได้ |
| `SELECT a.foo "Foo" FROM ...` (ใน subselect) | `SELECT a.foo Foo FROM ...` |
| `SELECT a1.id FROM acc a1, acc a2 WHERE a1.id (+) = a2.id` | ใช้ ANSI: `RIGHT JOIN ... ON` |
| `SELECT * FROM transaction` | `SELECT id, tranid, trandate FROM transaction` |
| `SELECT TOP 100 ...` (full scan) | `WHERE id <= 100` หรือ `ROWNUM <= 100` |
| `WHERE TranType = 'CuTrSale'` (TransactionStatus) | `WHERE TranType LIKE 'CuTrSale%'` |
| ผสม `TO_DATE` กับ `TO_TIMESTAMP` ใน filter เดียว | ใช้ type เดียวตลอด |

---

## 14. Volume / Governance Ceilings

| Limit | Value | Notes |
|---|---|---|
| `query.runSuiteQL` non-paged max rows | **5,000** | เกินนั้นต้องใช้ paged หรือ id-range |
| `query.runSuiteQLPaged` page size | min 5, default 50, **max 1,000** | ใช้ 1000 เสมอเพื่อลด round-trip |
| REST SuiteQL `limit` per page | max **1,000** (default 10) | header `Prefer: transient` |
| REST SuiteQL total rows (Connect ปิด) | **100,000 hard cap** | hit cap → switch to id-range cursor |
| REST SuiteQL total rows (Connect เปิด) | **unlimited** (paged) | check feature state ก่อน architecture |
| `IN` clause max args | **1,000** | เกินนั้น split เป็นหลาย query |
| Governance per `runSuiteQL` / `runSuiteQLPaged` | **10 units** | per call (ไม่ใช่ per row) |
| Suitelet/UE/Client budget | 1,000 units | ~95 SuiteQL calls |
| Scheduled Script | 10,000 units | ~990 calls |
| RESTlet | 5,000 units | ~490 calls |
| Map/Reduce — getInputData | 10,000 units | ~990 calls |
| Map/Reduce — map | 1,000 units / invocation | ~95 per record |
| Map/Reduce — reduce | 5,000 units / key | ~490 per key |
| Map/Reduce — summarize | 10,000 units | ~990 calls |
| Map/Reduce key length | 3,000 chars | |
| Map/Reduce value size | 10 MB | |
| Map/Reduce persisted total | 50 MB | |
| RESTlet request/response | 10 MB string | |
| Concurrency (REST/RESTlet/SOAP combined) | account-level (~25 typical) | reserve via Integration Record → HTTP 400 `SSS_REQUEST_LIMIT_EXCEEDED` |

**DO**: log `runtime.getCurrentScript().getRemainingUsage()` รอบๆ SuiteQL call ใน long-running scripts

---

## 15. Pagination Patterns

เลือก pattern ตาม use case (Tim Dietrich + Oracle official guidance)

### Pattern A — `runSuiteQLPaged` iterator *(ง่ายสุด, ≤100k rows)*
```javascript
const rs = query.runSuiteQLPaged({
  query: 'SELECT id, tranid FROM transaction WHERE type=? ORDER BY id',
  params: ['SalesOrd'],
  pageSize: 1000   // ใช้ max เสมอ ลด round-trip
});
rs.iterator().each(page => {
  page.value.data.iterator().each(row => { /* process */; return true; });
  return true;
});
```
- ✅ pageSize **1000** เสมอ
- ❌ อย่าใช้ pageSize 50 (default) ใน bulk

### Pattern B — ROWNUM double-subquery *(สำหรับ Suitelet UI pagination)*
```sql
SELECT * FROM (
  SELECT ROWNUM AS rn, sub.* FROM (
    SELECT id, tranid FROM transaction
    WHERE type='SalesOrd' ORDER BY id
  ) sub
) WHERE rn BETWEEN :start AND :end
```
ใช้แทน `OFFSET` (silently ignored)

### Pattern C — ID-range cursor *(แนะนำสำหรับ >100k rows)*
```sql
SELECT id, lastmodifieddate, tranid FROM transaction
WHERE id >= 10000000 AND id < 20000000
```
- หาแบ่ง keyspace จาก `MIN(id)` / `MAX(id)`
- รวมกับ `lastmodifieddate >= TO_DATE(...)` สำหรับ incremental
- **อย่า** ใช้ REST `offset` เกิน 100,000 เด็ดขาด (hard cap เมื่อ Connect ปิด)

### Pattern D — `lastmodifieddate` window cursor *(incremental sync)*
```sql
SELECT id, lastmodifieddate, tranid FROM transaction
WHERE lastmodifieddate >= TO_DATE('2026-05-01 00:00:00','YYYY-MM-DD HH24:MI:SS')
  AND lastmodifieddate <  TO_DATE('2026-05-02 00:00:00','YYYY-MM-DD HH24:MI:SS')
ORDER BY lastmodifieddate, id
```
Persist high-watermark ใน custom record ระหว่าง run  
ระวัง sub-second clustering: ใช้ composite cursor `WHERE lastmodifieddate > ? OR (lastmodifieddate = ? AND id > ?)`

---

## 16. Connect vs N/query vs REST SuiteQL

| Dimension | REST SuiteQL | N/query (SuiteScript) | SuiteAnalytics Connect |
|---|---|---|---|
| Setup cost | Low (TBA + REST enabled) | ใน script | License + JDBC/ODBC drivers |
| Row cap | 100k (Connect off) / unlimited (Connect on) | 5k non-paged / unlimited paged | unlimited |
| Page max | 1,000 | 1,000 | n/a |
| `WITH` / CTE | ✅ | ✅ | ❌ |
| `LISTAGG` | ✅ (works) | ✅ (works) | ❌ |
| `(+)` right join | ❌ ใช้ ANSI | ❌ ใช้ ANSI | ❌ ใช้ ANSI |
| Real-time | ✅ | ✅ | designed for static/batch |
| Best for | Microservices, on-demand | Server-side scripts, MR | Data warehouse, BI tools |
| Auth | TBA (HMAC-SHA256) / OAuth 2.0 | inline | Driver auth |
| Compression | gzip ถ้า client ส่ง `Accept-Encoding: gzip` | n/a | n/a |

**Critical 2026.1 changes**:
- `NetSuite.com` data source **ลบ** — ทุก Connect ต้อง migrate มา `NetSuite2.com`
- Drivers ต้อง TLS 1.3 (Windows ≥8.10.158.0)
- SuiteQL Metadata Tool added — schema introspection ดีขึ้น

**Connect = read-only** เสมอ; write ต้องผ่าน REST/SuiteScript

---

## 17. Tables with Volume Risk (paper-mill / TEIBTO context)

| Table | Risk | ข้อควรระวัง |
|---|---|---|
| `transaction` + `transactionline` | สูงมาก | filter `type` + date/id range เสมอ; `mainline='T'` ดูแค่ header, `mainline='F' AND taxline='F'` ดูแค่ line |
| `inventoryassignment` | สูง (เมื่อใช้ WMS / lot-serial) | composite PK `(transaction, transactionline)`; ใช้ reconstruct inventory journal |
| **`inventorybalance`** ⭐ | low | **ใช้แทน reconstruction** สำหรับ current snapshot — เร็วกว่า inventoryassignment มาก |
| `inventoryNumber` + `binInventoryBalance` | ปานกลาง-สูง | bin/lot/serial detail; filter `item` ก่อนเสมอ |
| `itembinquantity` | ปานกลาง | preferred bin per item-bin pair |
| `systemnote`, `systemnoteline` | มหาศาล | filter `recordtypeid` + `recordid` เสมอ; ห้าม unfiltered |
| `accountingbookgl` / `transactionaccountingline` | สูง | multi-book GL; pair กับ `BUILTIN.CONSOLIDATE` |
| `bomrevision`, `bomrevisioncomponent`, `bomassemblyitemmap` | ปานกลาง | TS1-MRP — filter `effectivestartdate <= :period_end` + `(effectiveenddate >= :period_start OR IS NULL)` |
| `assemblyitemmember` | ปานกลาง | single-level เท่านั้น; multi-level explosion ต้อง recurse ใน JS (`WITH RECURSIVE` **ไม่รองรับ**) |
| `customrecord_*` | แปรผัน | depends on Store Value + index settings |
| `file` (file cabinet) | สูง ถ้าไม่ filter | metadata only |

### Indexed columns ที่ optimal (filter ตัวนี้ก่อน)
- `id` (primary key)
- `lastmodifieddate`
- `transaction.tranid`, `transaction.type`
- `transactionline.transaction`, `transactionline.item`
- `entity` (on transaction), `subsidiary`, `account`, `posting`
- `recordtype`, `recordid` (system note)

### Snapshot fast-path (PWOC reconciliation)
```sql
-- ✅ ใช้ inventorybalance สำหรับ current snapshot (เร็วมาก)
SELECT item, location, quantityonhand, quantityavailable,
       quantityonorder, quantitycommitted
FROM inventorybalance
WHERE item = :itemid AND location = :locid
```
สำหรับ **as-of historical** ยังต้องใช้ journal pattern: `inventoryassignment + transactionline + transaction`

---

## 18. References

### Oracle official
- SuiteQL Overview: `section_156257770590.html`
- Using SuiteQL: `section_156257799794.html`
- Join Types: `section_160045092035.html`
- Syntax & Examples: `section_156257790831.html`
- Limitations: `section_156257796125.html`
- Supported / Unsupported Functions: `section_158513731864.html`
- Built-in Functions: `article_161950565221.html`
- **Performance & Best Practices: `article_0824094533.html`** ← canonical reference
- Using SuiteQL with the Connect Service: `section_156257805177.html`
- Executing SuiteQL Through REST: `section_157909186990.html`
- SuiteQL in N/query Module: `section_157960623712.html`
- Examples in N/query: `section_158039627694.html`
- Map/Reduce Best Practices + Governance

### Community / Blog (production-tested patterns)
- **Tim Dietrich** (`timdietrich.me/blog`) — pagination, BUILTIN.DF, MR+SuiteQL, transaction querying, system notes auditing, **SuiteQL Query Tool v2026.1** (de-facto prototyping environment, AI-assisted)
- **NetSuite Diagnostics** (`netsuitediagnostics.com/posts`) — BOM series (single-level, recursive, WO variance), inventoryDetail/assignment posts → directly applicable to TS1-MRP / PWOC
- **Tanwa Sripan** (`tanwasripan.com/blog`) — CONNECT BY hierarchy, CTE patterns
- **Marty Zigman / Prolecto** (`blog.prolecto.com`) — system notes decoding, bin complexity, allocation automation
- **Eric T Grubaugh / Stoic Software** (`gitlab.com/stoicsoftware`) — N/query patterns, "Basic Querying in SuiteScript"
- **NetSuite Professionals Slack archive** (`archive.netsuiteprofessionals.com`) — `#suiteql`, `#suitescript`, `#integrations`
- **GitHub `danbolick/SuiteQL-Knowledge-Base`** — well-curated quick reference
- **GitHub `GSpo7/suiteql-sql-library`** — example query patterns
- **Scott Danesi** (`scottdanesi.com`) — costed BOM revision query (avg-cost assemblies, paper-pulp costing)
- **Houseblend.io** — pragmatic SuiteQL+ERP joining + MR articles

### Companion artifact
- `Schema/compass_artifact_wf-d643140d-90df-4bd8-8546-9823996c3f5b_text_markdown.md` — production reference for high-volume TEIBTO context (TS1-MRP, PWOC, WCS, TMS); this skill ดึงเนื้อหาหลักมาจากที่นี่

---

## 19. Skill Usage Notes (สำหรับ Claude)

- ก่อนเขียน SuiteQL ให้ user เสมอ → check §2 (syntax requirements) + §14 (governance limits)
- ถ้า query มีโอกาส scan เกิน 10,000 rows → แนะนำ §8 (perf) + §15 (pagination patterns)
- ถ้า user เขียนสำหรับ Connect (warehouse/BI) → **ห้ามใช้** WITH/CTE และ LISTAGG
- ถ้า user ขอ string concat ให้ใช้ `||` เท่านั้น
- ถ้า user ใช้ date literal ตรง ๆ → แก้เป็น `TO_DATE` ทันที
- ถ้า user ต้อง `LISTAGG` / `DATEDIFF` / `LEFT` → แจ้งว่า unsupported + เสนอ alternative
- ถ้า user reference field ที่อาจเป็น calculated/WLONGVARCHAR → เตือน performance impact
- เมื่อใช้ใน N/query → reminder ว่า mapped result keys lowercase เสมอ
