# Prompt: add the Google GEM-15 (15 m) elevation source to a single-file HTML tool

Copy everything below the line into a new chat together with the HTML file you want changed. It was written from the implementation that is now in Field Sampler, so the code in section 3 is the same code, tested against a mock of the GEM-15 STAC catalogue.

---

You are adding **Google GEM-15** as a new elevation (DEM) source to the attached single-file HTML tool. Keep every existing source and feature working. Change only what is needed.

## 1. Goal

- Add one new option to the tool's existing elevation/DEM source selector: **"Google GEM-15 DTM: STAC (near-global, 15 m bare earth)"**. Place it after any LiDAR or national option and before Copernicus GLO-30 (or after the last existing option if there is no Copernicus).
- It must load the same way the existing online sources do: the user picks it, presses the tool's existing search/load button, and the elevation window for the current field/extent is read and fed into the tool's existing elevation pipeline (display, sampling, terrain derivatives, zoning, export). No new workflow.
- First read how the tool loads its existing COG-based DEMs (look for `GeoTIFF.fromUrl`, `readRasters`, STAC `fetch` calls, a source `<select>`, and a DEM state object). Reuse that reader and data structure. Do not invent a parallel one unless the tool has no projected-CRS support (see 4.3).

## 2. What GEM-15 is (facts to design around)

- Global surface (DSM) and terrain (DTM) elevation models at 15 m from sub-metre optical stereo imagery. Experimental. Land between about 60°S and 84°N, with some regions left out. Use the **DTM** (bare earth) unless the tool is explicitly about surface height.
- **Static STAC catalogue** (no API server, no search endpoint): `https://storage.googleapis.com/climate-and-sustainability/gem-15/stac/catalog.json`
  Layout: `catalog.json` → tile groups (folders named like `p40_p0`, meaning lat block 40, lon block 0, `p` = plus, `m` = minus) → one STAC item per **1° × 1° cell** at `stac/gem-15/<group>/<cell>/<cell>.json`. Cells are named by south-west corner, for example `w6s45` = lon 6 to 7, lat 45 to 46 (the Chamonix area, group `p40_p0`).
- Each item lists **separate Cloud-Optimized GeoTIFFs** per product (DTM, DSM, percentile and standard-deviation layers). Each COG is in the cell's **local UTM zone** (EPSG in `proj:epsg` on the item and in the GeoTIFF keys), **float32 metres**, **NaN = no data** (water, snow/ice, cloud shadow, low-texture ground; gaps are normal). A DTM cell is roughly 140 MB, a DSM cell several times that, so **never download whole files**: read only a window.
- **Vertical datum is EGM96 orthometric height** (EPSG:5773), not WGS84 ellipsoidal height. Copernicus GLO-30 uses EGM2008 and Canadian sources use CGVD-based datums, so absolute heights differ by metres between sources. Relative terrain products (slope, TPI, relative elevation, curvature) are unaffected.
- Other access routes exist (Earth Engine asset `projects/climate-and-sustainability/assets/gem-15_v1`, an `items.ndjson` file, `gs://climate-and-sustainability/gem-15`) but a browser tool should use the static STAC + COG route below. Do not add Earth Engine, Python or a server.
- README: https://github.com/google-research/google-research/tree/master/gem_15 · Earth Engine catalog: https://developers.google.com/earth-engine/datasets/catalog/projects_climate-and-sustainability_assets_gem-15_v1 · Preprint: https://doi.org/10.31223/X55N75
- Credit as **GEM-15, Google**. Third-party catalogues list the licence as CC BY 4.0 with the attribution "This dataset is produced by Google"; confirm on the official page before telling users so.

### The one trap: CORS

Plain `https://storage.googleapis.com/<bucket>/<object>` URLs send **no CORS headers**, so `fetch` and `GeoTIFF.fromUrl` fail in a browser. The same objects are readable with CORS and `Range` support through the bucket's JSON API:

`https://storage.googleapis.com/storage/v1/b/<bucket>/o/<url-encoded object>?alt=media`

Every request (catalogue JSON and COG reads) must go through that form. `GEM_cors()` below does the rewrite, including for `gs://` hrefs, and falls back to the plain URL.

## 3. Reference implementation (insert as-is, then adapt only the reader in 4.2)

Catalogue discovery handles three things I could not confirm against the live bucket, so it is written defensively: the exact asset key names (it matches `dtm` / `dsm` by name and skips `p5`/`p95`/`std` layers), the cell naming for **negative** longitudes and latitudes (it tries to parse the name and, if no cell matches, scans the group's items once and checks each item's `bbox`), and whether cells are linked as `child` or `item`. Keep it that way.

```js
/* ---------------------------------------------------------------- GEM-15 (Google)
   GEM-15: near-global 15 m surface (DSM) and terrain (DTM) elevation models from sub-metre optical stereo imagery (Google, experimental).
   Static STAC catalogue on Google Cloud Storage:  catalog.json -> tile groups (e.g. p40_p0) -> 1x1 degree cells (one item JSON per cell)
   -> separate Cloud-Optimized GeoTIFFs per product (DTM, DSM, ...) in the cell's local UTM zone. Float32 metres, EGM96 orthometric heights,
   NaN = no data (water, snow/ice, cloud shadow, low-texture ground).
   Plain storage.googleapis.com/<bucket>/<object> URLs send no CORS headers, so a browser app must read through the bucket's JSON API
   (storage/v1/b/<bucket>/o/<object>?alt=media), which allows CORS and Range requests. GEM_cors() does that rewrite for every request.
   Needs the host page to provide: GeoTIFF (geotiff.js) for GEM_read().  GEM_find() and the helpers have no dependencies.            */
const GEM = {
  root: "https://storage.googleapis.com/climate-and-sustainability/gem-15/stac/catalog.json",
  json: new Map(), groups: null, scans: new Map(),
};
function GEM_pub(h) { const m = /^gs:\/\/([^/]+)\/(.*)$/.exec(h); return m ? "https://storage.googleapis.com/" + m[1] + "/" + m[2] : h; }
function GEM_cors(h) {
  h = GEM_pub(h);
  const m = /^https?:\/\/storage\.googleapis\.com\/(?!storage\/)([^/?#]+)\/([^?#]+)/.exec(h) || /^https?:\/\/([^./]+)\.storage\.googleapis\.com\/([^?#]+)/.exec(h);
  if (!m) return h;
  let o = m[2]; try { o = decodeURIComponent(o); } catch (e) { /* keep as is */ }
  return "https://storage.googleapis.com/storage/v1/b/" + m[1] + "/o/" + encodeURIComponent(o) + "?alt=media";
}
function GEM_json(url) {
  if (!GEM.json.has(url)) {
    const p = (async () => {
      let err = "";
      for (const u of [...new Set([GEM_cors(url), url])]) {
        try { const r = await fetch(u); if (r.ok) return await r.json(); err = "HTTP " + r.status; } catch (e) { err = e.message; }
      }
      throw new Error("could not read " + url.split("/").slice(-2).join("/") + " (" + err + ")");
    })();
    GEM.json.set(url, p); p.catch(() => GEM.json.delete(url));
  }
  return GEM.json.get(url);
}
const GEM_kids = (j, base) => (j.links || []).filter((l) => l.href && (l.rel === "child" || l.rel === "item")).map((l) => new URL(GEM_pub(l.href), base).href);
/* tile groups are folders named <p|m><lat>_<p|m><lon> (p = plus, m = minus), e.g. p40_p0 holds the cells for 40..N, 0..E */
const GEM_GRP = /\/([pm])(\d+)_([pm])(\d+)(?=[/.]|$)/i;
/* cells are named w<west lon>s<south lat>, e.g. w6s45 = lon 6..7, lat 45..46; negative values are matched by their sign prefix, and anything unrecognised falls back to a bbox scan */
const GEM_CELL = /\/w(-|m|p)?(\d+)s(-|m|p)?(\d+)(?=[/.]|$)/i;
async function GEM_groups() {
  if (GEM.groups) return GEM.groups;
  const seen = new Set(); let level = [GEM.root];
  for (let d = 0; d < 4 && level.length; d++) {
    const next = [];
    for (const u of level) {
      if (seen.has(u)) continue; seen.add(u);
      const kids = GEM_kids(await GEM_json(u), u), gs = kids.filter((k) => GEM_GRP.test(k));
      if (gs.length) return (GEM.groups = gs.map((url) => { const m = GEM_GRP.exec(url); return { url, lat: (m[1].toLowerCase() === "m" ? -1 : 1) * +m[2], lon: (m[3].toLowerCase() === "m" ? -1 : 1) * +m[4] }; }));
      next.push(...kids.slice(0, 6));
    }
    level = next;
  }
  throw new Error("the GEM-15 STAC catalogue layout was not recognised");
}
function GEM_pickGroup(groups, lat, lon) {
  const la = Math.max(...groups.map((g) => g.lat).filter((v) => v <= lat)), lo = Math.max(...groups.map((g) => g.lon).filter((v) => v <= lon));
  return groups.find((g) => g.lat === la && g.lon === lo) || null;
}
async function GEM_item(url, depth) {
  const j = await GEM_json(url), d = depth || 0;
  if (j.type === "Feature") return j;
  if (d < 2) for (const k of GEM_kids(j, url).slice(0, 3)) { try { const it = await GEM_item(k, d + 1); if (it) return it; } catch (e) { /* try next link */ } }
  return null;
}
const GEM_hit = (it, bb) => { const b = it.bbox; return !b || (b[0] < bb[2] && b[2] > bb[0] && b[1] < bb[3] && b[3] > bb[1]); };
function GEM_scan(g, cells, setStatus) {
  if (!GEM.scans.has(g.url)) {
    GEM.scans.set(g.url, (async () => {
      const out = [], list = cells.slice(0, 400); let i = 0, done = 0;
      await Promise.all(Array.from({ length: 8 }, async () => {
        while (i < list.length) {
          const u = list[i++];
          try { const it = await GEM_item(u); if (it) out.push(it); } catch (e) { /* skip */ }
          if (setStatus && ++done % 10 === 0) setStatus("Indexing GEM-15 tiles… " + done + "/" + list.length);
        }
      }));
      return out;
    })());
  }
  return GEM.scans.get(g.url);
}
/* STAC items (one per 1x1 degree cell) that touch bbox = [west, south, east, north] in WGS84 */
async function GEM_find(bb, setStatus) {
  const groups = await GEM_groups(), got = new Map(), want = new Set();
  for (const x of [bb[0], bb[2]]) for (const y of [bb[1], bb[3]]) want.add(Math.floor(x) + "," + Math.floor(y));
  for (const key of want) {
    const [lo, la] = key.split(",").map(Number), g = GEM_pickGroup(groups, la + 0.5, lo + 0.5);
    if (!g) continue;
    const cells = GEM_kids(await GEM_json(g.url), g.url), sg = (s) => (s && /^[m-]$/i.test(s) ? -1 : 1);
    const urls = cells.filter((u) => { const m = GEM_CELL.exec(u); return m && sg(m[1]) * +m[2] === lo && sg(m[3]) * +m[4] === la; });
    let items = [];
    for (const u of urls.slice(0, 2)) { try { const it = await GEM_item(u); if (it) items.push(it); } catch (e) { console.warn(e); } }
    items = items.filter((it) => GEM_hit(it, bb));
    if (!items.length) items = (await GEM_scan(g, cells, setStatus)).filter((it) => GEM_hit(it, bb));
    items.forEach((it) => got.set(it.id, it));
  }
  return [...got.values()];
}
/* kind = "dtm" (bare earth) or "dsm" (surface). The percentile / standard-deviation layers are skipped. */
function GEM_asset(it, kind) {
  const A = it.assets || {}, ks = Object.keys(A), re = new RegExp(kind, "i"), bad = /(^|[_-])(p\d+|std|sd|stdev|count|mask)([_-]|$)/i;
  const k = ks.find((x) => x.toLowerCase() === kind) || ks.find((x) => re.test(x) && !bad.test(x)) || ks.find((x) => re.test((A[x].title || "") + " " + (A[x].description || "")) && !bad.test(x));
  return k ? A[k] : null;
}
/* opens the COG through the CORS-enabled endpoint; returns { tiff, epsg } for the host page to window-read */
async function GEM_open(it, kind) {
  const a = GEM_asset(it, kind);
  if (!a) throw new Error("no " + kind.toUpperCase() + " asset in " + it.id + " (assets: " + Object.keys(it.assets || {}).join(", ") + ")");
  const p = it.properties || {}, epsg = p["proj:epsg"] || (/(\d+)$/.exec(String(p["proj:code"] || a["proj:code"] || "")) || [])[1] || a["proj:epsg"] || null;
  let tiff, err;
  for (const u of [...new Set([GEM_cors(a.href), GEM_pub(a.href)])]) { try { tiff = await GeoTIFF.fromUrl(u); break; } catch (e) { err = e; } }
  if (!tiff) throw new Error("could not open the GEM-15 " + kind.toUpperCase() + " file for " + it.id + " — " + (err && err.message));
  return { tiff, epsg: epsg ? +epsg : null };
}
```

## 4. Integration steps

1. **Add the option** to the source selector, plus a small hint element that shows this note only while GEM-15 is selected (hidden for other sources):
   `GEM-15 is an experimental Google model: 15 m cells, heights above the EGM96 geoid (terrain layers are unaffected). Credit: GEM-15, Google. Water, snow/ice and low-texture ground can have gaps, and bare-earth accuracy drops under dense forest.`
2. **Reader.** Add an async function (name it after the tool's conventions) that takes the tool's current bbox `[west, south, east, north]` in WGS84 and:
   - calls `GEM_find(bbox, setStatus)`; if it returns nothing, throw: `No GEM-15 tile covers this field. GEM-15 covers land between about 60°S and 84°N and leaves some regions out.`
   - for up to 4 items, calls `GEM_open(item, "dtm")` and then reads a **window** of `await tiff.getImage()` around the bbox (pad about 150 m), using the tool's existing window reader and passing `epsg` as the CRS fallback. Read with `Promise.allSettled`; ignore cells that fail with "outside"; if none can be read, show the first real error.
   - drops rasters that contain no finite value above -500 and throws `GEM-15 has no elevation values over this field (water, snow/ice, cloud shadow and low-texture ground leave gaps). Use Copernicus or CDEM instead.` if none remain.
   - returns the tool's normal "list of rasters + source label" result, label `Google GEM-15 DTM (bare earth)`. When several cells are returned, the existing "first raster with a finite value wins" sampling should handle the overlap; if the tool only supports one raster, merge by sampling.
3. **Projected CRS.** GEM cells are UTM, not EPSG:4326. If the tool already reads projected COGs (NRCan CDEM or HRDEM in a Canadian tool, for example) it already has a bbox-to-pixel-window step and a lon/lat to map-CRS conversion for sampling: reuse them. If it only handles EPSG:4326, add the minimum: define the UTM CRS with proj4 (`EPSG:326nn` → `+proj=utm +zone=nn +datum=WGS84 +units=m +no_defs`, `EPSG:327nn` the same with `+south`), project the bbox corners to UTM to get the pixel window from `image.getOrigin()` and `image.getResolution()`, and project lon/lat to UTM before sampling the raster. Make sure proj4 and geotiff.js are present (reuse the tool's copies; if missing, inline pinned versions so a single-file tool still works offline).
4. **NaN handling.** Treat every non-finite value (and nodata) as missing everywhere downstream: min/max and legends, colour ramps, hillshade, slope and curvature, interpolation, statistics, exports. Never turn NaN into 0. If the tool already has a gap-filling or "heal"/inpaint step for DEM holes, make sure it applies to GEM-15 as well.
5. **Datum.** If the tool shows or exports absolute elevation, or mixes elevation sources in one project, label the source on the output (the label above does this) and mention in the help that GEM-15 is EGM96. Do not convert datums unless the tool already does.
6. **Errors and status.** Use the tool's existing status line and error toast. Status messages: `Searching the Google GEM-15 catalogue…`, `Indexing GEM-15 tiles… n/N` (from the fallback scan), `Reading GEM-15 DTM from n tile(s)…`. Do not block the UI thread for long; the existing pattern for other sources is the model.
7. **Attribution and help.** If the tool has a help dialog, README, "data sources" list or credits, add GEM-15 with these links: STAC catalogue `https://storage.googleapis.com/climate-and-sustainability/gem-15/stac/catalog.json`, bucket `https://storage.googleapis.com/climate-and-sustainability/gem-15`, README, Earth Engine catalog and the preprint DOI above, and one sentence explaining the CORS route (JSON API) so the next person doesn't "fix" it back to plain URLs. If the tool has an attribution control or PDF/report credits that list data sources, add "GEM-15, Google".
8. **Offline and sizes.** The tool must still work offline for everything that worked offline before. Cache nothing large; the module already caches the small catalogue JSON in memory.

## 5. Rules

- Do not remove, rename or reorder existing sources or change their behaviour. Do not add dependencies beyond what is needed for UTM (proj4) and COG reading (geotiff.js), and only if the tool lacks them.
- No API keys, no server, no Earth Engine, no Python.
- Do not rewrite unrelated code, and do not reformat the file.
- Match the tool's code style, naming prefix and UI patterns. Do not introduce new design patterns for the dropdown, hint or status line.

## 6. Testing (required before you report done)

You probably cannot reach the real bucket from your sandbox, so test against a **mock** and say clearly that the live catalogue was not exercised. Using Playwright (or the tool's own test method):

1. Generate small synthetic UTM Cloud-Optimized GeoTIFFs (float32, tiled 256, zlib, `ModelPixelScale` 15, `ModelTiepoint`, a `GeoKeyDirectory` with the UTM EPSG, GDAL nodata NaN, with a NaN hole) for DTM, DSM and a `dsm_p95` layer, and a mock STAC tree: `catalog.json` → `gem-15/catalog.json` → several `p40_m90`-style group catalogs → cell items with `bbox`, `properties["proj:epsg"]` and `assets` hrefs.
2. Intercept requests to `https://storage.googleapis.com/**` (for example `page.route`). Serve the objects **only** on `/storage/v1/b/climate-and-sustainability/o/<encoded>?alt=media` with CORS headers and `Range` support (`206` with `Content-Range`); answer plain object URLs without CORS headers, as the real bucket does.
3. Run three scenarios: (A) positive coordinates with the documented names (`p40_p0`, `w6s45`); (B) negative longitude with `wm81s43`-style names; (C) negative longitude with `w81s43`-style names and `gs://` hrefs, which must succeed through the fallback scan.
4. Assert: the source loads; the raster has the UTM CRS and 15 m cells; sampled values match the synthetic surface within 0.01 m; the NaN hole stays NaN; a field in the ocean or outside any group gives the "No GEM-15 tile covers this field" message; no plain (non-CORS) requests are made; page errors are empty. Check that the other sources' options are still present and unchanged.

## 7. Verify against the live catalogue (hand to the user)

At the end, give me this snippet to paste in the browser console on the page, on a normal internet connection, for a field in my area, and tell me what output means success:

```js
const it = await GEM_find([-80.52, 43.48, -80.50, 43.50], console.log);
console.log(it.map(i => i.id), Object.keys(it[0].assets), it[0].properties["proj:epsg"], it[0].bbox);
```

Success: at least one item, asset keys including a DTM entry, a UTM EPSG code (for example 32617 for southern Ontario) and a bbox around the field. If the item loads but no DTM asset is found, the error lists the real asset keys; adjust `GEM_asset()` to match.

## 8. Deliverable

The modified file (same filename, plus a version/date note if the tool has one), and a short summary of: what changed and where, how the tool's existing reader was reused, anything you could not verify, and the console snippet result I should look for.
