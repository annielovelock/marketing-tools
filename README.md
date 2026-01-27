# Marketing Tools

A lightweight set of marketing utilities built as static HTML/CSS/JS and published as a GitHub Pages site.

## Live site (GitHub Pages)
After you enable Pages in your repo settings, the site will be available at:
`https://<your-username>.github.io/<your-repo>/`

Live site: `https://annielovelock.github.io/marketing-tools/`

## Included tools
- UTM Link Generator + QR
- QR Code Generator
- Character-Safe Copy Checker
- Word + Character Counter
- AP-Style Title Capitalization
- Symbol Library
- SERP Preview + Meta Length Checker

## Local development
These are static files, so you can open them directly in the browser:
- Open `index.html` to access the home page and navigation.

Optional: run a tiny local server if you prefer:
```sh
python3 -m http.server 8080
```
Then visit `http://localhost:8080/`.

## Deploy to GitHub Pages
1. Push this folder to a GitHub repository.
2. In GitHub: `Settings` → `Pages`.
3. Set **Source** to the `main` branch and `/ (root)` folder.
4. Save. GitHub will provide your Pages URL.

## Updating the "Last Updated" footer
Each page includes a small footer label you can edit:
```
Last Updated: 1/27/26
```
Update the date on any page you modify to indicate whether or not tools may be up to date with the latest best practices.

## Notes
- All pages share the same stylesheet in `styles.css`.
- Navigation and layout are duplicated across the HTML files for simplicity.
