# Coastal_GeoProcessing


Build a coastal Digital Terrain Model (DTM) from `.xyz` point clouds, generate shore-normal transects, and extract key geomorphological features (shoreline, dune toe, dune crest, and beach slope) along a coastline — all from a single Jupyter notebook.

> Note: the current notebook file name in the repo appears as `Excracting_features.ipynb` (with a typo). Consider renaming it to `Extracting_features.ipynb` for clarity.

# Why this exists

Coastal feature extraction often requires a mix of GIS tools and custom logic. This notebook offers a reproducible, lightweight workflow that:

* Reads multiple `.xyz` tiles and assembles a working DTM (point-based).
* Derives shore-normal transects at regular spacing.
* Detects shoreline, dune toe, dune crest, and computes beach slope per transect.
* Exports vectors/tables you can use directly in GIS or further analysis (e.g., runup/TWL).

# Features

* Read and clean multiple `.xyz` files (handle no-data like `-999`).
* Keep everything in a projected CRS (default used here: **EPSG:3763 PT-TM06/ETRS89** — adjust if needed).
* Generate shore-normal transects from coastline bearings.
* Sample elevations along transects and detect:

  * Shoreline (position/elevation)
  * Dune toe (position/elevation)
  * Dune crest (position/elevation)
  * Beach slope (useful for runup/TWL)
* Export GeoPackage/CSV and optional figures.

# Repository structure (suggested)

```
Coastal_GeoProcessing/
├─ README.md
├─ Extracting_features.ipynb          # or: Excracting_features.ipynb (current)
├─ env/
│  └─ environment.yml                 # optional conda environment
├─ data/                              # not tracked; put your .xyz here
│  ├─ raw/
│  └─ processed/
└─ outputs/
   ├─ transects.gpkg
   ├─ features.csv
   └─ figures/
```

> Tip: add `data/` and `outputs/` to `.gitignore` so large files don’t go to Git.

# Installation

You only need Python + the common geo stack. A conda environment is recommended but optional.

```yaml
# Save as: env/environment.yml
name: coastal-geoprocessing
channels:
  - conda-forge
dependencies:
  - python=3.10
  - jupyterlab
  - pandas
  - numpy
  - matplotlib
  - geopandas
  - shapely
  - rasterio
  - scipy
  - scikit-learn
```

Create and activate:

```bash
conda env create -f env/environment.yml
conda activate coastal-geoprocessing
jupyter lab
```

# Quick start

1. Put your `.xyz` tiles under `data/raw/`.
2. Open the notebook (`Extracting_features.ipynb`) in Jupyter.
3. Update the configuration cells:

   * Path to `data/raw/`.
   * File selection (e.g., `files_to_read = all_files[191:290]`).
   * CRS (if not EPSG:3763).
   * Transect spacing, length (e.g., 50 m seaward + 100 m landward), and sampling tolerance.
4. Run cells top-to-bottom.
5. Check `outputs/` for `transects.gpkg`, `features.csv`, and plots in `figures/`.

# Inputs

* **Format**: ASCII `.xyz` with whitespace separators
* **Columns**:

  * `x` (projected easting, meters)
  * `y` (projected northing, meters)
  * `z` (elevation, meters)
* **CRS**: projected CRS in meters is required for distances/slopes (default used here: EPSG:3763).
* **No-data**: values like `-999` are cleaned in the notebook — adjust if your dataset uses a different sentinel.

# Outputs

* **Vector** (GeoPackage/GeoJSON): shore-normal transects with attributes such as ID and azimuth.
* **Table** (`features.csv`), one row per transect (typical columns):

  * `x_shoreline, y_shoreline, z_shoreline`
  * `x_toe, y_toe, z_toe`
  * `x_crest, y_crest, z_crest`
  * `beach_slope`
  * optional quality/diagnostic flags
* **Figures**: elevation profiles per transect with markers for detected features.

> Actual column names may differ slightly; see the final cells of the notebook for the exact schema used.

# Workflow (high-level)

```mermaid
flowchart LR
    A[.xyz point clouds] --> B[Load & clean]
    B --> C[GeoDataFrame (projected CRS)]
    C --> D[Coastline & local bearing]
    D --> E[Generate shore-normal transects]
    E --> F[Sample elevations along transects]
    F --> G[Detect shoreline/toe/crest]
    G --> H[Compute beach slope]
    H --> I[Export: GPKG, CSV, figures]
```

# Key parameters & tips

* **Transect geometry**

  * *Spacing*: every Nth coastline point or fixed distance alongshore.
  * *Length*: symmetric about coastline point (e.g., 50 m seaward + 100 m landward).
  * *Angle*: derive local coastline bearing; add/subtract 90° for shore-normal.
* **Sampling**

  * Use a small buffer/tolerance when querying DTM points near the transect.
  * Optional smoothing before detecting slope breaks/peaks.
* **Feature detection**

  * Crest/toe often correspond to slope/curvature changes or local maxima/minima.
  * Always spot-check in plots/GIS, especially where dunes are absent or armored.
* **CRS**

  * Keep everything in meters. Reproject inputs if necessary.

# Validation

* Visual inspection of a subset of transects (plots in the notebook).
* Cross-check positions/elevations in GIS.
* (Optional) Compare results against known coastal hazard maps or prior studies.

# Good practices

* Keep `data/raw/` immutable; write cleaned/intermediate to `data/processed/`.
* Version outputs (e.g., `features_2025-08-15.csv`).
* Record key parameters in a small YAML/JSON (future improvement).

Add this to `.gitignore` (create if missing):

```
data/
outputs/
*.ipynb_checkpoints
.DS_Store
.env
```



# FAQ

* **My data aren’t in EPSG:3763.**
  Update the CRS and reproject data/outputs to your local projected CRS (meters).

* **Where do I change transect spacing/length?**
  In the configuration cell near the top of the notebook (look for comments like “transect parameters”).

* **Why point-based DTM (not raster)?**
  It avoids committing to a grid resolution early and keeps original elevations; you can rasterize later if needed.

# Citation

If this notebook helps your work, please cite:

```
Barros, J.E.C. (2025). Coastal_GeoProcessing: shoreline & dune feature extraction from xyz DTMs.
GitHub repository.
```

# License

Choose a license (e.g., MIT or Apache-2.0) and add a `LICENSE` file at the repo root. If unsure, MIT is a simple permissive option.

# Acknowledgments

Developed as part of coastal hazard analysis workflows, including applications in Northern Portugal. Thanks to the open-source geo community (GeoPandas, Shapely, Rasterio, etc.) for outstanding tools.

---

**Next small steps (optional):**

* Rename the notebook to `Extracting_features.ipynb`.
* Add `env/environment.yml` (from above) to make setup one-command.
* Commit a tiny sample `.xyz` snippet (just a few lines) so others can test the notebook quickly.

