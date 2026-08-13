Here is a professional, comprehensive `README.md` formatted for GitHub. It explains what the script does, how it works, the configuration parameters, and how to run it.

***

# ArcGIS Pro Batch IDW Interpolation & Percentile Symbology

This Python script automates the spatial interpolation of geochemical point data (e.g., soil or sediment samples) using Inverse Distance Weighting (IDW) in ArcGIS Pro. It calculates exact percentile-based classification breaks from the original point data, generates IDW rasters for multiple elements, and applies highly customized classified symbology with precise legend formatting.

## Key Features

- **Batch Processing:** Runs IDW interpolation for 14 elements (As, Ba, Ca, Cr, Cu, Fe, Mn, Ni, Pb, Rb, Sr, Ti, V, Zn).
- **Mathematically Pure Classification:** Extracts percentile breaks (min, 5th, 25th, 50th, 75th, 90th, 95th, max) directly from the sample points. The actual classification break values are kept as exact floats so the raster classification perfectly matches the data distribution.
- **Custom Legend Formatting:** Generates contiguous, non-overlapping legend ranges (e.g., `4.2 - 5.0`, `5.1 - 6.7`). Display labels use standard half-up rounding (e.g., `5.05` -> `5.1`), while the underlying raster bounds remain exact.
- **CIM-Level Symbology Locking:** Uses the ArcGIS Pro CIM (Cartographic Information Model) API to lock manual class breaks, apply the "Prediction" color ramp, force descending legend order, and permanently set the layer blend mode to `Multiply`.
- **Symbology-Only Mode:** Ability to skip interpolation and re-apply/fix symbology on existing raster layers.
- **Map Exporting:** Optional automated export of per-element TIFF maps with the applied symbology.

## Requirements

- **ArcGIS Pro** (3.x recommended)
- **Spatial Analyst Extension** (checked out automatically by the script)
- **Python 3.x** (comes bundled with ArcGIS Pro)
- **NumPy** (comes bundled with ArcGIS Pro)
- A color ramp named **"Prediction"** available in the active ArcGIS Pro project's styles.

## Configuration

Before running the script, modify the following variables in the `Inputs and parameters` section to match your environment:

```python
IN_POINTS = "As mg/kg"          # Name of the input point feature layer
Z_FIELDS = [...]                # List of fields to interpolate
OUT_FOLDER = r"C:\path\to\outputraster" # Output directory for TIFFs
OUT_CELL_SIZE = 50              # Output raster cell size
POWER = 2                       # IDW power parameter
SEARCH_RADIUS = arcpy.sa.RadiusVariable(12, None) # 12 nearest points
IN_BARRIER = ""                 # Optional barrier polyline features

PERCENTILES = [5, 25, 50, 75, 90, 95] # Percentiles for the 7 classes
COLOR_RAMP_NAME = "Prediction"  # Must match a ramp in your Pro styles

RUN_IDW = True                  # False = apply symbology to existing rasters only
EXPORT_LOG_MAPS = False         # True = export per-element TIFF maps
```

### Geoprocessing Environment
The script sets specific environment settings during execution:
- **Extent:** `andros_gys` (Change this string to match your study area extent layer)
- **Mask:** `andros_municipality_community` (Change this to your mask layer)
- **Compression:** LZW
- **Output Coordinate System:** Matches the active map's spatial reference.

## How It Works

The script runs in three distinct phases:

1. **Phase 1: IDW Interpolation** (If `RUN_IDW = True`)
   - Iterates through the `Z_FIELDS`.
   - Executes the `arcpy.sa.Idw` tool.
   - Saves the output as `IDW_<Element>.tif` (e.g., `IDW_As.tif`).
   - Calculates raster statistics.

2. **Phase 2: Symbology Application**
   - Extracts exact percentile breaks from `IN_POINTS` for the given field.
   - Creates contiguous display labels using a custom `round_half_up` function (avoids Python's default banker's rounding so `5.05` correctly rounds to `5.1`).
   - Applies the "Prediction" color ramp using standard `arcpy.mp` to force Pro to generate the 7 class colors.
   - Dives into the layer's CIM definition to lock the exact float bounds, inject the custom labels, clear conflicting number formats, set the legend to descending order, and apply `Multiply` blend mode.

3. **Phase 3: Map Export** (If `EXPORT_LOG_MAPS = True`)
   - Iterates through the elements, turning each layer on individually.
   - Refreshes the active view.
   - Exports the map to a TIFF file in the `log_maps` subfolder.
   - Turns the layer off and proceeds to the next element.

## How to Run

1. Open **ArcGIS Pro** and load your project.
2. Ensure your input point layer, extent layer, and mask layer are present in the active map.
3. Open the **Python** pane (View -> Python).
4. Paste the entire script into the Python pane, or load it via an external Python IDE configured with the ArcGIS Pro Python environment.
5. Run the script. Check the console output for progress, success messages, or any fields that failed processing.

## Technical Notes

- **Banker's Rounding:** Python's built-in `round()` function uses "half-to-even" rounding (e.g., `round(5.05, 1)` returns `5.0`). This script uses a custom `round_half_up` math function to ensure standard rounding (`5.05` -> `5.1`) for the legend display.
- **CIM API Quirk:** When setting manual classification via the CIM API, `classificationMethod` must be set to `"Manual"`. If `"ManualInterval"` is used, ArcGIS Pro silently defaults back to Defined Interval.
- **Color Preservation:** The script uses `arcpy.mp` to set the color ramp first, which forces Pro to internally generate the hex colors for the 7 classes. The subsequent CIM pass only modifies the bounds and labels, preserving the generated colors perfectly.
