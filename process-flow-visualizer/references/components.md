# Component Catalog

Copy-paste blocks for parts **not** wired into `assets/template.html`. The full,
working versions of all of these live in `assets/example-import-landed-cost.html` —
when in doubt, open the example and lift the markup + matching CSS.

## Table of contents
1. Architecture tiers (tab 00)
2. Decision branch (if/else fork)
3. Option box (sub-options inside a stage)
4. Interactive selector matrix (e.g. Incoterm responsibility)
5. Ledger / impact block for Step-by-Step (with show/hide toggle)
6. Numeric meter + breakdown for Simulation

---

## 1. Architecture tiers (tab 00)

Layered top→down diagram: each tier = a label column + a row of boxes, joined by
labelled down-connectors. End with a "cross-cutting concerns" strip.

Keep the `.aflow` connector text short (a few words like "create records") — it sits in
a narrow column and a long enumeration wraps into a cramped multi-line pill. The `.arch`
family fills the tab width; don't give it a `max-width` the other tabs lack.

```html
<div class="arch">
  <div class="atier">
    <div class="atier-label">External<br>Parties</div>
    <div class="arow">
      <div class="abox ext"><b>Party A</b><small>role</small></div>
      <!-- more .abox.ext -->
    </div>
  </div>
  <div class="aflow"><span>what flows down</span></div>

  <div class="atier">
    <div class="atier-label">Domains</div>
    <div class="arow">
      <div class="abox" style="--c:var(--cyan)"><span class="ad"></span><b>Lane A</b><small>x · y</small></div>
      <!-- color each box via --c -->
    </div>
  </div>
  <div class="aflow"><span>create records</span></div>

  <div class="atier">
    <div class="atier-label">Core<br>Records</div>
    <div class="arow"><div class="abox rec"><b>Record</b><small>note</small></div></div>
  </div>
  <div class="aflow hot"><span>feed the engine</span></div>

  <!-- the highlighted central engine -->
  <div class="aengine">
    <div class="ae-head"><span class="ae-badge">★ Engine</span><h3>Central Engine</h3></div>
    <div class="ae-steps">
      <div class="ae-step"><span>A</span>Step A</div><div class="ae-arr">→</div>
      <div class="ae-step gate"><span>B</span>Gate<small>condition</small></div>
    </div>
  </div>
  <div class="aflow hot"><span>output</span></div>

  <div class="atier">
    <div class="atier-label">System of<br>Record</div>
    <div class="arow"><div class="abox sor"><b>Output</b><small>note</small></div></div>
  </div>
</div>
<p class="arch-note">อ่านบนลงล่าง … (อธิบายทิศทาง + บอกว่าไม่ใช่ 1:1)</p>
<div class="across">
  <div class="across-h">Cross-cutting concerns</div>
  <div class="across-list"><span class="xc">Concern 1</span><span class="xc">Concern 2</span></div>
</div>
```

CSS to copy from the example: `.arch .atier .atier-label .arow .abox(.ext/.rec/.sor) .ad
.aflow(.hot) .aengine .ae-head .ae-badge .ae-step(.gate) .ae-arr .arch-note .across .xc`.
Responsive: in the mobile media query add `.atier,.aflow{grid-template-columns:1fr}.ae-arr{display:none}`.

### Box icons (do this — it makes the architecture read at a glance)

Give every `.abox` an inline-SVG icon in a tinted chip. Color it by the lane with
`currentColor`, **never** `var()` inside an SVG attribute (that renders nothing — known gotcha).
The chip inherits the box's `--c`, so the icon auto-matches its lane; external boxes go muted.

```html
<div class="abox" style="--c:var(--cyan)"><span class="ad"></span>
  <span class="aico"><svg viewBox="0 0 24 24"><!-- inner paths only, no stroke/fill attrs --></svg></span>
  <b>Lane A</b><small>x · y</small></div>
```
```css
/* icon sits to the LEFT of the title (icon | title-over-sublabel), one tidy row */
.abox{display:grid;grid-template-columns:auto 1fr;grid-template-areas:"ico ttl" "ico sub";column-gap:11px;row-gap:3px;align-items:center}
.abox>b{grid-area:ttl;align-self:end} .abox>small{grid-area:sub;align-self:start;margin-top:0}
.aico{grid-area:ico;align-self:center;display:inline-flex;width:32px;height:32px;border-radius:9px;align-items:center;justify-content:center;
  color:var(--c,var(--cyan));background:color-mix(in srgb,var(--c,var(--cyan)) 13%,transparent);
  border:1px solid color-mix(in srgb,var(--c,var(--cyan)) 28%,transparent)}
.aico svg{width:18px;height:18px;display:block;fill:none;stroke:currentColor;stroke-width:1.7;stroke-linecap:round;stroke-linejoin:round}
.abox.ext .aico{color:var(--faint);background:color-mix(in srgb,var(--muted) 11%,transparent);border-color:color-mix(in srgb,var(--muted) 22%,transparent)}
```

Pick the glyph that matches the box's meaning — drop the inner paths into the `<svg viewBox="0 0 24 24">` above:

| Meaning | Inner SVG paths |
|---|---|
| person · requester · customer · staff | `<circle cx="12" cy="8" r="3.2"/><path d="M5.8 19c0-3.5 2.8-5.4 6.2-5.4S18.2 15.5 18.2 19"/>` |
| company · vendor · supplier | `<rect x="6" y="3" width="12" height="18" rx="1"/><path d="M9.5 7h1.2M13.3 7h1.2M9.5 11h1.2M13.3 11h1.2M9.5 15h5"/>` |
| bank · treasury institution | `<path d="M3 9l9-5 9 5"/><path d="M5 9.5v8M9.3 9.5v8M14.7 9.5v8M19 9.5v8"/><path d="M3.5 20.5h17"/>` |
| carrier · logistics · shipment | `<path d="M3 6.5h10.5v9H3z"/><path d="M13.5 9.5h3.5l3 3v3h-6.5z"/><circle cx="7" cy="18" r="1.6"/><circle cx="17.5" cy="18" r="1.6"/>` |
| document · PR · invoice · bill | `<path d="M7 3h6.5L18 7.5V21H7z"/><path d="M13.3 3v4.3H18"/><path d="M9.7 12.5h5M9.7 15.5h5"/>` |
| order · PO · procurement | `<circle cx="9.5" cy="19" r="1.3"/><circle cx="16.5" cy="19" r="1.3"/><path d="M3 4h2.2l2 11h10l2-7.5H6.5"/>` |
| warehouse · receipt · inventory | `<path d="M12 3.2l8 4.4v8.8L12 20.8l-8-4.4V7.6z"/><path d="M4.2 7.8l7.8 4.3 7.8-4.3M12 12.1v8.5"/>` |
| money · payment · finance · A/P | `<rect x="3" y="6.2" width="18" height="11.6" rx="1.6"/><circle cx="12" cy="12" r="2.6"/><path d="M6.2 9.2v5.6M17.8 9.2v5.6"/>` |
| ledger · GL · subledger | `<ellipse cx="12" cy="6" rx="7" ry="2.6"/><path d="M5 6v6c0 1.5 3.1 2.6 7 2.6s7-1.1 7-2.6V6M5 12v6c0 1.5 3.1 2.6 7 2.6s7-1.1 7-2.6v-6"/>` |
| audit · control · customs · compliance | `<path d="M12 3l7 3v5c0 4.4-3 7.9-7 9.9-4-2-7-5.5-7-9.9V6z"/><path d="M9.2 11.8l2.1 2.1 3.8-4"/>` |
| approve · match · gate · check | `<circle cx="12" cy="12" r="8.5"/><path d="M8.2 12.4l2.6 2.6 5-5.6"/>` |

The import/landed-cost example uses these on every architecture box — lift them from there verbatim.

---

## 2. Decision branch (if/else fork)

For a point where the flow forks on a condition (e.g. "period open vs closed").

```html
<div class="branch">
  <div class="bcond"><span class="diamond">◆</span>
    <div><div class="bq">เงื่อนไข?</div><div class="bnote">ตัดสินใจปลายทาง</div></div></div>
  <div class="boutcomes">
    <div class="bcard open"><div class="bh"><span class="bbadge ok">YES</span><h4>ทางที่ 1</h4></div>
      <ul><li>ผลลัพธ์…</li></ul></div>
    <div class="bcard closed"><div class="bh"><span class="bbadge no">NO</span><h4>ทางที่ 2</h4></div>
      <ul><li>ผลลัพธ์…</li></ul></div>
  </div>
</div>
```
Classes: `.branch .bcond .diamond .bq .bnote .boutcomes .bcard(.open/.closed) .bh .bbadge(.ok/.no)`.
`.bbadge` text uses `var(--on-accent)`. Mobile: `.boutcomes{grid-template-columns:1fr}`.

---

## 3. Option box (sub-options inside a stage)

A bordered panel that holds 2-4 lettered sub-options (A/B/C/D) — good for
"these steps are optional / alternative paths".

```html
<div class="opt-box">
  <div class="sbh"><span class="chip ap">Options</span><h3>หัวข้อ</h3></div>
  <div class="opt-grid">
    <div class="ocell"><div class="on">A</div><h4>Option A</h4><p>…</p>
      <div class="fields"><span class="field">field</span></div></div>
    <!-- more .ocell -->
  </div>
  <p class="opt-note">⊙ หมายเหตุสำคัญ <b>เน้น</b></p>
</div>
```
Also mark a single optional `.step` with: add class `opt` and a badge
`<span class="optbadge">OPTION</span>` in its `.sh`.

---

## 4. Interactive selector matrix

A row of selector buttons that re-renders a responsibility table (the Incoterm
pattern: pick a code → see who's responsible for each component → what enters the pool).

Markup: `.incoterm-box` with `.ib-tabs` (buttons `data-ic="CODE"`), `.ib-matrix`
(rows injected by JS), `.ib-pool` (summary), `.ib-prep` (note). JS pattern:

```js
var ROWS=[{k:'freight',label:'Main freight'}, /* … */];
var MATRIX={EXW:{freight:'B'}, FOB:{freight:'B'}, CIF:{freight:'S'} /* B=buyer,S=seller */};
function renderMatrix(code){
  var m=MATRIX[code], box=document.getElementById('icMatrix'); box.innerHTML=''; var pool=[];
  ROWS.forEach(function(r){var buyer=m[r.k]==='B';
    box.insertAdjacentHTML('beforeend','<div class="ic-row'+(buyer?' buyer':'')+'"><span class="ic-lab">'+r.label+'</span><span class="ic-badge '+(buyer?'b':'s')+'">'+(buyer?'Buyer':'Seller')+'</span></div>');
    if(buyer)pool.push(r.label);});
  document.getElementById('icPool').innerHTML=pool.map(function(p){return '<span class="ibp-chip">'+p+'</span>'}).join('');
}
document.querySelectorAll('.ibt').forEach(function(b){b.addEventListener('click',function(){
  document.querySelectorAll('.ibt').forEach(function(x){x.classList.remove('active')});b.classList.add('active');renderMatrix(b.dataset.ic);});});
renderMatrix('FOB');
```
Lift CSS for `.incoterm-box .ib-* .ic-row(.buyer) .ic-badge(.b/.s) .ibp-*` from the example.

---

## 5. Ledger / impact block for Step-by-Step

Optional "double-entry / impact" panel inside a step's aside. Domain-agnostic:
rename "GL / Dr / Cr" to whatever your impact model is (state change, side effects…).

Add to a STEP object: `gl:[{lab?:'label', dr:'…', cr:'…'}], glnote:'…'`. Render:

```js
function glHtml(s){
  if(!s.gl) return '<div class="gl-none">'+(s.glnote||'ไม่มีผล')+'</div>';
  return s.gl.map(function(e){return '<div class="gl-entry">'+(e.lab?'<div class="gle-lab">'+e.lab+'</div>':'')+
    '<div class="gl-line dr"><span class="drcr">Dr</span><span class="acct">'+e.dr+'</span></div>'+
    '<div class="gl-line cr"><span class="drcr">Cr</span><span class="acct">'+e.cr+'</span></div></div>';}).join('')+
    (s.glnote?'<div class="gl-none">⊙ '+s.glnote+'</div>':'');
}
```
Insert `<div class="sbs-gl"><div class="glh">Impact</div>'+glHtml(s)+'</div>` into the aside.
**Show/hide toggle** (great for mixed audiences): a `#glToggle` button + CSS
`#tab-steps.hide-gl .sbs-gl{display:none}` and JS toggling `hide-gl` on the panel.

---

## 6. Numeric meter + breakdown for Simulation

Shows a base value growing by allocated/added amounts, with a stacked bar and %.

```html
<div class="cost-meter" id="costMeter" hidden>
  <div class="cm-row"><span>Base</span><b id="cmBase">–</b></div>
  <div class="cm-row"><span>+ added</span><b id="cmLanded">–</b></div>
  <div class="cm-bar"><div class="cm-bar-base" id="cmBarBase"></div><div class="cm-bar-land" id="cmBarLand"></div></div>
  <div class="cm-row total"><span>Final</span><b id="cmFinal">–</b></div>
</div>
```
Use a money formatter and a breakdown array; reveal at the `final` event. Full logic
(including the completion summary `.sim-summary` with breakdown chips) is in the example.
**Always use real units** (฿, $, kg, %) — never abstract "u"; abstract units confuse users.
```
