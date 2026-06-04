---
name: process-flow-visualizer
description: >-
  Build a polished, single-file interactive HTML visualization of any multi-step
  process, workflow, pipeline, or system — with dark/light theme, full-screen, a
  glossary, and up to four lenses on the SAME process: 00 Architecture (high-level
  layered diagram), 01 Process Flow (detailed stage cards), 02 Simulation (a
  play/scrub timeline showing data, documents and outputs appearing over time), and
  03 Step by Step (a guided one-step-at-a-time walkthrough). Use whenever the user
  wants to วาด/visualize/diagram/แสดงเป็นภาพ a business process, ERP/NetSuite flow,
  approval flow, data pipeline, order-to-cash, procure-to-pay, import/landed-cost,
  or any sequence of steps as an interactive page — even if they only say "make a
  flow", "show the steps", "architecture diagram", "process map", or "simulation".
  Also use to add tabs/theme/glossary to an existing flow HTML. Prefer over a static
  diagram whenever the result will be explored, presented, or clicked through.
---

# Process Flow Visualizer

Produces ONE self-contained `.html` file (no build step, no external JS except
Google Fonts) that explains a process through up to four synchronized tabs sharing
one theme. It is the generalized recipe behind the import → landed-cost example in
`assets/example-import-landed-cost.html`.

## What you get
- **00 Architecture** — layered top→down diagram (external → domains → records → engine → output) + cross-cutting concerns. The big picture; put it first. Give each box an inline-SVG icon in a lane-colored chip (catalog + CSS in `references/components.md` §1) — it makes the layers read at a glance and is the single biggest polish win for this tab.
- **01 Process Flow** — detailed stage cards with sub-steps, fields, and special blocks (decision branches, option boxes, selector matrices).
- **02 Simulation** — a timeline (T+0 → T+N) you can play or scrub; an event log, the documents/outputs produced, and the data that accumulates, all updating in sync. Optional numeric meter + completion summary.
- **03 Step by Step** — a guided walkthrough: one step at a time with role, inputs, action, output document, optional ledger/impact, and the next trigger. Prev/Next + arrow keys + a clickable rail.
- **Always-on chrome**: dark/light toggle (persisted), full-screen button, a glossary modal, a per-tab description line, and a persistent color legend.

Drop any tab you don't need — they are independent. A simple linear process might
use only 01 + 03; a system overview might use only 00.

## Files in this skill
- `assets/template.html` — **start here.** The framework with theme/tabs/glossary/full-screen wired, plus working data-driven engines for Simulation and Step-by-Step. Fill every `<<ADJUST>>`.
- `assets/example-import-landed-cost.html` — a complete, real example. Lift any component from it verbatim.
- `references/components.md` — copy-paste snippets for blocks not in the template (architecture tiers, decision branch, option box, selector matrix, ledger block, numeric meter). Read it when you need one of those.

## Workflow
1. **Model the process first, in plain words.** List the lanes/domains (≤4 for the color system), the stages in order, the documents/outputs each stage produces, and any decision forks or optional paths. Confirm the spine with the user if it's ambiguous — a wrong process model is the only thing that makes the output useless.
2. **Copy `assets/template.html`** to the output path. Set the title, `<h1>`, summary, footer, and the 4 lane labels in the legend (`.glegend`) + `STAGE_COLOR` mapping.
3. **Fill the tabs you need**, deleting the rest (remove both the `<button>` and the `<div class="tab-panel">`). Pull advanced blocks from `references/components.md` / the example.
4. **Populate the two data arrays** — `EVENTS` (Simulation) and `STEPS` (Step by Step). These are the heart of the interactivity; everything else renders from them.
5. **Validate before delivering** (see Quality gate).
6. Save to the outputs directory and present the file.

## Conventions (carry these over to every build)
- **One concept, four lenses.** All tabs describe the *same* process. Keep stage names, colors, and document names identical across tabs so users can cross-reference.
- **4-color lane system**, consistent everywhere: `--cyan / --violet / --green / --amber`. Map each to a lane once and never reuse a color for a different meaning. The legend explains them.
- **Real units only** in numbers (฿, $, kg, %, days) — never abstract "u".
- **`<<ADJUST>>` marks every spot you must edit.** None may survive into the final file.
- **No company/client names baked into reusable parts** unless this build is for a specific deliverable.
- **Theme via CSS variables only.** Never hard-code a hex in markup or inline style for anything themed — use `var(--…)`. Text/icons sitting on a bright accent fill must use `var(--on-accent)`, never `var(--bg)`.
- Keep it a **single file**. Fonts from Google Fonts are fine; nothing else external.
- **`.kicker` is a per-tab section label, not a page banner.** Use it *inside* a tab to title that lens (the example does "High-Level Architecture · Full Process", "Simulation · Lifecycle T+0 → T+35"). Don't add a kicker above the `<h1>` naming the whole process — the `<footer>` already carries that, so a header kicker just reads as a duplicate of the footer.
- **Keep `.aflow` connector labels short** — a few words like "create records" / "feed the engine". They live in a narrow column; a long enumeration wraps into a cramped multi-line pill. Put the detail in the boxes, not the connector.

## Data shapes
```js
// Simulation
EVENTS = [{ t:Number,            // position on 0..TMAX timeline
            stage:1|2|3|4,        // lane -> color + which pipe node lights up
            title, note,
            doc:{icon,name,meta,tag,tagc},  // a document/output produced here
            fields:[...],         // data that becomes "known" at this point
            final:true }]         // (last event) reveals the completion summary

// Step by Step
STEPS = [{ n:Number, role, rc:'var(--…)', sh:'short rail label',
           title, what, input:[...], action,
           doc:{ic,name}, next, opt:true,
           gl:[{lab?,dr,cr}], glnote }]  // gl/glnote optional (see components.md #5)
```

## Critical gotchas (these are battle scars — honor them)
- **A modal with `display:flex` will not close via the `hidden` attribute.** You MUST also add `.modal[hidden]{display:none}` (higher specificity than the class). The template already does this for the glossary; replicate it for any new overlay.
- **`var()` does NOT work in an SVG presentation attribute.** `stroke="var(--x)"` renders nothing. Use inline style instead: `style="stroke:var(--x)"` (or `stroke="currentColor"` + set `color`). The template's arrow already uses the style form.
- **Contrast on accents flips between themes.** Bright dark-mode accents want dark text; darker light-mode accents want white text. Solve it once with `--on-accent` (dark in dark theme, white in light) and use it for ALL text/icons on accent fills (`.num`, badges, `.ctrl.primary`, big step numbers…).
- **Don't float controls over a long title.** Top-right `position:absolute` controls overlap a wrapping `<h1>`. The template puts controls in a flex `.topbar` row beside the title so they reserve space and wrap below on narrow screens.
- **Simulation speed: tune the per-frame increment, not the frame rate.** In `tick()` the `t += INC*speed` constant sets duration: total seconds ≈ `TMAX / (INC*60)`. Aim for ~15–25 s at 1×. Too large = "วิ่งเร็วมาก".
- **Mobile:** the template's media query stacks `.sim-grid` and `.sbs-body`; if you add 2-column blocks (branch, matrix, architecture tiers) add their `grid-template-columns:1fr` overrides too.
- **Light mode is not just inverted tokens.** White panels on a near-white `--bg` with shadows tuned for dark read as flat and washed-out ("จืด"). The template's `[data-theme="light"]` block earns its keep: a deeper `--bg` + firmer `--line` so cards separate, a soft elevation shadow on the card classes, and a real surface for transparent/dashed external boxes (`.abox.ext`) so they don't disappear. Keep it when retheming; if you invent a new card type, add it to the light elevation list or it will look flat in light mode only.
- **Architecture fills the width like every other tab.** Don't cap `.arch` / `.arch-note` / `.across` with a `max-width` that `.stage` / `.sim-grid` / `.sbs-body` don't have — on a wide screen the Architecture tab then sits as a lonely centered column while every other tab is full-bleed, and the mismatch reads as a bug.

## Clarity rules (so first-time users don't get lost)
- Every tab gets a one-line description (`TAB_DESC` map) and the shared legend stays visible on all tabs.
- Put a one-time **affordance hint** next to anything clickable that doesn't look like a button (selector matrix, step rail): e.g. "↘ กดลำดับด้านล่าง / ↔ ลากแถบ".
- Define jargon in the **glossary** rather than inline; one `.gloss-item` per term. Include the *why* for non-experts (e.g. why a closed accounting period can't be edited).
- For audience-mixed detail (e.g. accounting entries), add a **show/hide toggle** so non-specialists can collapse it.

## Quality gate (run before presenting)
```bash
# 1) DOM balance (ignore tags inside comments)
python3 -c "import re,sys;h=open(sys.argv[1]).read();h=re.sub(r'<!--.*?-->','',h,flags=re.S);print('div',h.count('<div'),h.count('</div>'));print('section',h.count('<section'),h.count('</section>'))" FILE.html
# 2) JS syntax
sed -n '/<script>/,/<\/script>/p' FILE.html | sed '1d;$d' > /tmp/c.js && node --check /tmp/c.js && echo "JS OK"
# 3) no leftover markers
grep -c '<<ADJUST' FILE.html   # must be 0
```
Also eyeball: toggle dark↔light (nothing unreadable), open+close the glossary, play+scrub the simulation to the end (summary appears), walk the steps to the last one (Next disables). Then `present_files`.

## Test prompts this skill should handle
- "วาด flow ของ order-to-cash แบบมี simulation และ step by step"
- "ทำ architecture high-level + process flow ของ approval workflow ให้ดูเป็นภาพ มี dark/light"
- "เปลี่ยน process นำเข้าเดิมให้เป็น onboarding พนักงานใหม่ ใช้รูปแบบเดิม"
