# Field Sampler

A single-file, browser-based tool for planning and collecting soil samples in the field. No server, no build step, no install: everything runs client-side in `index.html`.

## Features

- **Sampling plans:** load shapefile (.zip), CSV, KML/KMZ or GeoJSON; points become check-off targets and polygons become zones with subsample pins.
- **Plan it yourself:** grid overlay, bulk composite zones, generated sample points.
- **Manual pins:** the yellow **+** button on the map drops pins by tap/click, right-click or long-press, or at your current GPS location. Point IDs auto-increment.
- **Edit mode:** per-layer **Edit** button. Drag pins (with confirmation), tap a pin to change its number or coordinates, add pins, renumber.
- **Manual zones:** draw zone polygons, split with a drawn line, merge zones, rename/renumber (text or numbers), drop subsample pins, undo.
- **Colours:** per-layer colour options, plus boundary and elevation colours.
- **Elevation:** NRCan CDEM, Copernicus GLO-30 (STAC) or an uploaded GeoTIFF; download the elevation as a GeoTIFF.
- **Boundary:** draw, import or record by walking; export as Shapefile or GeoJSON.
- **Exports:** bulk GeoJSON and PDF report for the whole project, plus per-layer exports (Shapefile, GeoJSON, CSV, KML, GPX) for points, zones and subsample pins.
- **Projects:** save/open `.fsproj` files. Works offline once loaded (basemap tiles and online elevation need a connection).

## Notes

- The app code in `index.html` is a minified bundle. Edit it with care, or keep the original source project and rebuild.
- Loaded elevation data is not stored in `.fsproj` project files; reload it after opening a project.
