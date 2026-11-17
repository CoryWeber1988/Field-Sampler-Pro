# Field Sampler: Help and data sources

Field Sampler is a single-file, browser-based tool for planning and collecting soil samples. Everything runs on your device; nothing is uploaded. Only the satellite, elevation and basemap sources listed in [Online data sources](#sources) are contacted, and only when you ask for them.

## Quick start

1. Open the tool, choose **Metric** or **Imperial**, and tick the accuracy disclaimer. Then pick **Plan your own** (build a plan from boundary, imagery and terrain) or **Load an existing sample plan** (go straight to collecting). **Open a saved project** reopens a `.fsproj` file.
2. **Plan tab:** set the field boundary, search Sentinel-2 imagery, load an elevation source, calculate terrain layers, cluster a layer into zones, then build a grid, composite or sample points.
3. **Send to Collection** at the end of planning. Imagery and terrain become background layers; zone maps become zone sampling layers.
4. **Collect tab:** check off points as you sample, drop subsample pins, add notes and edit points. Export or save the project when you are done.
5. The home button (house icon) in the header returns to the welcome page. The **?** button opens this help.

## Plan tab, step by step

**Basemap.** Satellite imagery is the default. **Street** is also available. Choose **No tiles (offline)** when you have no signal; your plan and pins don't need a map.

**Field boundary.** Load a boundary file (Shapefile .zip, KML, KMZ or GeoJSON), **draw** it on the map, or **record** it by walking the perimeter. Once set, its area shows in the panel and it becomes the default boundary for the grid, composite and elevation tools. You can export it as GeoJSON or Shapefile and change its colour.

**Step 2: Imagery and aerial.** Needs a data connection.

- Pick a month and year. Dates with a usable Sentinel-2 scene that fully covers the field show on the calendar. Use the **Maximum cloud cover** slider (default 30 %) to filter, or tick **Also show dates over the cloud limit**.
- Select a date, tick the layers to add (**Aerial** true colour, NDVI, MSAVI, VARI, NDRE, NDYI, Bare Soil Index or your own saved indices), optionally keep **Mask clouds and shadows** on, then **Add selected as background layers**.
- **Add your own index:** give it a name and a formula using bands B02 (blue), B03 (green), B04 (red), B05 to B07 (red-edge), B08 (NIR), B8A (narrow NIR), B11 and B12 (SWIR). Functions: sqrt, abs, min, max, pow, log, exp.
- **Use your own imagery or raster (GeoTIFF):** add a single band as-is, or a normalised difference (A minus B) / (A plus B) of two bands.

**Elevation source (DEM).** Pull elevation for the field from a public DEM or upload your own GeoTIFF, then calculate terrain layers from it.

1. Set the field boundary first (or load points or zones, or zoom the map in to the field, about 20 km across or less).
2. Choose a **Source**, then **Search and load**. Only a window around your field is read, not the whole dataset. Or choose **Upload GeoTIFF** to use your own file.
3. Adjust the colour ramp, opacity, **Show on map** and **Clip to boundary**. The legend shows the 2nd to 98th percentile range, and the range line shows the full min, max and relief.
4. **Download GeoTIFF** saves the loaded elevation as a float32 GeoTIFF in WGS84 (EPSG:4326), optionally clipped to the boundary. **Clear elevation** removes it.

| Source | Coverage and resolution | Use it when |
| --- | --- | --- |
| **NRCan CDEM** | Canada, about 16 to 30 m | You are in Canada and want a quick, reliable base layer. |
| **NRCan HRDEM LiDAR** | Canada, 1 to 2 m, only where LiDAR projects have been released | It covers your field. This is the most detailed bare-earth (DTM) option. |
| **Google GEM-15 DTM** | Near-global land, 15 m bare earth | HRDEM doesn't cover your field, or you are outside Canada and want something finer than 30 m. |
| **Copernicus GLO-30** | Global, 30 m | Nothing finer is available. It is a surface model, so trees and buildings are included. |
| **Upload GeoTIFF** | Your file | You have your own survey or LiDAR-derived DEM. |

> **Vertical datums differ between sources**, so absolute heights from two sources can differ by a metre or more. The terrain layers (relative elevation, TPI, slope, relative slope index) are not affected.

**About GEM-15.** GEM-15 is an experimental Google product: global surface (DSM) and terrain (DTM) models at 15 m, made from sub-metre optical stereo imagery. Field Sampler uses the **DTM** (bare earth). Good to know:

- Heights are above the EGM96 geoid, in metres. Water, snow and ice, cloud shadow and low-texture ground such as salt pans have no data. By default, gaps inside your boundary are filled when you calculate terrain layers (see the heal setting below).
- Bare-earth accuracy is lower under dense forest. It is a 15 m model: it will not show small drainage features that HRDEM LiDAR does.
- Coverage is land between about 60°S and 84°N and some regions are left out (see the [GEM-15 README](https://github.com/google-research/google-research/tree/master/gem_15)). If your field has no tile you get a message suggesting another source.
- The first load asks Google's catalogue which 1° cell covers your field, then reads only the part of the file you need. A field that spans two cells reads both.
- Please credit the dataset as **GEM-15, Google** if you publish maps made from it, and check the licence on the [Earth Engine catalog page](https://developers.google.com/earth-engine/datasets/catalog/projects_climate-and-sustainability_assets_gem-15_v1).

**Step 3: Terrain layers.** Needs an elevation source loaded. Tick any of: **Elevation**, **Relative elevation** (0 to 1 in a local window), **Topographic position index (TPI)**, **Slope (%)**, **Relative slope index (0 to 1)**, then **Calculate terrain layers**. Under **Advanced settings**:

- **Analysis cell size:** the working grid. Don't go much finer than the source (15 m for GEM-15).
- **Smooth the elevation first (σ):** Gaussian smoothing before slope and TPI, which reduces noise.
- **Neighbourhood radius:** the window used for relative elevation and TPI.
- **Channel threshold:** the contributing area used to mark drainage channels.
- **Heal elevation around boundary:** voids a strip of this width outside the boundary, and any no-data holes inside it, and refills them by interpolation (default 25 m, 0 turns it off). Useful for edge artefacts and for GEM-15 gaps.

**Step 4: Zone creation.** Choose one layer, the number of zones (1 to 10), smoothing passes and a minimum zone size, then **Create zones**. Zones are k-means clusters numbered from the lowest to the highest layer average. Download a zone map as GeoTIFF, Shapefile or GeoJSON, with your choice of coordinate system for the GeoTIFF (vector files are always WGS84).

**Sampling design.**

- **Grid overlay:** set cell width and height and rotate the grid; it previews live on the map.
- **Bulk composite:** create composite zones across the field boundary.
- **Sample points:** generate points inside the boundary by **random**, **stratified**, **spatial coverage** or **composite** methods.
- **Drop pins manually** (yellow + button): by tap or click, right-click or long-press, or at your GPS location. Point IDs auto-increment. **Undo last** removes the last pin.
- **Draw zones:** draw zone polygons by hand; split with a drawn line, merge, rename or renumber. No boundary is needed.

**Step 5: Finish planning.** **Send layers to Collection** and choose which layers become background layers and which zone maps become zone sampling layers (with subsample pins).

## Collect tab

- **Load a sampling plan:** Shapefile (.zip), CSV, KML, KMZ or GeoJSON. Points become check-off targets; polygons become zones where you drop subsample pins.
- Check off each point as you sample it and add notes. The counter in the header shows your progress.
- **Edit mode** (per layer, behind a caution prompt): drag pins with confirmation, snap a pin to your current GPS location, or tap a pin to change its number or coordinates. Every move adds an automatic note with the old and new coordinates.
- **Background layers** from planning (imagery, terrain, zones) appear in the layer list. The eye shows or hides a layer and the arrow downloads it.
- GPS features need a secure context (HTTPS or localhost). The hosted GitHub Pages version qualifies.

## Saving, exporting and working offline

- **Project:** save or open `.fsproj` files. Imagery and terrain layers are saved as images, viewable but not re-downloadable as GeoTIFF; rebuild them in the Plan tab to download. The loaded elevation is not stored in the project, so reload it after opening one.
- **Export menu:** bulk GeoJSON and a PDF report for the whole project. Per layer: Shapefile, GeoJSON, CSV, KML and GPX for points, zones and subsample pins; GeoTIFF for imagery, terrain and zone layers.
- **Offline:** once loaded, the tool works with no signal. Switch the basemap to **No tiles (offline)**. Searching imagery, loading online elevation and loading map tiles need a connection.

## Troubleshooting

| Problem | What to try |
| --- | --- |
| "Set a field boundary or load points/zones first" | Elevation needs to know where your field is. Set a boundary, load points or zones, or zoom the map in to the field. |
| Elevation search failed | Check your connection, then try another source. Some networks block some hosts. You can always upload your own elevation GeoTIFF. |
| GEM-15: "No tile covers this field" | The area is outside GEM-15 coverage. Use HRDEM, CDEM or Copernicus. |
| GEM-15: "no elevation values over this field" | The field is mostly water, snow or ice, or a low-texture surface. Use another source. |
| GEM-15 slow on the first load | The catalogue is read first (a few small files), then the elevation window. If a tile can't be found by name, the tool indexes that tile group once, which can add a few seconds. |
| No satellite dates on the calendar | Raise the cloud limit or tick **Also show dates over the cloud limit**. Only scenes that fully cover the field are listed. |
| Map tiles failed to load | You are offline or the tile service is blocked. Choose **No tiles (offline)**; everything else still works. |
| GPS doesn't work | Open the tool from HTTPS (such as GitHub Pages) or `localhost`, and allow location access in the browser. |

## Online data sources and links

These are every online service Field Sampler can contact. All are public and need no account or API key. Nothing from your project is sent to them except the field's bounding box (as a search area) and map tile positions.

| Data | Used for | Endpoints and documentation |
| --- | --- | --- |
| **Esri World Street Map** (basemap tiles) | Street basemap | [Tile service](https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer) |
| **Esri World Imagery** (basemap tiles) | Satellite basemap (default) | [Tile service](https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer) |
| **Earth Search STAC API** (Element 84) | Finds Sentinel-2 scenes and Copernicus DEM tiles | [API root](https://earth-search.aws.element84.com/v1) · [search endpoint](https://earth-search.aws.element84.com/v1/search) · collections `sentinel-2-l2a` and `cop-dem-glo-30` |
| **Sentinel-2 L2A Cloud-Optimized GeoTIFFs** | Imagery, aerial and index layers (bands read directly from the scene files) | [AWS Open Data registry](https://registry.opendata.aws/sentinel-2-l2a-cogs/) |
| **Copernicus DEM GLO-30** | Copernicus elevation source | [AWS Open Data registry](https://registry.opendata.aws/copernicus-dem/) |
| **NRCan Canadian Digital Elevation Model (CDEM)** | CDEM elevation source | [Record on Open Government Portal](https://open.canada.ca/data/en/dataset/7f245e4d-76c2-4caa-951a-45d1d2051333) · [STAC item](https://datacube.services.geo.ca/stac/api/collections/cdem/items/cdem-canada) · [COG file](https://datacube-prod-data-public.s3.amazonaws.com/store/elevation/cdem-cdsm/cdem/cdem-canada-dem.tif) |
| **NRCan HRDEM (CanElevation Series) LiDAR** | HRDEM elevation source (DTM) | [Record on Open Government Portal](https://open.canada.ca/data/en/dataset/957782bf-847c-4644-a757-e383c0057995) · STAC search: [stac/api/search](https://datacube.services.geo.ca/stac/api/search) and [pgstac/api/search](https://datacube.services.geo.ca/pgstac/api/search) (collection `hrdem-lidar`) |
| **Google GEM-15** | GEM-15 elevation source (DTM) | [STAC catalogue](https://storage.googleapis.com/climate-and-sustainability/gem-15/stac/catalog.json) · [Cloud Storage bucket](https://storage.googleapis.com/climate-and-sustainability/gem-15) · [README](https://github.com/google-research/google-research/tree/master/gem_15) · [Earth Engine catalog](https://developers.google.com/earth-engine/datasets/catalog/projects_climate-and-sustainability_assets_gem-15_v1) · [Earth Engine app](https://climate-and-sustainability.projects.earthengine.app/view/gem-15) · [Preprint (DOI)](https://doi.org/10.31223/X55N75) |

**How GEM-15 is read.** The catalogue is a static STAC tree on Google Cloud Storage: `catalog.json`, then tile groups (such as `p40_p0`), then one item per 1° × 1° cell (`stac/gem-15/<group>/<cell>/<cell>.json`). Each item lists separate Cloud-Optimized GeoTIFFs (DTM, DSM and others) in the cell's local UTM zone. Plain `storage.googleapis.com/<bucket>/<object>` links don't allow browser (CORS) access, so Field Sampler reads the same files through the bucket's JSON API (`https://storage.googleapis.com/storage/v1/b/climate-and-sustainability/o/<object>?alt=media`), which allows CORS and byte-range requests. That is why only the part of each file covering your field is downloaded.

**Credits.** Imagery: Contains modified Copernicus Sentinel data. Copernicus DEM: produced using Copernicus WorldDEM-30, © DLR e.V. 2010-2014 and © Airbus Defence and Space GmbH 2014-2018, provided under the Copernicus programme by the European Union and ESA. NRCan CDEM and HRDEM: contain information licensed under the [Open Government Licence – Canada](https://open.canada.ca/en/open-government-licence-canada). Basemaps: © Esri. GEM-15: Google (experimental product; see the preprint: Batchu, Neumann, Li, Allen, Zhang and Van Arsdale, "GEM-15: Global surface and terrain elevation models (15 meters) from sub-meter optical stereo imagery", EarthArXiv, 2026). Always confirm each provider's current licence and attribution terms before publishing.

## Bundled libraries (no network needed)

These are included inside the page, so they work offline: [Leaflet](https://leafletjs.com) (maps), geotiff.js (GeoTIFF reading), proj4 (map projections), [jsPDF](https://github.com/MrRio/jsPDF) (PDF reports), JSZip (zip files), [pako](https://github.com/nodeca/pako) (compression) and [PapaParse](https://github.com/mholt/PapaParse) (CSV). Licences are kept in the page source.

## License and source

Field Sampler is free software, copyright (C) 2026 Cory Weber, licensed under the GNU Affero General Public License v3.0 or later. It comes with **no warranty**. You may redistribute and modify it under that licence, and anyone using a modified version over a network must be offered its source. The licence text is in [LICENSE](LICENSE), the source code is in this repository, and bundled library licences are in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Accuracy

Field Sampler is a planning and field-recording aid. Positions, boundaries, areas, grids, zones and map layers are approximate. Phone GPS is often accurate only to a few metres, and public satellite and elevation layers (Sentinel-2 at 10 to 20 m, elevation at 1 to 30 m) can be affected by cloud, shadow, timing and processing. Zones are statistical groupings, not agronomic, financial, legal or survey advice. Verify results on the ground.
