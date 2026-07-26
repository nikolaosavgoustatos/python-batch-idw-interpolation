# Batch IDW Interpolation with Percentile-Based Symbology

## Overview

This script performs batch **Inverse Distance Weighting (IDW)** interpolation for multiple geochemical elements and applies a consistent **7-class percentile-based classified symbology** in ArcGIS Pro.

The class breaks are:

* Minimum
* 5th percentile
* 25th percentile
* 50th percentile
* 75th percentile
* 90th percentile
* 95th percentile
* Maximum

using the **ColorBrewer RdYlBu (7 Classes)** color ramp (reversed so higher values appear in red).

The script stores the calculated class breaks in `idw_breaks.json` so they can be reused later without recalculating the interpolations.

---

# How to Run

## First Run (Create IDW rasters)

Set:

```python
RUN_IDW = True
```

Then run the script.

This will:

* Perform IDW interpolation for every field listed in `Z_FIELDS`.
* Save each raster as `IDW_<Field>.tif`.
* Calculate raster statistics.
* Compute percentile class breaks.
* Save the class breaks to `idw_breaks.json`.
* Apply the classified symbology to all newly created rasters.

---

## Second Run (Fix / Reapply Symbology)

After the first run completes successfully, change:

```python
RUN_IDW = False
```

and run the script **again**.

This second pass:

* Does **not** recreate the IDW rasters.
* Loads the previously saved percentile breaks from `idw_breaks.json`.
* Reapplies the classified symbology to every raster.
* Performs an additional verification pass because ArcGIS Pro may occasionally revert some raster layers to **Stretch** symbology during batch processing.

Running the script a second time ensures that all raster layers end with the intended manual classified symbology.

---

# Output Files

The script creates:

* `IDW_<Field>.tif` — interpolated raster for each element.
* `idw_breaks.json` — stored percentile class breaks used for reproducible symbology.

---

# Requirements

* ArcGIS Pro
* Spatial Analyst extension
* Python environment included with ArcGIS Pro
* NumPy

Run the script from the ArcGIS Pro Python window, Notebook, or an IDE configured to use the ArcGIS Pro Python environment with the target project open.

---

# Notes

* The script must be executed from an **open ArcGIS Pro project** (`CURRENT` project).
* The active map must contain the required input layers referenced by the script.
* Existing rasters are overwritten when `RUN_IDW = True`.
* The second execution (`RUN_IDW = False`) is intentional and recommended to ensure ArcGIS Pro retains the manual classified symbology on every raster layer.
