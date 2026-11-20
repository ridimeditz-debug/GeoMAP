# GeoMap

Interactive map-work practice for **CBSE Class 12 Geography** (Fundamentals of Human Geography and India: People and Economy). Pure HTML, CSS and vanilla JavaScript with Leaflet. No backend, no login, no paid APIs. Progress is saved in the browser's `localStorage`.

## Run (fully offline)

Just **double-click `index.html`**. No server, no internet and no install are needed: Leaflet, all map data and all code are bundled (`data/bundle.js` and `js/app.bundle.js`), and there are no map tiles or CDN links.

If you edit anything in `js/` or the data, rebuild the bundle (plain Node, no downloads):

```bash
node tools/build-data.mjs      # only if you changed the datasets
node tools/build-offline.mjs
```

(`node tools/serve.mjs 8765` still works if you prefer a local server.)

## Deploy to GitHub Pages

1. Push this folder to a GitHub repository (the folder contents should be at the repository root, or in `/docs`).
2. Repository **Settings → Pages → Build and deployment**: Source = *Deploy from a branch*, choose your branch and `/ (root)` (or `/docs`).
3. Open `https://<user>.github.io/<repo>/`.

All asset and data paths are relative (data is resolved with `import.meta.url`, routing uses `#/hash` URLs), so the site works from a sub-path with no configuration. Leaflet is bundled in `vendor/`, and there are no map tiles, so nothing is fetched from third parties. `.nojekyll` is included.

## Features

| Area | What you get |
|---|---|
| Dashboard | Totals, topics completed, accuracy, attempts, best score, recent quizzes, quick access |
| Chapters | Book → chapter → category; only chapters with map work are active |
| World Atlas / India Atlas | Zoom and pan, category layers, distinct marker shapes, collision-aware labels, search, state/country name toggle, popups with verification status |
| Practice modes | Locate, Identify, MCQ, Mixed, Chapter-wise, Revision |
| Quiz | Question counter, progress bar, running score, timer, 10/20/30/full length, results screen, retry, answer review, skip |
| Progress | Per-category Not started / In progress / Completed tracker, revision queue and history, attempt history, reset |
| Syllabus page | Every entry with verification status, plus the original supplied syllabus preserved verbatim |
| Themes | Light and dark, responsive down to phone width |

Keyboard: `1`–`4` pick an option, `Enter` submits / goes to next.

## Locate scoring (tolerance system)

Each location has its own `fullKm` and `partialKm` in the data files.

* Distance ≤ `fullKm` → **1 mark**; ≤ `partialKm` → **0.5**; otherwise **0**.
* Distances are **great-circle (Haversine)** kilometres, never pixels.
* Region, state and line features are measured to the nearest point of the shape; a click *inside* a region or state is distance 0.
* **Zoom allowance:** 6 screen pixels are converted to km at the click's latitude and zoom (Web Mercator), capped at half of `fullKm`, and subtracted from the distance. A zoomed-out click is therefore not penalised for pixel-level imprecision.
* After submitting, the correct location, the full/half-mark circles (drawn in true metres, so projection is handled by Leaflet), the student's pin and a dashed line between them are shown, with a plain-language explanation of the marks.

Defaults: cities, ports, airports 50 / 100 km; Indian mines, refineries, ports, airports 40 / 100 km; canals and waterways 40–50 / 120–130 km; Indian states 25 / 100 km beyond the boundary; world regions 50 / 300 km beyond the boundary. Edit them in `tools/build-data.mjs` (then rebuild) or directly in `data/*.json`.

## Geographical accuracy and verification

Every entry has a status: `verified`, `needs_verification` or `excluded`. Only `verified` entries are used as quiz questions or answer options. Others are still drawn on the atlas with a visible **Verify** label and listed on the Syllabus page.

Flagged entries (kept exactly as supplied, not silently corrected):

| Entry | Status | Why |
|---|---|---|
| Cologne (South American sea ports) | excluded | Cologne is in Germany. The intended port may be Colón (Panama). Plotted at Cologne as supplied. |
| Wellington (Australia airports) | needs verification | Wellington is in New Zealand (Oceania). |
| St. Petersburg (Trans-Siberian terminus) | needs verification | Sources differ (St. Petersburg vs Moscow). |
| Bilaspur (bauxite) | needs verification | Several districts share the name. Plotted at Bilaspur, Chhattisgarh. |

Other notes: Neyveli is a lignite field; North Cape is plotted at Nordkapp; city and port entries use city-centre coordinates; the Ch. 4 regions are **generalised polygons** (schematic zones like the NCERT maps, not exact limits); crop "leading states" are the commonly cited top states and vary by year; population density uses the 2011 Census (Bihar highest and Arunachal Pradesh lowest among States, Delhi highest among Union Territories).

**Before using this for students, please confirm against the current NCERT textbooks and CBSE material:** the build could not cross-check them, so the four flags above, the crop state lists, the region polygons and the exact terminal stations are the first things to review. Change a status to `verified` in `tools/build-data.mjs` and rebuild to bring an entry into quizzes.

### Boundary data

`data/world-countries.geojson` and `data/india-states.geojson` are simplified open datasets (Natural Earth-derived world boundaries; a DataMeet-derived India state file). They are **not** an official Survey of India map and are for illustration. Where the external boundary depiction matters (board-exam material), replace `india-states.geojson` with the official Government of India outline: keep the same property names (`name`, `type` = `State` / `Union Territory`) and re-run the self-test.

## Project layout

```
index.html
css/        themes.css (tokens), style.css, responsive.css
js/         app.js (router/theme) · data.js · geo.js · scoring.js · quiz.js (question logic)
            quizui.js · map.js · atlas.js · dashboard.js · pages.js · progress.js · storage.js · util.js
data/       world-locations.json · india-locations.json · world-regions.json · world-routes.json
            questions.json · syllabus.json · syllabus-original.json · *.geojson (base maps)
vendor/     Leaflet 1.9.4
tools/      build-data.mjs (generates /data) · selftest*.mjs · serve.mjs · raw boundary files
```

Geographical data is separate from application logic. To change content, edit `tools/build-data.mjs` and run:

```bash
node tools/build-data.mjs
node tools/selftest.mjs && node tools/selftest-progress.mjs
```

## Testing

`tools/selftest.mjs` checks the Haversine distance, full/half/zero scoring for every point, that no unverified entry reaches a quiz, that every question has four unique options with one correct answer, and that mine/refinery coordinates fall in their stated states. `tools/selftest-progress.mjs` checks revision queueing and clearing, topic status and persistence. The UI was exercised in a browser (all quiz flows, results, persistence across reload, light/dark, phone width).
