# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

This is **the source of a Claude Code Skill**, not an app. The skill (`process-flow-visualizer`)
generates ONE self-contained interactive `.html` file that explains a multi-step process
through up to four synchronized tabs sharing one theme: **00 Architecture**, **01 Process Flow**,
**02 Simulation**, **03 Step by Step**.

`SKILL.md` is the authoritative spec — read it first. It is loaded into Claude's context when the
skill triggers, so its prose (workflow, conventions, gotchas, quality gate) is the real "code of
conduct" here. When you change skill behavior, you are usually editing `SKILL.md` and/or
`template.html`, not adding program logic.

## File layout

The canonical structure lives inside the packaged bundle `process-flow-visualizer.skill` (a zip):

```
process-flow-visualizer/
  SKILL.md                                  # skill definition: frontmatter (name/description) + instructions
  references/components.md                  # copy-paste blocks NOT in the template (arch tiers, decision branch,
                                            #   option box, selector matrix, ledger block, numeric meter)
  assets/template.html                      # the framework to copy + fill (start here for any build)
  assets/example-import-landed-cost.html    # a complete real example; lift components verbatim
```

The loose files at the directory root (`SKILL.md`, `template.html`) are an export and are byte-identical
to their counterparts in the bundle. `files (15).zip` is a delivery wrapper around the three loose files.
The `.skill` bundle is the artifact that gets installed.

## How the generated HTML works (template.html architecture)

A user-facing build = copy `template.html`, then fill every `<<ADJUST>>` marker (none may survive into
the final file). Three machines drive it; keep them intact and only change data/labels:

1. **Theme + tabs + glossary chrome** (top of `<script>`): dark/light toggle persisted to localStorage,
   full-screen, glossary modal, and a `TAB_DESC` map (one description line per tab). Tabs are plain
   `data-tab` buttons paired with `#tab-*` panels — delete a tab by removing BOTH its `<button>` and its
   `<div class="tab-panel">`.
2. **Simulation engine**: renders entirely from the `EVENTS` array + `STAGE_COLOR` map + `TMAX`. `tick()`
   advances `t += 0.5*speed` per frame; the `0.5` constant (not the frame rate) sets total duration
   (≈ `TMAX / (0.5*60)` seconds at 1×). The event with `final:true` reveals the completion summary.
3. **Step-by-Step engine**: renders from the `STEPS` array; prev/next + arrow keys + a clickable rail.

**`EVENTS` and `STEPS` are the heart of the interactivity** — everything else renders from them. Their
exact shapes are documented in `SKILL.md` ("Data shapes"). All four tabs must describe the SAME process:
keep stage names, the 4-lane colors, and document names identical across tabs.

## The 4-lane color system (invariant)

Exactly four lane colors — `--cyan / --violet / --green / --amber` — map to four domains via `STAGE_COLOR`
and the `.glegend` legend. Never reuse a color for a different meaning. Everything themed must use CSS
variables; text/icons on a bright accent fill use `var(--on-accent)`, never `var(--bg)`.

## Validation (there is no build/lint/test toolchain)

Run the quality gate from `SKILL.md` before delivering a generated file. On this Windows host, use the
Bash tool (bash + python3 + node are available there); the commands are POSIX:

```bash
# 1) DOM balance (strips comments first)
python3 -c "import re,sys;h=open(sys.argv[1]).read();h=re.sub(r'<!--.*?-->','',h,flags=re.S);print('div',h.count('<div'),h.count('</div>'));print('section',h.count('<section'),h.count('</section>'))" FILE.html
# 2) JS syntax
sed -n '/<script>/,/<\/script>/p' FILE.html | sed '1d;$d' > /tmp/c.js && node --check /tmp/c.js && echo "JS OK"
# 3) no leftover markers (must print 0)
grep -c '<<ADJUST' FILE.html
```

Then eyeball: toggle dark↔light, open/close glossary, play+scrub the simulation to the end (summary
appears), walk steps to the last one (Next disables).

## Battle-scar gotchas (carry these into every build)

These are in `SKILL.md` but are the ones that silently break output:

- A modal with `display:flex` won't close via the `hidden` attribute alone — also add
  `.modal[hidden]{display:none}` (the template does this for the glossary; replicate for any new overlay).
- `var()` does NOT work inside an SVG presentation attribute (`stroke="var(--x)"` renders nothing) —
  use inline `style="stroke:var(--x)"` or `stroke="currentColor"`.
- Put theme/fullscreen controls in the flex `.topbar` row, not `position:absolute` — absolute controls
  overlap a wrapping `<h1>`.

## Repackaging the skill

After editing `SKILL.md` / `template.html` / `references/` / `assets/`, rebuild the bundle so the loose
files and the `.skill` zip stay in sync (they must remain byte-identical):

```bash
zip -r process-flow-visualizer.skill process-flow-visualizer/   # from a dir containing the canonical tree
```

## Relationship to the parent workspace

This skill lives inside the `netsuite-skill` workspace (see `../CLAUDE.md` for NetSuite naming
conventions, the PRD workflow, and the GitHub README requirement). Those rules apply if this skill is
ever pushed as its own repo — a README.md is required before pushing to GitHub.
