# PowerLines PUC Dashboard — implementation

Production build of the Public Utilities Commission dashboard and the one-page
state factsheet. Plain HTML/CSS/JS — no build step, no framework. Both pages
read live from a shared Google Sheet on each load and fall back to a baked
snapshot (`data.js`) if the sheet can't be reached.

## Files

| File | What it is |
|------|------------|
| `index.html` | The dashboard: US choropleth + six data boxes. Embed this in WordPress. |
| `factsheet.html` | One-page, print-ready state factsheet. Reads `?state=CODE` (e.g. `?state=CA`). |
| `data.js` | Baked fallback snapshot of the workbook. Used only if the live sheet fetch fails. |
| `assets/` | PowerLines logos (dark + white). |
| `vendor/` | d3, topojson-client, the US state geometry, and the brand fonts (`fonts.css`), all served locally — no third-party CDN at runtime. |

The dashboard's **View factsheet** button links to `factsheet.html?state=CODE`,
so host the two pages alongside each other (see WordPress notes below). The
button is disabled for Nebraska (no rate-regulated IOUs) and until a state is
selected.

## Data source

Both pages point at the same sheet, set once at the top of each file:

```js
const CONFIG = { sheetId: '1l58RfG7rVHjYX96k3hX8FI7xSF9BB3MMhs9sTi9HO0A', tabs: {...} };
```

- The sheet must be shared **"Anyone with the link" ▸ Viewer** — each visitor's
  browser fetches it directly, so org-only access would block public visitors.
- Edits appear on the next page load; Google caches the CSV endpoint for ~5 min,
  so changes surface within a few minutes. No re-upload, no redeploy.
- Leave `sheetId` empty to run purely off the baked `data.js` snapshot.
- Tab **names** must match `CONFIG.tabs` exactly — renaming a tab breaks that
  section's live pull (it falls back to the snapshot).

### Tabs read

`States` · `Annual Rate Increase Requests` (incl. column **Q** blank-state
explainers) · `PUC Info` · `Commissioners` · `CapEx` (column **F** = `YES`
flags a multi-state utility → asterisk + footnote) · `Utility Bills` (annual
avg columns **E–K** drive the charts; the factsheet's headline "latest average
household bill" is read from column **N**, with its month/year label taken from
the header cell **N2**) · `Dockets` (column **P** = display priority, **O** = docket URL) ·
`CapEx Total By State` · `Factsheets`.

## Updating

Edit the Google Sheet — that's the whole update protocol. To refresh the
**offline** fallback, regenerate `data.js` from the current workbook.

## Embedding in WordPress

1. Create a Page and add a **Custom HTML** block; paste the contents of
   `index.html`. Publish.
2. Host `factsheet.html`, `data.js`, `assets/`, and `vendor/` at paths relative
   to that page so the dashboard's factsheet buttons resolve. If you host the
   factsheet at a different URL, update the `href` in `index.html`'s `box1()`
   (`factsheet.html?state=...`) to the absolute URL.

The layout is a full-width responsive embed (breakpoints at 820 / 640 / 480px),
so it fills whatever container width WordPress gives it.

## Local preview

```
cd src && python3 -m http.server 8099
# open http://localhost:8099/index.html
# open http://localhost:8099/factsheet.html?state=CA
```
