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

## Deployment

Projects deploy via SDF (SuiteScript Definition Framework) bundles. The PRD template's §13 tracks bundle composition: script files, custom records/fields/lists, roles, saved searches, workflows.
