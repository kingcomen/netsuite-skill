# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a NetSuite development workspace owned by Wichit Wongta. It holds project PRDs and SuiteScript source for NetSuite customizations. The primary planning artifact is `PRD_TEMPLATE.md`, which is forked and filled in for each feature/module.

## PRD Workflow

Copy `PRD_TEMPLATE.md` → rename to `PRD-{ID}-{slug}.md` (e.g. `PRD-TEIBTO-IF-001-lot-tracking.md`) → fill in sections. The PRD ID format is `{PROJECT}-{MODULE}-{NNN}`.

## NetSuite Naming Conventions (Critical)

All custom objects require a **3-letter topic prefix** (not project prefix):

| Object type | Pattern | Example |
|---|---|---|
| Custom Record | `customrecord_XXX_<name>` | `customrecord_lot_scan_log` |
| Custom List | `customlist_XXX_<name>` | `customlist_apr_status` |
| Custom Transaction | `customtransaction_XXX_<name>` | `customtransaction_inv_adjustment` |
| Custom Field (body) | `custbody_XXX_<name>` | `custbody_lot_scan_ref` |
| Custom Field (record) | `custrecord_XXX_<name>` | `custrecord_lot_item` |

Script file naming: `cs_xxx.js` (Client Script), `ue_xxx.js` (User Event), `mr_xxx.js` (Map/Reduce), `lib_xxx.js` (shared library).

File placement: `/SuiteScripts/{project}/` with HTML assets in `/SuiteScripts/{project}/html/`.

## SuiteScript Governance Limits

Always track governance units in technical designs:

| Script Type | Unit Limit |
|---|---|
| User Event | 1,000 |
| Suitelet | 1,000 |
| Map/Reduce (map phase) | 1,000 |

## SuiteQL Skills

Use the built-in Claude Code skills for any SuiteQL work:
- `/ns-suiteql` — generate a SuiteQL query with automatic field guard against the 2026.1 schema
- `/netsuite-suiteql` — SuiteQL syntax reference (Oracle SQL vs SQL-92 rules, joins, BUILTIN.*, pagination)

## GitHub Repository Checklist

ทุก project ที่ push ขึ้น GitHub **ต้องมี README.md** ที่หน้าแรกของ repo อ่านแล้วเข้าใจได้ทันที ประกอบด้วย:

1. **ชื่อและคำอธิบาย** — project นี้ทำอะไร แก้ปัญหาอะไร
2. **ลิงก์ PRD** — ตาราง PRD ID, version, status พร้อม link ไปไฟล์จริง
3. **ขอบเขต** — In Scope ของ Phase ปัจจุบัน (bullet list)
4. **Tech Stack** — ตาราง Layer / Technology
5. **Skills ที่ใช้** — ลิงก์ไป `kingcomen/netsuite-skill` พร้อมระบุว่า skill ไหนใช้ทำอะไร

> README ต้องเป็นสิ่งแรกที่อ่านก่อนเปิด codebase — ห้าม push ขึ้น GitHub โดยไม่มี README.md

## Deployment

Projects deploy via SDF (SuiteScript Definition Framework) bundles. The PRD template's §13 tracks bundle composition: script files, custom records/fields/lists, roles, saved searches, workflows.
