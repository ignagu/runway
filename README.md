# Runway

A small, offline-first PWA for planning family travel against an annual budget.
Plain HTML/CSS/JS, no build step, no backend, no tracking. It runs as a static
site and installs to the iPad home screen.

Three tabs:

- **Dashboard** — a budget per year, what's allocated vs. free, whether it is
  pacing ahead or behind, and a month-by-month bar chart of spent vs. planned.
- **Calendar** — year, month and week views over the 2026 Spanish (Madrid) and
  Swedish holidays, long-weekend runs and single-day bridges (*puentes*), plus
  your own trips.
- **Trips** — trips tagged with one or more countries, and a budget breakdown
  whose categories you can extend.

## Never commit your data

**The repository contains code only.** It ships empty: no budget, no points, no
trips, no names, no figures.

Your data lives in two places, both of them yours:

1. `localStorage` in the browser on your device (the working copy).
2. A JSON file you export and keep in Files/iCloud (the durable copy).

Exported backups are personal. **Never commit one.** `.gitignore` already
excludes `runway-backup-*.json`, `/data/`, and `*.local.json`, and the app never
writes anything into the repository directory — exports go through the browser's
download flow. If `git status` ever shows a data file, delete it rather than
committing it.

## Running it

Any static file server works, from the repo root:

```sh
python3 -m http.server 8000
# then open http://localhost:8000/
```

Opening `index.html` directly via `file://` mostly works, but service workers
(and therefore offline mode) need `http://` or `https://`.

## Hosting on GitHub Pages

1. Push to a public repo with all files in the root.
2. **Settings → Pages → Deploy from branch → `main` / `/ (root)`**.
3. The app is live at `https://<user>.github.io/<repo>/`.

Every path in the app is relative, so the project subpath works without config.

## Installing on iPad

1. Open the Pages URL in Safari.
2. **Share → Add to Home Screen.**
3. Launch from the home screen: it opens standalone, without Safari chrome.

After the first load the service worker caches the app, so it works fully
offline. Google Fonts are the one optional network request; offline they fall
back to system fonts and the layout is unchanged.

## Backup and restore

- **Backup** hands you `runway-backup-YYYY-MM-DD.json`. How depends on the
  browser, because iOS and everything else disagree about downloads:
  - **iPhone/iPad** — the share sheet opens; choose **Save to Files** (iCloud
    Drive or On My iPad). iOS Safari ignores `<a download>` for generated
    files, and a home-screen app has no address bar to escape a blob URL, so
    the share sheet is the only route that reaches Files.
  - **Desktop and Android** — a normal download.
  - **Anything older that supports neither** — the JSON appears on screen with
    a Copy button, so a backup is always recoverable.
- **Restore** reads such a file back and replaces everything on the device
  (it asks first). On iPad, pick the file from Files.
- **Reset** clears the device back to empty.

Restore accepts older and partial files: missing categories are filled in,
numbers are coerced, and unknown statuses/types fall back to defaults.

### Linking a real file (desktop Chromium/Edge only)

Where the File System Access API exists, **Link data file** binds the app to a
real `.json` on disk and auto-saves to it on every change; **Open data file**
reloads from it. The file handle is remembered in IndexedDB between sessions.

iPad Safari has no such API, so those two buttons are hidden there and
Backup/Restore is the local-file workflow.

## The budget window

A budget belongs to a window ending 31 December. Set one part-way through the
current year and the window starts **today**, not 1 January — a budget entered in
August is a budget for August to December, and pacing it against a year that is
already two thirds gone would report you as hopelessly behind on day one.

The window is shown under the budget and can be changed to any date in that
year; set it to 1 January for a normal full-year budget. Everything follows from
it: what counts as allocated, free budget per remaining month, the percentage of
the period elapsed, and the even-pace line on the chart.

Trips starting before the window do not draw on it — they predate the budget.
They stay visible (faded in the chart, and their total called out under the
tiles) so nothing is quietly dropped. Budgets stored without a window keep the
full year, so existing figures do not shift.

## Categories

Six categories ship by default (Flights, Lodging, Food, Transport, Activities,
Other). You can add your own from the trip form. Names are compared with case,
accents, spacing and plurals ignored, so "lodgings" is recognised as the
existing "Lodging" rather than added beside it; a near-miss like "Lodgin" asks
before splitting your spend across two lines. A custom category with no money
against it anywhere can be removed.

## Holiday data

2026 only, hard-coded in `index.html`:

- **Spain** — the City of Madrid calendar: national + Comunidad de Madrid +
  Madrid local days.
- **Sweden** — *röda dagar* plus the observances people actually take off
  (Midsommarafton, Julafton, Nyårsafton). The window finder treats those as
  days off, which is how they work in practice.

Extending to 2027+ means computing the movable feasts (Easter and everything
hanging off it) and re-checking each national list — don't guess the dates.

## Files

```
index.html             the whole app (markup + CSS + JS)
sw.js                  service worker, cache-first offline
manifest.webmanifest   PWA manifest
icon-180.png           apple-touch-icon
icon-512.png           manifest icon
```
