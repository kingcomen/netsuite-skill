---
name: customhtml
description: Teibto Design System (TBT-DS) — Lit Web Components for NetSuite Suitelet custom HTML pages. Use this skill whenever building, editing, or reviewing any Teibto Suitelet UI that uses custom HTML (not serverWidget). Triggers on requests like "สร้าง Suitelet", "เขียนหน้า UI", "เพิ่ม component", "ทำ form", "สร้าง table", "ออกแบบ page NetSuite", or any mention of tbt-* components. Always consult this skill before writing any HTML for a Teibto Suitelet — never use raw HTML primitives when a tbt-* component exists.
---

# Teibto Design System (TBT-DS) — Skill Reference

> Read this entire file before writing any UI for Teibto Suitelet pages.
> Component prefix: `tbt-` | File Cabinet: `/SuiteScripts/Teibto/ds/v{X.Y.Z}/`

---

## 1. Architecture Overview

```
Suitelet (sl_xxx.js)
  └─ response.write(html)
       └─ sl_xxx.html  (in File Cabinet)
            ├─ <link> tbt-theme.css    ← design tokens
            ├─ <script> tbt-ds.min.js  ← all tbt-* components
            └─ <body> composed with tbt-* elements only
```

**No build step in dev** — Lit loads from CDN as ES module.  
**Production** — bundle to `tbt-ds.min.js` via Rollup, deploy to File Cabinet.

### Standard page include
```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Teibto · [Page Name]</title>
  <link rel="stylesheet" href="/sc/SuiteScripts/Teibto/ds/v1.0.0/tbt-theme.css">
  <link rel="stylesheet" href="/sc/SuiteScripts/Teibto/ds/v1.0.0/tbt-icons.css">
  <script type="module" src="/sc/SuiteScripts/Teibto/ds/v1.0.0/tbt-ds.min.js"></script>
</head>
<body class="tbt-page">
  <tbt-app-shell>
    <!-- page content here -->
  </tbt-app-shell>
</body>
</html>
```

---

## 2. Design Tokens (tbt-theme.css)

All visual decisions live here. Components use `var(--tbt-*)` only — never hardcode hex values.

### Brand Colors
```css
/* Primary — Navy Blue (from logo TEIBT letters) */
--tbt-primary:       #0D1171;
--tbt-primary-dark:  #080C55;
--tbt-primary-light: #1E2B99;
--tbt-primary-bg:    #EEF0FF;
--tbt-primary-text:  #0D1171;

/* Accent — Gradient (from logo O) */
--tbt-accent-purple:   #8B35C8;
--tbt-accent-blue:     #59BBF6;
--tbt-accent-gradient: linear-gradient(135deg, #8B35C8 0%, #59BBF6 100%);
```

### Surface Colors
```css
--tbt-bg-page:    #F5F7FA;
--tbt-bg-card:    #FFFFFF;
--tbt-bg-hover:   #F0F2FF;
--tbt-bg-active:  #E2E6FF;
--tbt-border:     #E2E8F0;
--tbt-border-strong: #CBD5E1;
```

### Text Colors
```css
--tbt-text-primary:   #0F172A;
--tbt-text-secondary: #64748B;
--tbt-text-muted:     #94A3B8;
--tbt-text-required:  #EF4444;
--tbt-text-link:      #0D1171;
```

### Semantic Colors
```css
--tbt-success:     #10B981;  --tbt-success-bg:  #ECFDF5;  --tbt-success-text:  #065F46;
--tbt-warning:     #F59E0B;  --tbt-warning-bg:  #FEF3C7;  --tbt-warning-text:  #92400E;
--tbt-danger:      #EF4444;  --tbt-danger-bg:   #FEE2E2;  --tbt-danger-text:   #991B1B;
--tbt-info:        #3B82F6;  --tbt-info-bg:     #DBEAFE;  --tbt-info-text:     #1E40AF;
```

### Typography
```css
--tbt-font:      'Inter', 'IBM Plex Sans Thai', system-ui, sans-serif;
--tbt-font-mono: 'JetBrains Mono', 'Courier New', monospace;

--tbt-size-xs:   11px;
--tbt-size-sm:   12px;
--tbt-size-base: 14px;
--tbt-size-md:   16px;
--tbt-size-lg:   20px;
--tbt-size-xl:   28px;

--tbt-weight-normal:   400;
--tbt-weight-medium:   500;
--tbt-weight-semibold: 600;
--tbt-weight-bold:     700;
```

### Spacing (4px scale)
```css
--tbt-space-1: 4px;   --tbt-space-2: 8px;   --tbt-space-3: 12px;
--tbt-space-4: 16px;  --tbt-space-5: 20px;  --tbt-space-6: 24px;
--tbt-space-8: 32px;  --tbt-space-10: 40px; --tbt-space-12: 48px;
```

### Radius, Shadow, Transition
```css
--tbt-radius-sm: 6px;   --tbt-radius-md: 8px;
--tbt-radius-lg: 12px;  --tbt-radius-pill: 9999px;

--tbt-shadow-sm:    0 1px 2px rgb(13 17 113 / 0.06);
--tbt-shadow-md:    0 4px 12px rgb(13 17 113 / 0.08);
--tbt-shadow-focus: 0 0 0 3px rgb(139 53 200 / 0.20);

--tbt-transition-fast: 100ms ease;
--tbt-transition-base: 150ms ease;
```

---

## 3. Component Inventory

| Component | Tag | Status | Description |
|---|---|---|---|
| App Shell | `<tbt-app-shell>` | v1 | Page wrapper with menubar + content area |
| Menubar | `<tbt-menubar>` | v1 | Top navigation bar |
| Sidebar | `<tbt-sidebar>` | v1 | Collapsible side navigation |
| Subtab | `<tbt-subtab>` | v1 | Tab navigation within a page |
| Button | `<tbt-button>` | v1 | Action button (primary, secondary, danger, ghost) |
| Badge | `<tbt-badge>` | v1 | Status indicator |
| Field | `<tbt-field>` | v1 | Label + value display pair |
| Field Grid | `<tbt-field-grid>` | v1 | Responsive grid of fields |
| Form | `<tbt-form>` | v1 | Form wrapper with submit handling |
| Search | `<tbt-search>` | v1 | Search input with debounce |
| Dropdown | `<tbt-dropdown>` | v1 | Select / dropdown list |
| Multi-select | `<tbt-multiselect>` | v1 | Multiple selection input |
| Date Picker | `<tbt-datepicker>` | v1 | Date input with calendar popup |
| Table | `<tbt-table>` | v1 | Data table with sort, pagination |
| Section | `<tbt-section>` | v1 | Content section with optional title |
| Summary | `<tbt-summary>` | v1 | KPI summary cards row |

---

## 4. Component API

### tbt-button
```html
<tbt-button variant="primary">Save</tbt-button>
<tbt-button variant="secondary">Cancel</tbt-button>
<tbt-button variant="danger">Delete</tbt-button>
<tbt-button variant="ghost" icon="printer">Print</tbt-button>
<tbt-button variant="primary" loading>Saving…</tbt-button>
<tbt-button variant="accent">ปุ่ม gradient</tbt-button>
```
Props: `variant` (primary|secondary|danger|ghost|accent), `icon` (Tabler icon name), `loading`, `disabled`, `size` (sm|md|lg)

### tbt-field + tbt-field-grid
```html
<tbt-field-grid columns="3">
  <tbt-field label="Document No." value="SO-0001"></tbt-field>
  <tbt-field label="Date" value="2026-05-21"></tbt-field>
  <tbt-field label="Status" required>
    <tbt-badge variant="success">Approved</tbt-badge>
  </tbt-field>
</tbt-field-grid>
```
Props: `label`, `value`, `required`, `muted`

### tbt-table
```html
<tbt-table
  .columns=${[
    { key: 'tranid', label: 'Document No.', sortable: true },
    { key: 'date',   label: 'Date',         sortable: true },
    { key: 'amount', label: 'Amount',       align: 'right' }
  ]}
  .rows=${data}
  paginate
  page-size="50">
</tbt-table>
```
Props: `columns` (Array), `rows` (Array), `paginate`, `page-size`, `loading`, `empty-message`

### tbt-search
```html
<tbt-search
  placeholder="ค้นหาเอกสาร..."
  debounce="300"
  @tbt-search=${e => handleSearch(e.detail.value)}>
</tbt-search>
```

### tbt-dropdown
```html
<tbt-dropdown
  label="สถานะ"
  .options=${[{value:'A', label:'Approved'}, {value:'P', label:'Pending'}]}
  value="A"
  @tbt-change=${e => handleChange(e.detail.value)}>
</tbt-dropdown>
```

### tbt-multiselect
```html
<tbt-multiselect
  label="Subsidiary"
  .options=${subsidiaries}
  .value=${selected}
  @tbt-change=${e => handleChange(e.detail.values)}>
</tbt-multiselect>
```

### tbt-datepicker
```html
<tbt-datepicker
  label="วันที่เอกสาร"
  value="2026-05-21"
  format="YYYY-MM-DD"
  @tbt-change=${e => handleDate(e.detail.value)}>
</tbt-datepicker>
```

### tbt-subtab
```html
<tbt-subtab>
  <tbt-tab label="ข้อมูลทั่วไป" active>
    <!-- content -->
  </tbt-tab>
  <tbt-tab label="รายการสินค้า">
    <!-- content -->
  </tbt-tab>
</tbt-subtab>
```

### tbt-menubar
```html
<tbt-menubar logo="/sc/SuiteScripts/Teibto/assets/teibtologo.png">
  <tbt-menu-item href="/app/site/hosting/scriptlet.nl?script=xxx" label="หน้าหลัก"></tbt-menu-item>
  <tbt-menu-group label="ขาย">
    <tbt-menu-item href="..." label="ใบเสนอราคา"></tbt-menu-item>
  </tbt-menu-group>
</tbt-menubar>
```

### tbt-sidebar
```html
<tbt-sidebar collapsible>
  <tbt-sidebar-item icon="home" label="Dashboard" href="..."></tbt-sidebar-item>
  <tbt-sidebar-item icon="file-invoice" label="เอกสาร" active></tbt-sidebar-item>
</tbt-sidebar>
```

---

## 5. Composition Patterns

### Document view page (standard)
```html
<tbt-app-shell>
  <tbt-menubar slot="menubar" ...></tbt-menubar>
  <main slot="content">
    <tbt-section title="Quotation · QT-0001">
      <tbt-field-grid columns="4">
        <tbt-field label="Document No." value="QT-0001"></tbt-field>
        <tbt-field label="Customer" value="บริษัท ABC"></tbt-field>
        <tbt-field label="Date" value="21 May 2026"></tbt-field>
        <tbt-field label="Status">
          <tbt-badge variant="warning">Pending</tbt-badge>
        </tbt-field>
      </tbt-field-grid>
    </tbt-section>

    <tbt-section title="รายการสินค้า">
      <tbt-table .columns=${cols} .rows=${lines}></tbt-table>
    </tbt-section>

    <tbt-section>
      <tbt-summary>
        <tbt-summary-item label="Subtotal" value="100,000"></tbt-summary-item>
        <tbt-summary-item label="VAT 7%" value="7,000"></tbt-summary-item>
        <tbt-summary-item label="Total" value="107,000" highlight></tbt-summary-item>
      </tbt-summary>
    </tbt-section>

    <footer>
      <tbt-button variant="primary" icon="device-floppy">Save</tbt-button>
      <tbt-button variant="secondary">Cancel</tbt-button>
    </footer>
  </main>
</tbt-app-shell>
```

### List / search page
```html
<tbt-section title="รายการเอกสาร">
  <div slot="actions">
    <tbt-search placeholder="ค้นหา..." @tbt-search=${onSearch}></tbt-search>
    <tbt-button variant="primary" icon="plus">New</tbt-button>
  </div>
  <tbt-table .columns=${cols} .rows=${rows} paginate page-size="50"></tbt-table>
</tbt-section>
```

### การส่งข้อมูลจาก Suitelet JS → HTML
```javascript
// sl_xxx.js (server side)
const data = { tranId: 'QT-0001', lines: [...] };
const html = `
  <script>
    window.__TBT_DATA__ = ${JSON.stringify(data)};
  </script>
`;
// inject ก่อน </body>
```
```javascript
// ใน HTML (client side)
const data = window.__TBT_DATA__;
```

### การส่งข้อมูลกลับ server (RESTlet call)
```javascript
async function save(payload) {
  const res = await fetch('/app/site/hosting/restlet.nl?script=xxx&deploy=1', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
  return res.json();
}
```

---

## 6. Governance Rules (Hard — ห้ามละเมิด)

1. **ห้าม hex สี** ใน component JS — ใช้ `var(--tbt-*)` เท่านั้น
2. **ห้าม `<style>` block** ใน consumer page — style อยู่ใน component เท่านั้น
3. **ห้าม `style="..."` inline** สำหรับ visual — ยกเว้น grid-column หรือ token override
4. **ห้ามใช้ raw HTML primitive** เมื่อมี tbt-* component แล้ว
5. **Sentence case** สำหรับ label ทุกตัว ("Document info" ไม่ใช่ "Document Info")
6. **ระบุ version ชัดเจน** ใน path — ห้ามใช้ `/latest/`
7. **Code เป็น English** — comment อาจเป็นไทยผสม

---

## 7. File Cabinet Path Convention

```
/SuiteScripts/Teibto/ds/v1.0.0/
  tbt-theme.css
  tbt-icons.css
  tbt-ds.min.js

/SuiteScripts/Teibto/{module}/
  sl_{module}_{action}.js    ← Suitelet server
  sl_{module}_{action}.html  ← HTML template

/SuiteScripts/Teibto/assets/
  teibtologo.png
```

---

## 8. Common Pitfalls (จากประสบการณ์จริง)

| ❌ ผิด | ✅ ถูก |
|---|---|
| `<button class="btn-primary">` | `<tbt-button variant="primary">` |
| สีใน component JS: `color: #0D1171` | `color: var(--tbt-primary)` |
| `response.write` ส่ง HTML inline ยาวๆ | โหลด HTML จาก File Cabinet แทน |
| ไม่ระบุ `window.__TBT_DATA__` type | ใส่ JSON.stringify + parse ฝั่ง client เสมอ |
| fetch ไป Suitelet ด้วย GET + body | ใช้ RESTlet สำหรับ POST data |
| `<script src="https://...">` ใน NS | ตรวจ CSP ก่อน — ใช้ File Cabinet เป็น fallback |
| Shadow DOM query: `document.querySelector('tbt-button button')` | ใช้ event / property ของ component แทน |
| `import { html } from 'lit'` ใน browser โดยไม่มี importmap | ใช้ full CDN URL หรือ bundle แล้ว |

---

## 9. Component File Template

```javascript
/**
 * @component tbt-xxx
 * @version 1.0.0
 * @author Wichit Wongta
 *
 * [Brief description and when to use]
 */
import { LitElement, html, css } from 'https://cdn.jsdelivr.net/npm/lit@3/+esm';

class TbtXxx extends LitElement {
  static properties = {
    propA: { type: String, reflect: true },
    propB: { type: Boolean }
  };

  static styles = css`
    :host { display: block; }
    /* var(--tbt-*) tokens only */
  `;

  render() {
    return html`<slot></slot>`;
  }
}

customElements.define('tbt-xxx', TbtXxx);
```

---

## 10. Dark Mode

TBT-DS รองรับ dark mode ผ่าน `@media (prefers-color-scheme: dark)` ใน `tbt-theme.css`  
Surface และ text tokens จะ override อัตโนมัติ — brand colors (`--tbt-primary`, gradient) คงเดิม

---

## 11. Versioning

| Change | Version bump |
|---|---|
| Bug fix, style tweak | PATCH (1.0.x) |
| New component, new prop | MINOR (1.x.0) |
| Renamed prop, removed variant | MAJOR (x.0.0) + migration guide |

อัปเดต `package.json`, `CHANGELOG.md`, และ File Cabinet path ทุกครั้งที่ bump version
