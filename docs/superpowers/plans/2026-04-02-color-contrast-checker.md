# Color Contrast Checker — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a WCAG color contrast checker tool to the marketing-tools site.

**Architecture:** Single self-contained HTML file (`colorcontrast.html`) with inline JS, sharing the existing `styles.css`. Two-column grid layout (inputs left, results right) matching the SERP preview pattern. WCAG 2.1 relative luminance formula calculated in pure JS with real-time updates on input.

**Tech Stack:** Vanilla HTML/CSS/JS. No external dependencies.

---

### Task 1: Create the Color Contrast Checker page

**Files:**
- Create: `colorcontrast.html`

- [ ] **Step 1: Create `colorcontrast.html` with full markup and inline JS**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Color Contrast Checker</title>
  <meta name="description" content="Check WCAG color contrast ratios for text and background color combinations. Free browser-based accessibility tool." />
  <link rel="icon" href="toolbox-solid.svg" type="image/svg+xml" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link rel="stylesheet" href="styles.css" />
</head>
<body class="page-contrast">
  <nav class="siteNav">
    <div class="navInner">
      <a class="navBrand" href="index.html">Marketing Tools</a>
      <button class="navToggle" type="button" aria-expanded="false" aria-controls="navLinks">Menu</button>
      <div class="navLinks" id="navLinks">
        <a href="utmlinkgenerator.html">UTM Generator</a>
        <a href="qrcodegenerator.html">QR Code</a>
        <a href="charactercopychecker.html">Copy Checker</a>
        <a href="wordcharcounter.html">Word Counter</a>
        <a href="aptitles.html">AP Title Caps</a>
        <a href="symbollibrary.html">Symbol Library</a>
        <a href="serppreview.html">SERP Preview</a>
        <a href="colorcontrast.html" aria-current="page">Contrast Checker</a>
      </div>
    </div>
  </nav>

  <main class="wrap">
    <h1>Color Contrast Checker</h1>
    <p>Check WCAG 2.1 contrast ratios for any text and background color combination.</p>

    <div class="grid">
      <!-- Left column: inputs -->
      <div class="card">
        <h2>Colors</h2>

        <div class="colorField">
          <label for="textColor">Text color</label>
          <div class="colorInput">
            <input id="textColorPicker" type="color" value="#000000" />
            <input id="textColor" type="text" value="#000000" placeholder="#000000" />
          </div>
        </div>

        <div class="colorField">
          <label for="bgColor">Background color</label>
          <div class="colorInput">
            <input id="bgColorPicker" type="color" value="#FFFFFF" />
            <input id="bgColor" type="text" value="#FFFFFF" placeholder="#FFFFFF" />
          </div>
        </div>

        <div class="actions" style="margin-top: 12px;">
          <button type="button" id="swapBtn">Swap colors</button>
        </div>
      </div>

      <!-- Right column: results -->
      <div class="card">
        <h2>Results</h2>

        <div id="preview" class="contrastPreview">
          <p class="previewLarge">Large text sample (18px+)</p>
          <p class="previewNormal">Normal body text sample at standard size for readability testing.</p>
        </div>

        <div class="contrastRatio">
          <span class="contrastLabel">Contrast ratio</span>
          <span class="contrastValue" id="ratioDisplay">21:1</span>
        </div>

        <div class="contrastResults">
          <div class="contrastResult">
            <span>AA Normal</span>
            <span class="badge" id="aaNormal">Pass</span>
          </div>
          <div class="contrastResult">
            <span>AA Large</span>
            <span class="badge" id="aaLarge">Pass</span>
          </div>
          <div class="contrastResult">
            <span>AAA Normal</span>
            <span class="badge" id="aaaNormal">Pass</span>
          </div>
          <div class="contrastResult">
            <span>AAA Large</span>
            <span class="badge" id="aaaLarge">Pass</span>
          </div>
        </div>
      </div>
    </div>
  </main>

  <footer class="siteFooter"><small>Last Updated: 4/2/26</small></footer>
  <script src="script.js"></script>
  <script>
  (function () {
    const $ = id => document.getElementById(id);

    const textColor = $("textColor");
    const textColorPicker = $("textColorPicker");
    const bgColor = $("bgColor");
    const bgColorPicker = $("bgColorPicker");
    const preview = $("preview");
    const ratioDisplay = $("ratioDisplay");
    const swapBtn = $("swapBtn");

    function parseHex(hex) {
      hex = hex.trim().replace(/^#/, "");
      if (hex.length === 3) hex = hex[0]+hex[0]+hex[1]+hex[1]+hex[2]+hex[2];
      if (hex.length !== 6 || !/^[0-9a-fA-F]{6}$/.test(hex)) return null;
      return {
        r: parseInt(hex.slice(0,2), 16),
        g: parseInt(hex.slice(2,4), 16),
        b: parseInt(hex.slice(4,6), 16)
      };
    }

    function relativeLuminance(rgb) {
      const [rs, gs, bs] = [rgb.r, rgb.g, rgb.b].map(c => {
        c = c / 255;
        return c <= 0.04045 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
      });
      return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs;
    }

    function contrastRatio(lum1, lum2) {
      const lighter = Math.max(lum1, lum2);
      const darker = Math.min(lum1, lum2);
      return (lighter + 0.05) / (darker + 0.05);
    }

    function setBadge(id, pass) {
      const el = $(id);
      el.textContent = pass ? "Pass" : "Fail";
      el.className = "badge " + (pass ? "ok" : "over");
    }

    function update() {
      const fg = parseHex(textColor.value);
      const bg = parseHex(bgColor.value);

      if (!fg || !bg) {
        ratioDisplay.textContent = "—";
        ["aaNormal","aaLarge","aaaNormal","aaaLarge"].forEach(id => {
          $(id).textContent = "—";
          $(id).className = "badge";
        });
        return;
      }

      const fgHex = "#" + [fg.r,fg.g,fg.b].map(c => c.toString(16).padStart(2,"0")).join("");
      const bgHex = "#" + [bg.r,bg.g,bg.b].map(c => c.toString(16).padStart(2,"0")).join("");

      textColorPicker.value = fgHex;
      bgColorPicker.value = bgHex;

      preview.style.color = fgHex;
      preview.style.backgroundColor = bgHex;

      const ratio = contrastRatio(relativeLuminance(fg), relativeLuminance(bg));
      const rounded = Math.floor(ratio * 100) / 100;
      ratioDisplay.textContent = rounded.toFixed(2) + ":1";

      setBadge("aaNormal", ratio >= 4.5);
      setBadge("aaLarge", ratio >= 3);
      setBadge("aaaNormal", ratio >= 7);
      setBadge("aaaLarge", ratio >= 4.5);
    }

    textColor.addEventListener("input", update);
    bgColor.addEventListener("input", update);

    textColorPicker.addEventListener("input", () => {
      textColor.value = textColorPicker.value;
      update();
    });

    bgColorPicker.addEventListener("input", () => {
      bgColor.value = bgColorPicker.value;
      update();
    });

    swapBtn.addEventListener("click", () => {
      const tmp = textColor.value;
      textColor.value = bgColor.value;
      bgColor.value = tmp;
      update();
    });

    update();
  })();
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify the page loads and functions**

Run: Open `http://localhost:8090/colorcontrast.html` in the browser.
Expected: Two-column layout. Default black on white shows 21:1 ratio, all four badges show "Pass". Changing text color to `#777777` should drop ratio and flip some badges to "Fail".

- [ ] **Step 3: Commit**

```bash
git add colorcontrast.html
git commit -m "feat: add Color Contrast Checker tool"
```

---

### Task 2: Add page-specific CSS to styles.css

**Files:**
- Modify: `styles.css` (append at end, before the closing comment or EOF)

- [ ] **Step 1: Add contrast checker styles to styles.css**

Append the following to the end of `styles.css`:

```css
/* ─── Color Contrast Checker ─────────────────────────────── */

.page-contrast .wrap { max-width: 960px; }

.page-contrast .grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  align-items: start;
}

@media (max-width: 780px) {
  .page-contrast .grid { grid-template-columns: 1fr; }
}

.page-contrast .card h2 {
  margin: 0 0 14px;
  font-size: 14px;
  font-weight: 600;
  color: var(--text);
}

.colorField {
  margin-bottom: 14px;
}

.colorField label {
  display: block;
  margin-bottom: 7px;
}

.colorInput {
  display: flex;
  gap: 8px;
  align-items: center;
}

.colorInput input[type="color"] {
  width: 40px;
  height: 40px;
  padding: 2px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background: var(--bg-deep);
  cursor: pointer;
  flex-shrink: 0;
}

.colorInput input[type="text"] {
  flex: 1;
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
}

.contrastPreview {
  border-radius: var(--radius);
  padding: 20px;
  margin-bottom: 16px;
  border: 1px solid var(--border);
  transition: color 0.15s, background-color 0.15s;
}

.previewLarge {
  font-size: 20px;
  font-weight: 600;
  margin: 0 0 8px;
  line-height: 1.3;
}

.previewNormal {
  font-size: 14px;
  margin: 0;
  line-height: 1.5;
}

.contrastRatio {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 14px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background: var(--bg-deep);
  margin-bottom: 12px;
}

.contrastLabel {
  font-size: 12px;
  font-weight: 500;
  color: var(--text-2);
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

.contrastValue {
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
  font-size: 18px;
  font-weight: 600;
  color: var(--text);
}

.contrastResults {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.contrastResult {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 12px;
  border-radius: var(--radius);
  border: 1px solid var(--border);
  background: var(--bg-deep);
  font-size: 13px;
  color: var(--text-2);
}
```

- [ ] **Step 2: Verify styles render correctly**

Run: Refresh `http://localhost:8090/colorcontrast.html` in the browser.
Expected: Color pickers sit beside hex inputs, preview panel shows colored text, results grid shows 2x2 pass/fail badges.

- [ ] **Step 3: Commit**

```bash
git add styles.css
git commit -m "style: add contrast checker page styles"
```

---

### Task 3: Add Contrast Checker to navigation and homepage on all pages

**Files:**
- Modify: `index.html` (add nav link + homepage card)
- Modify: `utmlinkgenerator.html` (add nav link)
- Modify: `qrcodegenerator.html` (add nav link)
- Modify: `charactercopychecker.html` (add nav link)
- Modify: `wordcharcounter.html` (add nav link)
- Modify: `aptitles.html` (add nav link)
- Modify: `symbollibrary.html` (add nav link)
- Modify: `serppreview.html` (add nav link)

- [ ] **Step 1: Add nav link and homepage card to `index.html`**

In the `.navLinks` div, after the SERP Preview link, add:
```html
        <a href="colorcontrast.html">Contrast Checker</a>
```

In the `.homeGrid` div, after the SERP Preview card, add:
```html
      <a class="homeCard" href="colorcontrast.html">
        <div class="homeCardIcon">🎨</div>
        <h2 class="homeTitle">Color Contrast Checker</h2>
        <p class="homeDesc">Check WCAG contrast ratios for any text and background color pair.</p>
      </a>
```

- [ ] **Step 2: Add nav link to all other tool pages**

In each of the 7 tool HTML files, find the closing `</div>` of `.navLinks` and add the Contrast Checker link before it. The nav link to add (same in all files):
```html
        <a href="colorcontrast.html">Contrast Checker</a>
```

Note: The existing tool pages use emoji-prefixed nav labels (e.g., `🔗 UTM Link Generator`). For consistency within those pages, use:
```html
        <a href="colorcontrast.html">🎨 Contrast Checker</a>
```

The homepage (`index.html`) uses the no-emoji nav style — use the plain version there.

- [ ] **Step 3: Update homepage meta description**

In `index.html`, update the meta description to include the new tool:
```html
<meta name="description" content="Free browser-based marketing tools — UTM generator, QR codes, SERP preview, color contrast checker, and more. No sign-up required." />
```

- [ ] **Step 4: Verify navigation works site-wide**

Run: Navigate between pages in the browser. Confirm:
- Contrast Checker appears in nav on every page
- Contrast Checker card appears on homepage
- Link navigates to `colorcontrast.html`

- [ ] **Step 5: Commit**

```bash
git add index.html utmlinkgenerator.html qrcodegenerator.html charactercopychecker.html wordcharcounter.html aptitles.html symbollibrary.html serppreview.html
git commit -m "feat: add Contrast Checker to site navigation and homepage"
```

---

### Task 4: Run Lighthouse audit and fix any issues

**Files:**
- Possibly modify: `colorcontrast.html` (if audit finds issues)

- [ ] **Step 1: Run Lighthouse audit on the new page**

Navigate to `http://localhost:8090/colorcontrast.html` and run a Lighthouse accessibility audit.
Expected: 100 Accessibility, 100 Best Practices, 100 SEO.

- [ ] **Step 2: Fix any failing audits**

If any audits fail, fix the issues in `colorcontrast.html` or `styles.css`.

- [ ] **Step 3: Commit and push**

```bash
git add -A
git commit -m "chore: fix any Lighthouse issues on contrast checker"
git push
```
