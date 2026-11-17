# Third-party notices

Field Sampler's own code is licensed under the [GNU AGPLv3 or later](LICENSE). `index.html` also bundles the open-source libraries below. Each remains under its own licence, and its copyright and licence notices are preserved in the `index.html` source (see the `Bundled license information` comment block and the per-library headers near the end of the file). All are compatible with distribution under AGPLv3.

| Library | Used for | Licence |
| --- | --- | --- |
| [Leaflet](https://leafletjs.com) 1.9.4 | Maps | BSD-2-Clause |
| [geotiff.js](https://github.com/geotiffjs/geotiff.js) | Reading GeoTIFFs | MIT |
| [LERC](https://github.com/Esri/lerc) (Esri) | LERC decoding for GeoTIFFs | Apache-2.0 |
| [proj4js](https://github.com/proj4js/proj4js) | Map projections | MIT |
| [jsPDF](https://github.com/parallax/jsPDF) 4.2.1 | PDF reports | MIT |
| [html2canvas](https://html2canvas.hertzen.com) 1.4.1 | Page capture for PDF reports | MIT |
| [svg-pathdata](https://github.com/nfroidure/svg-pathdata) | SVG path parsing (jsPDF dependency) | MIT |
| [DOMPurify](https://github.com/cure53/DOMPurify) 3.4.14 | HTML sanitising | Apache-2.0 or MPL-2.0 (used under Apache-2.0) |
| [JSZip](https://stuk.github.io/jszip/) 3.10.1 | Zip / shapefile / KMZ handling | MIT or GPLv3 (used under MIT) |
| [pako](https://github.com/nodeca/pako) 2.2.0 | Compression | MIT AND Zlib |
| [Papa Parse](https://www.papaparse.com) 5.6.0 | CSV parsing | MIT |
| [toGeoJSON](https://github.com/mapbox/togeojson) | KML / GPX to GeoJSON | BSD-2-Clause |

## Data and imagery credits

These are not bundled; they are fetched online on request. See [HELP.md](HELP.md#online-data-sources-and-links) for endpoints and the full credit text.

- Imagery: Contains modified Copernicus Sentinel data.
- Copernicus DEM: produced using Copernicus WorldDEM-30, © DLR e.V. 2010-2014 and © Airbus Defence and Space GmbH 2014-2018, provided under the Copernicus programme by the European Union and ESA.
- NRCan CDEM and HRDEM: contain information licensed under the [Open Government Licence – Canada](https://open.canada.ca/en/open-government-licence-canada).
- Basemaps: © Esri.
- Google GEM-15: Google (experimental product; see the HELP.md credit for the reference).

## Keeping this file current

If you add, remove, or upgrade a bundled library, update this table in the same pull request.
