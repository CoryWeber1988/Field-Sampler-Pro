# Field Sampler

A single-file, browser-based tool for planning and collecting soil samples in the field. No server, no build step, no install: everything runs client-side in `index.html`.

## Help

The full user guide, including a list of every online dataset and link the tool uses, is in **[HELP.md](HELP.md)**. The same content is built into the app: tap the **?** button in the header, or **Help and data sources** on the welcome page.

## Workflow

1. **Landing page:** basics of the tool, unit selection (metric / imperial) and an accuracy disclaimer. Declining shows a "Thank you for visiting" page. After agreeing, choose **Load an existing sample plan** (opens the **Collect** tab) or **Plan your own** (opens the **Plan** tab). The home button in the header returns to the landing page.
2. **Plan tab:** boundary, Sentinel-2 imagery search (calendar of dates; Aerial, NDVI, MSAVI, VARI, NDRE, NDYI, Bare Soil Index, your own formulas, or your own GeoTIFF), elevation (CDEM, HRDEM LiDAR, Google GEM-15, Copernicus or upload) with terrain layers (elevation, relative elevation, TPI, slope %, relative slope index), k-means zoning (1-10 zones) of any single layer, grid overlay with a live rotation preview, composites and sample points. Every layer and zone map can be downloaded (GeoTIFF, plus Shapefile / GeoJSON for zones).
3. **Send to Collection:** at the end of planning, choose which layers go to the Collect tab as background layers and which zone maps become zone sampling layers.
4. **Collect tab:** load plans, check off points, drop subsample pins, add notes, edit points.

## Features


- **Sampling plans:** load shapefile (.zip), CSV, KML/KMZ or GeoJSON; points become check-off targets and polygons become zones with subsample pins.
- **Plan it yourself:** grid overlay, bulk composite zones, generated sample points.
- **Manual pins:** the yellow **+** button on the map drops pins by tap/click, right-click or long-press, or at your current GPS location. Point IDs auto-increment. While planning, tap an existing pin and confirm to delete it, or use **Undo last**.
- **Edit mode:** per-layer **Edit** button, behind a caution prompt. Drag pins (with confirmation), snap a pin to your current GPS location, or tap a pin to change its number or coordinates. Every move adds an automatic note with the previous and new coordinates.
- **Manual zones:** draw zone polygons, split with a drawn line, merge zones, rename/renumber (text or numbers), drop subsample pins, undo.
- **Colours:** per-layer colour options, plus boundary and elevation colours.
- **Elevation:** NRCan CDEM, NRCan HRDEM LiDAR, Google GEM-15 (15 m bare-earth DTM, near-global), Copernicus GLO-30 (STAC) or an uploaded GeoTIFF; download the elevation as a GeoTIFF.
- **Boundary:** draw, import or record by walking; export as Shapefile or GeoJSON.
- **Exports:** bulk GeoJSON and PDF report for the whole project, plus per-layer exports (Shapefile, GeoJSON, CSV, KML, GPX) for points, zones and subsample pins.
- **Projects:** save/open `.fsproj` files. Works offline once loaded (basemap tiles and online elevation need a connection).

- **Basemap:** Satellite imagery is the default background. Choose **No tiles (offline)** for use with no signal.
- **Mode banner:** the orange bar at the top of the map shows the current action (drawing, recording, dropping pins) with its Finish / Cancel buttons; it wraps on small screens and stays clear of the map controls.

## Run locally

Open `index.html` in a browser. GPS features need a secure context, so for those serve it over localhost:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy to GitHub Pages

1. Create a repo and push these files to the `main` branch.
2. In the repo go to **Settings → Pages → Build and deployment → Source** and choose **GitHub Actions**.
3. The workflow in `.github/workflows/pages.yml` publishes the site on every push to `main`. Run it by hand from the **Actions** tab if needed.

The site will be at `https://<your-username>.github.io/<repo-name>/`. Pages is served over HTTPS, so GPS works there.

## Repo layout

| File | Purpose |
| --- | --- |
| `index.html` | The whole app (code, libraries and the in-app help, bundled in one file) |
| `HELP.md` | User guide and list of online data sources (the in-app help is generated from the same content) |
| `docs/add-gem15-elevation-prompt.md` | Reusable prompt for adding the Google GEM-15 elevation source to other single-file HTML tools |
| `.github/workflows/pages.yml` | GitHub Pages deployment |
| `.nojekyll` | Tells Pages to serve the file as-is |
| `.gitignore` | Ignores OS/editor/scratch files |

## Notes

- `index.html` is a bundle: the libraries are minified, while the app code (including the planner) is readable inside the main script.
- Imagery and terrain layers are saved in `.fsproj` files as images (viewable, not re-downloadable as GeoTIFF); rebuild them in the Plan tab to download.
- Loaded elevation data is not stored in `.fsproj` project files; reload it after opening a project.
- **Google GEM-15** is read from a public Google Cloud Storage bucket through the bucket's JSON API (`storage/v1/b/<bucket>/o/<object>?alt=media`), because plain `storage.googleapis.com/<bucket>/<object>` links send no CORS headers. See [HELP.md](HELP.md#online-data-sources-and-links).
- When you change the help text, update both `HELP.md` and the `helpPage` block in `index.html` (between the `HELP BEGIN` / `HELP END` comments) so they stay in sync.
