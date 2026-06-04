# Process Flow Visualizer

A Claude Code **skill** that turns any multi-step process, workflow, pipeline, or system
into **one polished, self-contained interactive HTML page** — no build step, no external JS
(only Google Fonts). Drop the `.skill` into Claude Code and ask it to "วาด/visualize" a flow.

> เป็น skill ของชุด [`kingcomen/king-netsuite-skill`](https://github.com/kingcomen/king-netsuite-skill) —
> ใช้สร้างหน้าอธิบาย process (ERP / NetSuite / business flow) ให้ดูเป็นภาพแล้วคลิกเล่นได้ในไฟล์เดียวจบ

## What you get — one process, up to four synchronized tabs

| Tab | คืออะไร |
|---|---|
| **00 Architecture** | แผนภาพ layered (External → Domains → Records → Engine → Output) + cross-cutting concerns — แต่ละกล่องมี **inline-SVG icon** สีตาม lane |
| **01 Process Flow** | การ์ดแต่ละ stage ละเอียด: sub-steps, fields, decision branch, option box |
| **02 Simulation** | timeline กด play/ลากได้ (T+0 → T+N) เห็นเอกสาร/ข้อมูล/ยอดเงินค่อย ๆ โผล่ตามเวลา + completion summary |
| **03 Step by Step** | เดินทีละขั้น: role, input, action, เอกสารที่ออก, GL impact, prev/next + arrow keys + clickable rail |

**Always-on chrome:** dark/light toggle (จำค่า), full-screen, glossary modal, per-tab description,
persistent 4-lane colour legend. เลือกใช้เฉพาะแท็บที่ต้องการได้ — ลบที่เหลือออกได้อิสระ.

## Scope (this version)

- 4-lane colour system (`--cyan / --violet / --green / --amber`, `--red` = state accent) — consistent ทุกแท็บ
- Architecture เต็มความกว้าง + icon catalog (person, building, bank, truck, document, cart, box, money, ledger, shield, check)
- Light-mode palette + card elevation (กล่องไม่จืด), icon + title อยู่แถวเดียวกัน
- Data-driven engines: `EVENTS` (Simulation) + `STEPS` (Step by Step) เป็นหัวใจของ interactivity
- Real units only (฿, $, kg, %, days) — ไม่มี abstract "u"

## Tech Stack

| Layer | Technology |
|---|---|
| Output | Single self-contained `.html` (opens via `file://`) |
| Styling | Plain CSS, theme via CSS variables, `color-mix()` |
| Behaviour | Vanilla JS (tabs, theme persist, simulation engine, step engine) |
| Icons | Inline SVG, lane-coloured via `currentColor` (no icon library) |
| Fonts | Google Fonts (only external dependency) |

## Files

| Path | Purpose |
|---|---|
| `SKILL.md` | Skill definition — workflow, conventions, gotchas, quality gate (read first) |
| `assets/template.html` | The framework to copy and fill (`<<ADJUST>>` markers) |
| `assets/example-import-landed-cost.html` | A complete real example — lift components verbatim |
| `references/components.md` | Copy-paste blocks: architecture tiers + icon catalog, decision branch, option box, selector matrix, ledger block, numeric meter |
| `evals/evals.json` | Eval prompts + assertions used to validate the skill |
| `../process-flow-visualizer.skill` | Packaged, installable bundle |

## Install

A `.skill` is a zip bundle — installing means unpacking it into Claude Code's skills folder
(`~/.claude/skills/`). Copy-paste one block:

**Git Bash / macOS / Linux**
```bash
git clone git@github.com:kingcomen/king-netsuite-skill.git
unzip -o "king-netsuite-skill/process-flow-visualizer/process-flow-visualizer.skill" -d ~/.claude/skills/
```

**Windows PowerShell** (`Expand-Archive` needs a `.zip` name, so copy first)
```powershell
git clone git@github.com:kingcomen/king-netsuite-skill.git
Copy-Item "king-netsuite-skill\process-flow-visualizer\process-flow-visualizer.skill" "$env:TEMP\pfv.zip"
Expand-Archive "$env:TEMP\pfv.zip" "$env:USERPROFILE\.claude\skills\" -Force
```

Both land the skill at `~/.claude/skills/process-flow-visualizer/`. **Restart Claude Code**
(or run `/doctor`) so it loads the new skill.

> Already cloned the repo? Skip `git clone` and run only the `unzip` / `Expand-Archive` line,
> pointing at your local `process-flow-visualizer/process-flow-visualizer.skill`.

## Use

Ask Claude, e.g. *"วาด flow ของ order-to-cash แบบมี simulation และ step by step"* or
*"ทำ architecture high-level + process flow ของ approval workflow ให้ดูเป็นภาพ มี dark/light"*.
Claude copies `template.html`, fills it from your process, runs the quality gate, and hands back one `.html`.

## Related skills

Part of [`kingcomen/king-netsuite-skill`](https://github.com/kingcomen/king-netsuite-skill):
- `/ns-suiteql`, `/netsuite-suiteql` — SuiteQL generation + reference
- `/teibto-ui-component` — Teibto Design System Web Components for Suitelet UI
