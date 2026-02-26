# New Data Sources for Water Quality Prediction (South Africa)

Use these **publicly available datasets** to add features beyond Landsat + PET. They are relevant to runoff, dilution, erosion, and land use—all drivers of alkalinity, conductance, and phosphorus.

---

## 1. More TerraClimate Variables (easiest – same pipeline)

You already use **PET** from TerraClimate via the Planetary Computer. The same Zarr dataset has **14 variables**. Add these with your existing `load_terraclimate_dataset()`, `filterg()`, and `assign_nearest_climate()`.

| Variable | Description | Why relevant for water quality |
|----------|-------------|--------------------------------|
| **ppt** | Monthly precipitation (mm) | Dilution, runoff, nutrient load |
| **soil** | Soil moisture, total column (mm) | Runoff potential, erosion |
| **aet** | Actual evapotranspiration (mm) | Water balance, concentration of solutes |
| **q** | Runoff (mm) | Direct link to streamflow and pollutant transport |
| **def** | Climatic water deficit | Drought / concentration effects |
| **tmin** / **tmax** | Min/max temperature (°C) | Biological and chemical rates |
| **vpd** | Vapor pressure deficit | Evaporation, water stress |
| **srad** | Downward shortwave radiation | Energy for evaporation and biology |

**How to add (same notebook):**

- After `ds = load_terraclimate_dataset()`, run `filterg(ds, 'ppt')`, then `assign_nearest_climate(..., 'ppt')` for training and validation. Repeat for `soil`, `aet`, `q`, etc.
- Merge the new columns (e.g. `ppt`, `soil`, `aet`, `q`) into your TerraClimate CSVs by matching `Latitude`, `Longitude`, `Sample Date`, then add those columns to the benchmark model’s feature set.

**TerraClimate overview:**  
https://planetarycomputer.microsoft.com/dataset/terraclimate  
**Variable list (e.g. for Zarr):**  
https://www.climatologylab.org/terraclimate-variables.html

---

## 2. Elevation (DEM)

Elevation affects slope, flow accumulation, and erosion. Useful for alkalinity and sediment‑related parameters.

**Option A – Open Topo Data (no STAC, good for point extraction)**  
- **Data:** ASTER GDEM 30 m  
- **API:** `GET https://api.opentopodata.org/v1/aster30m?locations=lat,lng`  
- **Use:** For each row in `water_quality_training_dataset.csv` and `submission_template.csv`, call the API with `Latitude,Longitude` and store elevation in a new column (e.g. `elevation_m`).  
- **Docs:** https://www.opentopodata.org/datasets/aster/

**Option B – Planetary Computer (raster, same ecosystem)**  
- **Data:** Copernicus DEM GLO-30, NASADEM, or ASTER  
- **Use:** STAC search by bbox/time, then sample at your point coordinates (similar to how you sample Landsat/TerraClimate).  
- **Catalog:** https://planetarycomputer.microsoft.com/catalog (search “DEM” or “elevation”)

---

## 3. Land Cover (ESA WorldCover)

Land use (water, crops, urban, forest) drives runoff quality and quantity.

- **Data:** ESA WorldCover 10 m, 2020/2021  
- **Where:** Microsoft Planetary Computer (STAC)  
- **Use:** Load the WorldCover asset for South Africa (or your area of interest), sample at each `Latitude, Longitude` (and optionally aggregate in a small buffer). Use the class label (e.g. 10 = trees, 20 = shrubland, 40 = cropland, 50 = urban, 80 = water) as a categorical or one‑hot feature.  
- **Docs:** https://planetarycomputer.microsoft.com/dataset/esa-worldcover (or search “WorldCover” in the catalog)

---

## 4. Other satellite / climate (optional)

- **MODIS vegetation indices (e.g. NDVI):** Planetary Computer has MODIS products; NDVI can proxy vegetation and seasonal growth (relevant for nutrients and erosion).  
- **CHIRPS precipitation:** Alternative to TerraClimate `ppt`; higher resolution in some regions. May require a different ingestion (e.g. Google Earth Engine or direct CHIRPS API) if you want a second precipitation source.

---

## Suggested order of implementation

1. **TerraClimate:** Add **ppt**, **soil**, **aet**, **q** using your existing TerraClimate notebook; merge into training/validation CSVs and add these as features in the benchmark model.  
2. **Elevation:** Add **elevation_m** via Open Topo Data API (or a DEM on Planetary Computer) for each training and validation point.  
3. **Land cover:** Add **WorldCover** class at each point (and optionally buffer stats) from Planetary Computer.

After adding a new source, re-run the benchmark notebook with the updated feature set and check the leaderboard score.
