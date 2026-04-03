# Palette Generator — Design Spec

## Overview

A new tool for the Marketing Tools suite that generates color harmony palettes from a base hex color. Users paste or pick a color, and the page displays six palette types with copyable swatch values in hex, RGB, or HSL format.

**File:** `palettegenerator.html`
**Body class:** `page-palette`

## Input

An inline row (no card wrapper) containing:

- **Color picker** (`<input type="color">`) — native browser picker
- **Hex text input** — accepts 3- or 6-digit hex with or without `#`
- **Format toggle** — three small buttons: Hex (default), RGB, HSL. Active button uses the site's `--blue` accent. Selection controls all swatch labels globally.

The picker and text input sync bidirectionally (same pattern as `colorcontrast.html`). The input auto-normalizes shorthand hex (e.g., `#abc` -> `#aabbcc`).

## Palette Sections

Six full-width sections stacked vertically below the input row. Each section has:

- A heading with the palette name
- A "Copy all" button (right-aligned in the heading row) that copies all values in the active format as a comma-separated list
- A horizontal row of swatches (wraps on mobile)

### Palettes

All math is done in HSL space. Convert input hex to HSL, apply hue rotations, convert back.

| Palette | Swatches | Logic |
|---------|----------|-------|
| Monochromatic | 5 | Same hue/saturation, lightness at 15%, 30%, 50%, 70%, 85% |
| Complementary | 2 | Base, +180deg |
| Analogous | 5 | Base, +/-30deg, +/-60deg |
| Triadic | 3 | Base, +120deg, +240deg |
| Split-Complementary | 3 | Base, +150deg, +210deg |
| Tetradic | 4 | Base, +60deg, +180deg, +240deg |

## Swatches

Each swatch is a clickable element:

- **Color rectangle** — filled with the generated color, rounded corners
- **Value label** — displayed below in the active format (hex/RGB/HSL), monospace font
- **Base indicator** — the base color swatch in each palette has a `--accent` border to distinguish it from generated colors
- **Click to copy** — copies the swatch value to clipboard. Brief "Copied!" flash feedback replacing the value text, then reverts after ~1.2s (same pattern as UTM page's copy button).

## Color Math

All conversions are inline JS, no external library.

### Hex -> RGB
Parse 3- or 6-digit hex string to `{r, g, b}` integers (0-255). Reuse the `parseHex` pattern from `colorcontrast.html`.

### RGB -> HSL
Standard conversion: normalize RGB to 0-1, compute max/min channel, derive hue (0-360), saturation (0-1), lightness (0-1).

### HSL -> RGB
Standard reverse conversion for rendering back to hex.

### Format output
- **Hex:** `#4b73ff`
- **RGB:** `rgb(75, 115, 255)`
- **HSL:** `hsl(227, 100%, 65%)`

## Integration

- **Nav:** Add "Palette Generator" link to the `.navLinks` in all pages' `<nav>` element
- **Homepage:** Add a card to the `.homeGrid` in `index.html` with icon `fa-solid fa-palette`, title "Palette Generator", description "Generate color harmonies from any base color."
- **README:** Add row to the tools table: "Palette Generator — Generate complementary, analogous, triadic, and other color harmonies from any base hex"

## Styles

Uses shared `styles.css`. Page-specific styles go in an inline `<style>` block in the `<head>` (consistent with other tools). Key custom styles needed:

- `.paletteRow` — flex row for swatches, `gap: 10px`, `flex-wrap: wrap`
- `.swatch` — the clickable swatch element (rectangle + label)
- `.swatch.base` — accent border for the base color
- `.formatToggle` — the Hex/RGB/HSL button group
- `.inputRow` — the inline row for picker + hex + toggle

## Responsive

Swatch rows wrap naturally with flexbox. On narrow viewports, swatches stack into a grid-like pattern. The inline input row wraps as needed.

## No External Dependencies

All color math is vanilla JS. No CDN scripts needed.
