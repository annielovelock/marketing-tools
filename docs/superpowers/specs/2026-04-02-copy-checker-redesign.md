# Copy Checker Redesign Spec

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild `charactercopychecker.html` with a sidebar-nav layout that is sleeker and less visually dense than the current 2-column card grid.

**Architecture:** Replace the current grid of platform cards with a two-panel layout: a narrow fixed sidebar listing platforms on the left, and a main content panel on the right showing the active platform's fields. All JS logic (`LIMITS`, `state`, `renderField`, `buildCopyAllTextWithHeaders`) is preserved unchanged; only HTML structure and CSS change.

**Tech Stack:** Vanilla HTML/CSS/JS, shared `styles.css`, Font Awesome 6.5.1, no build step.

---

## Layout

- Two-panel layout inside `.wrap`: left sidebar (~140px wide) + right main panel (flex: 1)
- Sidebar is sticky (stays in view while main panel scrolls)
- Page max-width: 1100px (unchanged, controlled by `.page-charcopy .wrap`)
- Sidebar contains:
  - Page title ("Copy Checker") in small uppercase label style
  - "Clear all" and "Copy all" as small secondary buttons (stacked, full sidebar width)
  - Platform nav list: 7 items (Google, Meta, LinkedIn, X, Email, YouTube, TikTok), each with its existing SVG icon + platform name
  - Active platform is highlighted (blue background pill, white text)
  - Inactive platforms: muted text (`--text-2`), hover lightens to `--text`
- Main panel contains:
  - Platform heading (h2 with icon, same as current card h2 style)
  - Fields for the active platform rendered by `renderField` (unchanged logic)
  - No card box/border around the panel — fields sit directly in the panel

## Field Design

Each field (rendered by `renderField`):
- Label row: label text on left (uppercase, `--text-3`, 11px) + character count (`0 / 30`) on right
- Count color: `--accent` (#8fb0ff) when within limit; `--red` (#ff7878) when over
- No separate status badge — color on the count is sufficient
- Input or textarea below, full width (unchanged sizing)
- "Copy" button below the input, right-aligned, quiet secondary style (`miniBtn`)
- No Trim button
- 20px gap between fields, no internal dividers

## Behavior

- On load: Google Search is the active platform
- Clicking a sidebar platform: updates active highlight, re-renders main panel with that platform's fields
- `state` object persists all platform values across platform switches (no data loss on switch)
- "Clear all": clears `state` for all keys, re-renders current panel
- "Copy all (with headers)": copies all filled fields across all platforms (unchanged `buildCopyAllTextWithHeaders` logic)
- Character count updates live on input (unchanged)
- "Copy" button per field: copies that field's value; shows "Copied" for 900ms (unchanged)

## CSS Changes (in `styles.css`)

All changes scoped to `.page-charcopy` to avoid affecting other pages.

- Remove `.page-charcopy .grid` (2-column grid) — replace with flex row layout for sidebar + panel
- Add `.charcopy-layout`: `display: flex; gap: 0; align-items: flex-start;`
- Add `.charcopy-sidebar`: `width: 140px; flex-shrink: 0; position: sticky; top: 20px; padding-right: 16px;`
- Add `.charcopy-panel`: `flex: 1; min-width: 0;`
- Add `.charcopy-nav-item`: platform nav button/link styles (active + hover states)
- Update `.page-charcopy .row`: keep label + count layout, remove status badge styles
- `.field` spacing: `margin-bottom: 20px` (unchanged)
- `.page-charcopy .actions`: right-align, margin-top: 4px
- Remove card box from main panel (no `background`, `border`, `border-radius` on the panel itself)

## HTML Structure

```html
<main class="wrap">
  <div class="charcopy-layout">
    <aside class="charcopy-sidebar">
      <div class="charcopy-sidebar-label">Copy Checker</div>
      <button id="clearAllBtn">Clear all</button>
      <button id="copyAllBtn">Copy all</button>
      <nav class="charcopy-nav">
        <!-- one button per platform, data-group attribute -->
        <button class="charcopy-nav-item active" data-group="Google Search">
          <svg class="icon">...</svg> Google
        </button>
        ...
      </nav>
    </aside>
    <div class="charcopy-panel" id="charcopyPanel">
      <!-- renderPlatform() writes fields here -->
    </div>
  </div>
  <div class="note">Limits vary by placement and device. Treat these as guardrails.</div>
</main>
```

## JS Changes

- Remove static `.field` divs from HTML — panel is rendered dynamically
- Add `activePlatform` variable (default: `"Google Search"`)
- Add `renderPlatform(group)` function: filters `LIMITS` by group, renders all fields for that group into `#charcopyPanel`
- Sidebar nav click handler: sets `activePlatform`, updates active class, calls `renderPlatform`
- `renderAll()` replaced by `renderPlatform(activePlatform)` on init
- `clearAllBtn`: clears all state keys, calls `renderPlatform(activePlatform)` (unchanged data, updated render target)
- All other JS logic (`renderField`, `buildCopyAllTextWithHeaders`, `copyText`, `statusBadge`, `countChars`, `trimToLimit`, `LIMITS`, `state`) unchanged

## Out of Scope

- No changes to other pages
- No changes to nav, footer, or OG meta tags
- No new platforms or limit values
- No mobile-specific sidebar collapse (existing single-column breakpoint at 1000px can stack sidebar above panel)
