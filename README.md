# Marketing Tools

A free, open-source collection of browser-based utilities for digital marketers. No sign-ups, no tracking, no server required — everything runs locally in your browser.

**Live site:** [annielovelock.github.io/marketing-tools](https://annielovelock.github.io/marketing-tools/)

## Tools

| Tool | Description |
|------|-------------|
| **UTM Link Generator** | Build campaign tracking URLs with automatic normalization |
| **QR Code Generator** | Convert any URL or text into a high-res PNG QR code |
| **Character-Safe Copy Checker** | Real-time character counts against platform limits (Google meta titles/descriptions, LinkedIn, X, and more) |
| **Word + Character Counter** | Quick word and character counts for any text |
| **AP-Style Title Capitalization** | Auto-apply Associated Press headline capitalization rules |
| **SERP Preview** | See how your page title and meta description will render in Google search results |
| **Symbol Library** | Searchable collection of commonly used marketing symbols and special characters |
| **Color Contrast Checker** | Check WCAG contrast ratios for any text and background color pair |

## Getting Started

These are static files — no build step or dependencies to install.

**Option 1:** Open `index.html` directly in your browser.

**Option 2:** Run a local server:

```sh
python3 -m http.server 8080
```

Then visit `http://localhost:8080/`.

## Tech Stack

- Vanilla HTML, CSS, and JavaScript
- [QRious](https://github.com/neocotic/qrious) for QR code generation (loaded via CDN)
- Hosted on GitHub Pages

## Contributing

Contributions are welcome! Feel free to open an issue or submit a pull request.

## License

MIT
