# Kerala rectangle — 2025 satellite layers

Open **qgis/Kerala_rectangle_2025_final_layers.qgz** in QGIS. The project has colors and legends already set. NDVI is initially visible. To view another variable, uncheck NDVI and check the desired layer. The blue line is Kerala; the purple line is the study rectangle.

The GeoTIFFs are the numerical data for analysis. The PNG is a preview. Files use lossless DEFLATE compression; no ZIP extraction is needed.

## Download the large rasters

This repository folder contains the project, styles, boundaries, processing reports, preview, and the small LULC raster. Download the following four files from the dataset Release into **the `rasters/` folder**, keeping their names unchanged:

- [NDVI_Kerala_rectangle_2025_February.tif](https://github.com/punyajitmishra/LandslideResearch/releases/download/kerala-satellite-2025-v1.0/NDVI_Kerala_rectangle_2025_February.tif)
- [NDWI_Kerala_rectangle_2025_February.tif](https://github.com/punyajitmishra/LandslideResearch/releases/download/kerala-satellite-2025-v1.0/NDWI_Kerala_rectangle_2025_February.tif)
- [LST_Celsius_Kerala_rectangle_2025_February.tif](https://github.com/punyajitmishra/LandslideResearch/releases/download/kerala-satellite-2025-v1.0/LST_Celsius_Kerala_rectangle_2025_February.tif)
- [LST_uncertainty_K_Kerala_rectangle_2025_February.tif](https://github.com/punyajitmishra/LandslideResearch/releases/download/kerala-satellite-2025-v1.0/LST_uncertainty_K_Kerala_rectangle_2025_February.tif)

Open `qgis/Kerala_rectangle_2025_final_layers.qgz` in QGIS once all five TIFFs are present. `SHA256SUMS.txt` records the SHA-256 checksums of the five numerical rasters. The original Landsat downloads, obsolete layer shortcut, and cached statistics are not part of this publication.

## Folder layout

```text
kerala_satellite_2025/
├── README.md
├── SHA256SUMS.txt
├── rasters/       # LULC in Git; download the four Release TIFFs here
├── boundaries/    # Study rectangle and Kerala reference outline
├── qgis/          # Ready-to-open project
│   └── styles/    # Reusable layer colors and legends
├── reports/       # Validation, coverage and processing records
└── previews/      # NDVI coverage image
```

Keep these folders together. The QGIS project uses relative paths. From this dataset folder on macOS, verify the five downloaded rasters with `shasum -a 256 -c SHA256SUMS.txt`.

## Files and meaning

| File prefix | Meaning | Values / units | Time represented |
|---|---|---|---|
| NDVI | Vegetation index | Dimensionless, −1 to 1 | Landsat observations on 18, 24 and 25 February 2025 |
| NDWI | McFeeters green/NIR water index | Dimensionless, −1 to 1 | Same February observations |
| LST_Celsius | Land surface temperature | Degrees Celsius; surface, not air temperature | Same February daytime satellite observations |
| LULC | Annual land-cover classification | Integer class codes below | Full calendar year 2025 |
| LST_uncertainty_K | USGS temperature uncertainty | Kelvin; temperature differences are also degrees Celsius | Matched to LST observations |

The NDWI supplied here is `(green − NIR)/(green + NIR)`. It is a surface-water index. The NIR/SWIR moisture index sometimes also called NDWI is a different variable, commonly called NDMI; these names must not be interchanged in the methods section.

## Common grid and boundary

- Study rectangle: **74.75–77.85° E, 8.15–12.90° N**. It includes sea and parts of neighboring states.
- CRS: **EPSG:32643, WGS 84 / UTM zone 43N**.
- Pixel spacing: **30 m**; all five rasters have matching grid alignment, dimensions and extent.
- NDVI, NDWI, LST and uncertainty are Float32 with **−9999 as NoData**. LULC is Byte with **0 as NoData**.
- A common extent does not imply a valid observation in every cell. Do not replace NoData with zero: zero can be a meaningful index value.
- Use `reports/Final_layer_validation.json` for final coverage, numerical ranges, compression and file sizes. Coverage is measured against the included Kerala reference outline.
- Landsat temperature is distributed on a 30 m grid, but the thermal sensor's native spatial detail is coarser than the optical bands.

## Landsat processing

Six downloaded Collection 2 Level-2 scenes were used:

```
LC08_L2SP_144052_20250225_20250304_02_T1
LC08_L2SP_144053_20250225_20250304_02_T1
LC08_L2SP_144054_20250225_20250304_02_T1
LC09_L2SP_145051_20250224_20250225_02_T1
LC09_L2SP_145052_20250224_20250225_02_T1
LC09_L2SP_145053_20250224_20250225_02_T1
```

Clear, unsaturated observations were selected using QA_PIXEL bits 0–5 and QA_RADSAT=0. Surface reflectance was scaled as `DN × 0.0000275 − 0.2`. Index inputs must have nonnegative corrected reflectance and a denominator greater than 0.000001.

- NDVI: `(B5 reflectance − B4 reflectance)/(B5 reflectance + B4 reflectance)`.
- NDWI: `(B3 reflectance − B5 reflectance)/(B3 reflectance + B5 reflectance)`.
- LST: `ST_B10 × 0.00341802 + 149 − 273.15`. Fill pixels and QA water pixels are excluded.
- Temperature uncertainty: `ST_QA × 0.01`. No additional uncertainty cutoff was imposed; retain this layer when reviewing unusually hot or uncertain pixels.

In the original six-scene mosaic, the last valid scene in lexicographic scene order wins overlaps. This is a spatial mosaic, not an average, and preserves each selected measurement. Differences between adjacent acquisitions may leave visible seams.

Two additional Tier 1 scenes, obtained through the Microsoft Planetary Computer's public USGS Landsat archive, fill NoData only within a southern/eastern window. The original valid mosaic values remain unchanged:

```
LC08_L2SP_143054_20250218_20250226_02_T1
LC08_L2SP_143055_20250218_20250226_02_T1
```

The 143054 scene is used first, followed by 143055 for remaining gaps. `reports/Southern_gap_fill_report.json` records the public source URLs and counts. Nearby dates reduce gaps but do not make this an annual average. Nearest-neighbor resampling was used for these Landsat inputs and continuous-layer mosaics.

## LULC source and classes

The LULC is the **Impact Observatory / Esri / Microsoft Sentinel-2 10 m annual land-cover map for 2025**, tile 43P, licensed **CC BY 4.0**. It is an existing global classification, not a new classifier trained on the downloaded Landsat scenes. Its accuracy for Kerala has not been independently assessed here.

The categorical source was resampled to the common 30 m grid using **mode** resampling. This preserves class codes and represents the dominant source class at the output scale; do not use bilinear interpolation for LULC. Retain the source attribution in research outputs.

| Code | Class |
|---|---|
| 0 | NoData |
| 1 | Water |
| 2 | Trees |
| 4 | Flooded vegetation |
| 5 | Crops |
| 7 | Built area |
| 8 | Bare ground |
| 9 | Snow/ice |
| 10 | Clouds — exclude from land-cover predictors |
| 11 | Rangeland |

Treat these class codes as categories rather than an ordered numerical scale when preparing the AI model.

## Boundary reference

`boundaries/kerala_state_four_corners.geojson` is the chosen study rectangle. `boundaries/Kerala_state_reference.geojson` is a separate outline used to verify coverage and display Kerala. The reference comes from geoBoundaries gbOpen IND ADM1, sourced from DataMeet / Election Commission of India, representing 2011, licensed CC BY 2.5 India. Final raster clipping uses the rectangle, not the state outline.

## Sources

- [USGS scale factors](https://www.usgs.gov/faqs/how-do-i-use-a-scale-factor-landsat-level-2-science-products)
- [USGS quality-assessment bands](https://www.usgs.gov/landsat-missions/landsat-collection-2-quality-assessment-bands)
- [USGS NDVI](https://www.usgs.gov/landsat-missions/landsat-normalized-difference-vegetation-index)
- [Esri annual land cover](https://livingatlas.arcgis.com/landcover/)
- [Impact Observatory land-cover documentation](https://docs.impactobservatory.com/lulc-maps/maps-for-good.html)
- [2025 tile 43P source GeoTIFF](https://lulctimeseries.blob.core.windows.net/lulctimeseriesv003/lc2025/43P_20250101-20251231.tif)
- [geoBoundaries India ADM1 reference metadata](https://www.geoboundaries.org/api/current/gbOpen/IND/ADM1/)
