---
name: ns-suiteql
description: Generate a SuiteQL query against the NetSuite 2026.1 schema export, with automatic guard that strips fields the tenant cannot use. TRIGGER when the user asks to write/build/generate a SuiteQL query, a NetSuite SQL report, an Inventory/Item/Transaction query, or anything starting with `/ns-suiteql`. Always run this skill before writing raw SuiteQL — never bypass the deny-list check.
---

# ns-suiteql — SuiteQL generator with deny-list guard

You are generating a SuiteQL query for a specific NetSuite tenant. The 2026.1 schema export (~164 MB) lives at the absolute path `~/Claude-Project/Netsuite/Schema/netsuite-schema.json` and lists every column the platform *could* expose, but per-tenant feature flags (License Plate, certain WMS modules, OneWorld-only, etc.) gate actual availability. The export does NOT mark these. You must consult a maintained deny-list before emitting any column.

**Path constants used by this skill** (do not hardcode elsewhere — keep references here):
- Schema export: `~/Claude-Project/Netsuite/Schema/netsuite-schema.json`
- Deny-list memory: `~/.claude/projects/C--Users-wichi-Claude-Project-Netsuite/memory/project_unsupported_schema_fields.md`
- Output directory: `<CWD>/Schema/` (current working directory of the Claude Code session, created if missing) — or whatever path the user specifies.

> **Companion skill:** `netsuite-suiteql` (in same `.claude/skills/` dir) is the SuiteQL syntax reference manual — Oracle vs SQL-92 rules, supported/unsupported functions, BUILTIN.* catalog, performance best practices, governance limits, pagination patterns, Connect vs N/query vs REST trade-offs. Apply its rules to whatever this skill produces. In particular: `||` not `+`, `TO_DATE` not date literals, no `[]` brackets, `SUBSTR` not `SUBSTRING`, `INSTR` not `LOCATE`, batch IN ≤ 1000, no `OFFSET` (use ROWNUM). **CTE/WITH** is OK in N/query + REST but **not in SuiteAnalytics Connect** — ถ้าต้อง portable หลีกเลี่ยง.

## Inputs to clarify (only if not given)

If the user's request is missing essentials, ask once via `AskUserQuestion`:
- Target table(s) — e.g. `InventoryBalance`, `transactionLine`, `item`
- Granularity — what each row represents
- Filters / scope — locations, item prefix, date range
- Output destination — `.sql` file in `Schema/` (default), inline answer, or other path

Do NOT ask for clarification on conventions covered below; just apply them.

## Procedure

1. **Load the deny-list** — read `~/.claude/projects/C--Users-wichi-Claude-Project-Netsuite/memory/project_unsupported_schema_fields.md` and parse the table → unusable column rows. Treat its table as the single source of truth.

2. **Resolve real columns** — for each requested table, extract its column list from the schema export at `~/Claude-Project/Netsuite/Schema/netsuite-schema.json`. The file is large; do NOT cat it. Use:
   ```bash
   node -e "const d=JSON.parse(require('fs').readFileSync(require('os').homedir()+'/Claude-Project/Netsuite/Schema/netsuite-schema.json','utf8'));const t=d.tables.find(x=>x.id==='<TABLE>');t.columns.forEach(c=>console.log(c.id+'\t'+c.dataType));"
   ```
   Or spawn an Explore agent if multiple tables / fuzzy lookup is needed.

3. **Apply guard** — before emitting the SQL, intersect requested columns with the deny-list. Any match is dropped silently from SELECT/JOIN/WHERE and recorded in an `OMITTED:` note returned with the query. Do not use a column you have not seen in step 2 — if the user named a column the schema doesn't have, surface that as a separate error and stop.

4. **Apply NetSuite conventions**
   - INTEGER FK columns (`item`, `location`, `inventoryStatus`, `subsidiary`, `class`, `department`, `entity`, `binNumber`, `inventoryNumber`) → output the id AND `BUILTIN.DF(<col>)` for the display name.
   - Standard joins when relevant: `item i ON i.id = <t>.item`, `bin bn ON bn.id = <t>.binNumber`, `inventoryNumber inv ON inv.id = <t>.inventoryNumber`, `location l ON l.id = <t>.location`.
   - Date math: use `TRUNC(SYSDATE)` not `SYSDATE` to avoid time-of-day drift; use `BUILTIN.RELATIVE_RANGES` only when the user explicitly asks for relative ranges.
   - Quantity rows: filter `quantityOnHand <> 0` (or `> 0` for on-hand-only reports) to skip the empty buckets NetSuite leaves around.
   - For aging anchors on lots, prefer `COALESCE(custitemnumber_twms_lot_receiptdate, custitemnumber_mfg_lotmanufacturingdate)` and add a `*_source` column flagging which path was used.
   - Don't reference `lastModifiedDate` or `<table>.createdDate` as cohort/aging anchors — they mutate or pre-date the current stock.

5. **Format the output**
   - Header comment block: purpose, source table(s), granularity, engine = SuiteQL.
   - Section comments dividing CTEs / pivots / data-quality checks if multiple queries.
   - Ordered `SELECT` with aligned `AS <snake_case>` aliases.
   - Filters under a `-- Filters` comment with common ones commented out as examples.

6. **Write to disk** — default path `<CWD>/Schema/<descriptive_name>.sql` (the `Schema/` folder of whatever the current Claude Code session's CWD is — create the folder if it doesn't exist) unless the user specified otherwise. Use Write (or Edit if updating). Then in the chat reply, summarise:
   - File path
   - Tables touched
   - **OMITTED columns** (if any) with reason: "in deny-list / feature gated"
   - Any caveats (e.g. fallback chosen, filter assumptions)

## Anti-patterns — never do these

- Emitting `licensePlateNumber` or any other deny-listed column.
- Inventing a column the schema export doesn't contain.
- Using `cat` / `head` on `~/Claude-Project/Netsuite/Schema/netsuite-schema.json` (164 MB will blow context).
- Silent fallback in date anchors — always add a flag column showing which date was used.
- Single-line CTE-less queries when the request involves bucketing or pivoting; readability first.

## Self-check before declaring done

- [ ] Every column in SELECT/JOIN/WHERE was confirmed present in schema export.
- [ ] No deny-list column slipped through.
- [ ] BUILTIN.DF added for INTEGER FK display columns the user will read.
- [ ] OMITTED list reported back to the user (or stated "none").
- [ ] File written to `Schema/` (or user-specified path).
