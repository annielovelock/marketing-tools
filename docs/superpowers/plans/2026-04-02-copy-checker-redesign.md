# Copy Checker Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild `charactercopychecker.html` with a sidebar-nav layout — platform list on the left, fields in a main panel on the right — replacing the current 2-column card grid.

**Architecture:** Three changes in sequence: (1) add CSS for the sidebar layout, (2) rewrite the HTML structure, (3) update the JS to render fields dynamically per selected platform. All existing JS logic (`LIMITS`, `state`, `renderField`, `buildCopyAllTextWithHeaders`, etc.) is preserved with minimal modifications.

**Tech Stack:** Vanilla HTML/CSS/JS, shared `styles.css`, Font Awesome 6.5.1 (CDN), no build step. Verification is manual in-browser — open the file locally and check.

---

## Files

- Modify: `charactercopychecker.html` — full rewrite of `<main>` content and `<script>` block
- Modify: `styles.css` — replace grid styles, add sidebar/panel/nav-item styles (all scoped to `.page-charcopy`)

---

### Task 1: CSS — Replace grid with sidebar layout

**Files:**
- Modify: `styles.css` lines 483–527 (the Character copy checker section)

- [ ] **Step 1: Replace the grid + card + meta CSS block**

In `styles.css`, replace the entire block from `/* ─── Character copy checker ──────────────────────────────── */` through `.page-charcopy .row .actions { margin-top: 0; }` (lines 474–527) with:

```css
/* ─── Character copy checker ──────────────────────────────── */

.page-charcopy .wrap { max-width: 1100px; }

.charcopy-layout {
  display: flex;
  align-items: flex-start;
  gap: 0;
  margin-top: 20px;
}

.charcopy-sidebar {
  width: 148px;
  flex-shrink: 0;
  position: sticky;
  top: 20px;
  padding-right: 20px;
}

.charcopy-sidebar-label {
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: .08em;
  color: var(--text-3);
  margin-bottom: 10px;
}

.charcopy-sidebar .toolbar {
  flex-direction: column;
  gap: 5px;
  margin: 0 0 14px;
}

.charcopy-sidebar .toolbar button {
  width: 100%;
  text-align: left;
  font-size: 12px;
  padding: 6px 10px;
}

.charcopy-nav {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.charcopy-nav-item {
  display: flex;
  align-items: center;
  gap: 7px;
  width: 100%;
  padding: 7px 10px;
  background: none;
  border: none;
  border-radius: 6px;
  color: var(--text-2);
  font-size: 13px;
  font-family: 'Inter', system-ui, sans-serif;
  cursor: pointer;
  text-align: left;
  transition: color .15s, background .15s;
}

.charcopy-nav-item:hover {
  color: var(--text);
  background: var(--bg-deep);
}

.charcopy-nav-item.active {
  background: var(--blue);
  color: #fff;
}

.charcopy-nav-item .icon {
  color: inherit;
  flex-shrink: 0;
}

.charcopy-panel {
  flex: 1;
  min-width: 0;
}

.charcopy-panel h2 {
  margin: 0 0 18px;
  font-size: 15px;
  font-weight: 600;
  color: var(--text);
  display: flex;
  align-items: center;
  gap: 8px;
}

.field { margin-bottom: 20px; }

.page-charcopy .row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  margin-bottom: 8px;
}

.page-charcopy .row label {
  display: inline;
  margin-bottom: 0;
}

.charcopy-count {
  font-size: 12px;
  font-family: 'Inter', system-ui, sans-serif;
  color: var(--accent);
  flex-shrink: 0;
}

.page-charcopy .actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 5px;
}

@media (max-width: 1000px) {
  .charcopy-layout { flex-direction: column; }
  .charcopy-sidebar { width: 100%; position: static; padding-right: 0; padding-bottom: 16px; }
  .charcopy-nav { flex-direction: row; flex-wrap: wrap; }
}
```

- [ ] **Step 2: Verify CSS applies cleanly**

Open `charactercopychecker.html` in a browser. The page will look broken (JS not yet updated) but confirm no CSS parse errors in DevTools console. No red errors should appear.

- [ ] **Step 3: Commit**

```bash
git add styles.css
git commit -m "refactor(copy-checker): replace grid CSS with sidebar layout styles"
```

---

### Task 2: HTML — Rewrite main structure

**Files:**
- Modify: `charactercopychecker.html` — replace `<main>` content (lines 39–130)

- [ ] **Step 1: Replace the `<main>` block**

Replace everything from `<main class="wrap">` through `</main>` with:

```html
<main class="wrap">
  <h1>Character-Safe Copy Checker</h1>
  <p>Paste copy, see live counts, and trim to common platform limits.</p>

  <div class="charcopy-layout">
    <aside class="charcopy-sidebar">
      <div class="charcopy-sidebar-label">Platforms</div>
      <div class="toolbar">
        <button id="clearAllBtn">Clear all</button>
        <button id="copyAllBtn">Copy all</button>
      </div>
      <nav class="charcopy-nav">
        <button class="charcopy-nav-item active" data-group="Google Search">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M21.35 11.1H12v2.8h5.35c-.8 2.3-3 3.9-5.35 3.9a6 6 0 1 1 0-12c1.6 0 3 .6 4.1 1.6l2-2A8.7 8.7 0 1 0 12 20.7c4.9 0 8.2-3.4 8.2-8.2 0-.6-.05-1-.15-1.4z" />
          </svg>
          Google
        </button>
        <button class="charcopy-nav-item" data-group="Meta">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 2C6.5 2 2 6.4 2 11.8c0 3.1 1.5 5.8 3.8 7.6V22l3.3-1.8c.9.2 1.9.4 2.9.4 5.5 0 10-4.4 10-8.8S17.5 2 12 2z" />
          </svg>
          Meta
        </button>
        <button class="charcopy-nav-item" data-group="LinkedIn">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M4.98 3.5C4.98 4.9 3.9 6 2.5 6S0 4.9 0 3.5 1.1 1 2.5 1s2.48 1.1 2.48 2.5zM.5 8.5h4v14h-4v-14zM8.5 8.5h3.8v1.9h.1c.5-.9 1.8-1.9 3.7-1.9 4 0 4.8 2.6 4.8 6v8h-4v-7.1c0-1.7 0-3.8-2.3-3.8s-2.6 1.8-2.6 3.7v7.2h-4v-14z" />
          </svg>
          LinkedIn
        </button>
        <button class="charcopy-nav-item" data-group="X">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M18.3 2H21l-6.6 7.6L22 22h-6.2l-4.9-6.4L5.5 22H2.8l7.1-8.1L2 2h6.3l4.4 5.8L18.3 2z" />
          </svg>
          X
        </button>
        <button class="charcopy-nav-item" data-group="Email">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M2 4h20v16H2V4zm10 7L4 6v12h16V6l-8 5z" />
          </svg>
          Email
        </button>
        <button class="charcopy-nav-item" data-group="YouTube">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M23 7.5s-.2-1.7-.8-2.5c-.7-.9-1.6-.9-2-.9C17.4 4 12 4 12 4s-5.4 0-8.2.1c-.4 0-1.3 0-2 .9C1.2 5.8 1 7.5 1 7.5S.8 9.5.8 11.4v1.2C.8 14.5 1 16.5 1 16.5s.2 1.7.8 2.5c.7.9 1.7.9 2.1 1C6.6 20 12 20 12 20s5.4 0 8.2-.1c.4 0 1.3 0 2-.9.6-.8.8-2.5.8-2.5s.2-2 .2-3.9v-1.2C23.2 9.5 23 7.5 23 7.5zM9.8 15.3V8.7l6.4 3.3-6.4 3.3z" />
          </svg>
          YouTube
        </button>
        <button class="charcopy-nav-item" data-group="TikTok">
          <svg class="icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M16.5 1c.6 2.8 2.3 4.5 5 5v3.2c-1.9 0-3.6-.6-5-1.6v6.9c0 4-3.3 7.5-7.5 7.5S1.5 18.6 1.5 14.6c0-3.7 2.9-6.8 6.7-7.4v3.5c-.9.3-1.5 1.1-1.5 2 0 1.2 1 2.2 2.3 2.2s2.5-1 2.5-2.6V1h4.1z" />
          </svg>
          TikTok
        </button>
      </nav>
    </aside>
    <div class="charcopy-panel" id="charcopyPanel"></div>
  </div>
  <div class="note">
    Limits vary by placement and device. Treat these as guardrails.
  </div>
</main>
```

- [ ] **Step 2: Verify HTML structure in browser**

Open `charactercopychecker.html`. You should see:
- Sidebar on the left with platform buttons
- Empty main panel on the right (JS not yet updated — no fields rendered yet)
- No JS console errors about missing elements

- [ ] **Step 3: Commit**

```bash
git add charactercopychecker.html
git commit -m "refactor(copy-checker): rewrite HTML with sidebar + panel structure"
```

---

### Task 3: JS — Dynamic platform rendering

**Files:**
- Modify: `charactercopychecker.html` — replace the `<script>` block (lines 132–347)

- [ ] **Step 1: Add `type` field to all LIMITS entries and add PLATFORM_ICONS**

The new JS needs to know which fields are `input` vs `textarea` without static HTML. Add a `type` property to each LIMITS entry, and add a `PLATFORM_ICONS` object. Replace the entire `<script>` block with the following. Start with the constants:

```javascript
const LIMITS = {
  google_headline:  { label: "Headline",        limit: 30,   group: "Google Search", type: "input"    },
  google_description:{ label: "Description",    limit: 90,   group: "Google Search", type: "textarea" },
  meta_primary:     { label: "Primary text",    limit: 125,  group: "Meta",          type: "textarea" },
  meta_headline:    { label: "Headline",         limit: 40,   group: "Meta",          type: "input"    },
  meta_linkdesc:    { label: "Link description", limit: 30,   group: "Meta",          type: "input"    },
  li_intro:         { label: "Intro text",       limit: 150,  group: "LinkedIn",      type: "textarea" },
  li_headline:      { label: "Headline",         limit: 70,   group: "LinkedIn",      type: "input"    },
  li_desc:          { label: "Description",      limit: 150,  group: "LinkedIn",      type: "textarea" },
  x_post:           { label: "Post text",        limit: 280,  group: "X",             type: "textarea" },
  email_subject:    { label: "Subject line",     limit: 60,   group: "Email",         type: "input"    },
  email_preheader:  { label: "Preheader",        limit: 90,   group: "Email",         type: "input"    },
  yt_title:         { label: "Title",            limit: 100,  group: "YouTube",       type: "input"    },
  yt_description:   { label: "Description",      limit: 5000, group: "YouTube",       type: "textarea" },
  tt_caption:       { label: "Caption",          limit: 2200, group: "TikTok",        type: "textarea" }
};

const PLATFORM_ICONS = {
  "Google Search": `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M21.35 11.1H12v2.8h5.35c-.8 2.3-3 3.9-5.35 3.9a6 6 0 1 1 0-12c1.6 0 3 .6 4.1 1.6l2-2A8.7 8.7 0 1 0 12 20.7c4.9 0 8.2-3.4 8.2-8.2 0-.6-.05-1-.15-1.4z" /></svg>`,
  "Meta":          `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.5 2 2 6.4 2 11.8c0 3.1 1.5 5.8 3.8 7.6V22l3.3-1.8c.9.2 1.9.4 2.9.4 5.5 0 10-4.4 10-8.8S17.5 2 12 2z" /></svg>`,
  "LinkedIn":      `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M4.98 3.5C4.98 4.9 3.9 6 2.5 6S0 4.9 0 3.5 1.1 1 2.5 1s2.48 1.1 2.48 2.5zM.5 8.5h4v14h-4v-14zM8.5 8.5h3.8v1.9h.1c.5-.9 1.8-1.9 3.7-1.9 4 0 4.8 2.6 4.8 6v8h-4v-7.1c0-1.7 0-3.8-2.3-3.8s-2.6 1.8-2.6 3.7v7.2h-4v-14z" /></svg>`,
  "X":             `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M18.3 2H21l-6.6 7.6L22 22h-6.2l-4.9-6.4L5.5 22H2.8l7.1-8.1L2 2h6.3l4.4 5.8L18.3 2z" /></svg>`,
  "Email":         `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M2 4h20v16H2V4zm10 7L4 6v12h16V6l-8 5z" /></svg>`,
  "YouTube":       `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M23 7.5s-.2-1.7-.8-2.5c-.7-.9-1.6-.9-2-.9C17.4 4 12 4 12 4s-5.4 0-8.2.1c-.4 0-1.3 0-2 .9C1.2 5.8 1 7.5 1 7.5S.8 9.5.8 11.4v1.2C.8 14.5 1 16.5 1 16.5s.2 1.7.8 2.5c.7.9 1.7.9 2.1 1C6.6 20 12 20 12 20s5.4 0 8.2-.1c.4 0 1.3 0 2-.9.6-.8.8-2.5.8-2.5s.2-2 .2-3.9v-1.2C23.2 9.5 23 7.5 23 7.5zM9.8 15.3V8.7l6.4 3.3-6.4 3.3z" /></svg>`,
  "TikTok":        `<svg class="icon" viewBox="0 0 24 24" fill="currentColor"><path d="M16.5 1c.6 2.8 2.3 4.5 5 5v3.2c-1.9 0-3.6-.6-5-1.6v6.9c0 4-3.3 7.5-7.5 7.5S1.5 18.6 1.5 14.6c0-3.7 2.9-6.8 6.7-7.4v3.5c-.9.3-1.5 1.1-1.5 2 0 1.2 1 2.2 2.3 2.2s2.5-1 2.5-2.6V1h4.1z" /></svg>`
};

const state = {};
```

- [ ] **Step 2: Add helper functions (unchanged from original)**

```javascript
function $(sel, root = document) {
  return root.querySelector(sel);
}

function $all(sel, root = document) {
  return Array.from(root.querySelectorAll(sel));
}

function countChars(str) {
  return (str || "").length;
}

async function copyText(text) {
  try {
    await navigator.clipboard.writeText(text);
    return true;
  } catch {
    return false;
  }
}
```

- [ ] **Step 3: Add updated `renderField` (removes Trim, removes status badge, adds color-coded count)**

```javascript
function renderField(container, key, type) {
  const conf = LIMITS[key];
  const current = state[key] || "";
  container.innerHTML = `
    <div class="row">
      <label>${conf.label}</label>
      <span class="charcopy-count" data-count>0 / ${conf.limit}</span>
    </div>
    ${type === "input"
      ? `<input type="text" data-input placeholder="Type or paste copy…" />`
      : `<textarea data-input placeholder="Type or paste copy…"></textarea>`
    }
    <div class="actions">
      <button class="miniBtn" type="button" data-copy>Copy</button>
    </div>
  `;
  const input = container.querySelector('[data-input]');
  const countEl = container.querySelector('[data-count]');
  input.value = current;

  function update() {
    const v = input.value || "";
    state[key] = v;
    const count = countChars(v);
    countEl.textContent = `${count} / ${conf.limit}`;
    countEl.style.color = count > conf.limit ? 'var(--red)' : 'var(--accent)';
  }
  input.addEventListener("input", update);
  container.querySelector('[data-copy]').addEventListener("click", async () => {
    const ok = await copyText(input.value || "");
    if (!ok) return;
    const btn = container.querySelector('[data-copy]');
    const prev = btn.textContent;
    btn.textContent = "Copied";
    setTimeout(() => (btn.textContent = prev), 900);
  });
  update();
}
```

- [ ] **Step 4: Add `renderPlatform` and sidebar nav wiring**

```javascript
let activePlatform = "Google Search";
const panel = document.getElementById("charcopyPanel");

function renderPlatform(group) {
  activePlatform = group;
  const keys = Object.keys(LIMITS).filter(k => LIMITS[k].group === group);
  panel.innerHTML = `<h2>${PLATFORM_ICONS[group] || ""}${group}</h2>`;
  keys.forEach(key => {
    const div = document.createElement("div");
    div.className = "field";
    panel.appendChild(div);
    renderField(div, key, LIMITS[key].type);
  });
}

$all(".charcopy-nav-item").forEach(btn => {
  btn.addEventListener("click", () => {
    $all(".charcopy-nav-item").forEach(b => b.classList.remove("active"));
    btn.classList.add("active");
    renderPlatform(btn.getAttribute("data-group"));
  });
});
```

- [ ] **Step 5: Add `buildCopyAllTextWithHeaders` and toolbar buttons (unchanged logic)**

```javascript
function buildCopyAllTextWithHeaders() {
  const groups = [...new Set(Object.values(LIMITS).map(c => c.group))];
  const lines = [];
  groups.forEach(group => {
    const keys = Object.keys(LIMITS).filter(k => LIMITS[k].group === group);
    const filled = keys
      .map(k => ({ k, v: (state[k] || "").trim() }))
      .filter(x => x.v.length);
    if (!filled.length) return;
    lines.push(group);
    lines.push("-".repeat(group.length));
    filled.forEach(({ k, v }) => {
      lines.push(`${LIMITS[k].label}: ${v}`);
    });
    lines.push("");
  });
  return lines.join("\n").trim();
}

$("#clearAllBtn").addEventListener("click", () => {
  for (const key of Object.keys(LIMITS)) state[key] = "";
  renderPlatform(activePlatform);
});

$("#copyAllBtn").addEventListener("click", async () => {
  const text = buildCopyAllTextWithHeaders();
  const ok = await copyText(text);
  const btn = $("#copyAllBtn");
  const prev = btn.textContent;
  btn.textContent = ok ? "Copied" : "Copy failed";
  setTimeout(() => (btn.textContent = prev), 1000);
});

renderPlatform("Google Search");
```

- [ ] **Step 6: Verify full functionality in browser**

Open `charactercopychecker.html` and check:
1. Google fields appear on load in the main panel
2. Clicking Meta in the sidebar switches to Meta fields; Google data is not lost (switch back and check)
3. Typing in a field updates the count (e.g., type 35 chars in Google Headline — count turns red)
4. Copy button copies the field value; button shows "Copied" briefly
5. Clear all wipes all fields in the current panel
6. Copy all (with headers) copies all filled fields across all platforms
7. At narrow viewport (< 1000px), sidebar stacks above panel

- [ ] **Step 7: Commit**

```bash
git add charactercopychecker.html
git commit -m "refactor(copy-checker): dynamic platform rendering with sidebar nav"
```
