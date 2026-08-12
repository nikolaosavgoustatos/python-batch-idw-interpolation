# ArcGIS Pro Batch IDW & Log-Scaled Symbology

This Python script automates Inverse Distance Weighting (IDW) interpolation for multiple geochemical elements and applies highly customized, log-scaled classified symbology. It is designed to run inside an active ArcGIS Pro project and leverages both the `arcpy.mp` API and direct CIM (Cartographic Information Model) manipulation to bypass standard UI limitations.

## Key Features

- **Batch IDW Interpolation**: Processes multiple trace elements (As, Ba, Ca, Cr, Cu, Fe, Mn, Ni, Pb, Rb, Sr, Ti, V, Zn) from a single input point feature class.
- **Log-Scaled Classification**: Generates classification breaks evenly spaced in `log10` space, then back-transforms them (`10**x`) so the map rendering follows a log distribution while the legend labels remain in true ppm.
- **Custom Contiguous Labels**: Generates non-overlapping range labels rounded *UP* to 1 decimal place (e.g., `"4.0 - 5.1"`, `"5.2 - 6.8"`). Includes floating-point noise absorption so values like `4.0001` correctly become `4.0` instead of `4.1`.
- **Color Ramp Preservation**: Applies the "Prediction" color ramp from the ArcGIS Colors style. Uses a hybrid `arcpy.mp` + CIM approach to ensure Pro generates the colors internally and preserves them without falling back to flat black/red.
- **Layer Blend Mode**: Locks the layer blend mode to **Multiply** directly in the CIM definition so it persists reliably.
- **Descending Legend**: Sets the legend order so the highest concentrations appear at the top.
- **Automated Map Export**: Optional phase to toggle each layer visible, export a high-resolution TIFF, and hide it again.

## Requirements

- **ArcGIS Pro** (3.x recommended)
- **Spatial Analyst** extension license
- **Python 3.x** (comes bundled with ArcGIS Pro)
- **NumPy** (comes bundled with ArcGIS Pro)
- An active ArcGIS Pro project with the target map open.

## Setup & Configuration

Open the script in the ArcGIS Pro Python editor (or your preferred IDE) and modify the parameters in the **Inputs and parameters** section at the top of the script:

```python
IN_POINTS = "As mg/kg"               # Name of the input point layer
Z_FIELDS = ["As", "Ba", "Ca", ...]   # List of z-value fields to interpolate
OUT_FOLDER = r"C:\Path\To\Output"    # Directory to save rasters and maps

OUT_CELL_SIZE = 50
POWER = 2
SEARCH_RADIUS = arcpy.sa.RadiusVariable(12, None)

NUM_CLASSES = 7
COLOR_RAMP_NAME = "Prediction"       # Must exist in project styles

RUN_IDW = True                       # Set to False to only re-apply symbology
EXPORT_LOG_MAPS = False              # Set to True to export TIFFs
```

## How to Run

1. Open your ArcGIS Pro project.
2. Ensure the input point layer (`IN_POINTS`) and boundary masks (if referenced in `env_kwargs`) are loaded in the active map.
3. Open the Python pane (`View` -> `Python`) or run the script via the ArcGIS Pro Python environment.
4. Execute the script. 

*Note: Because the script uses `arcpy.mp.ArcGISProject("CURRENT")`, it must be run from within an open ArcGIS Pro session.*

## Technical Notes & CIM Workarounds

ArcGIS Pro's standard `arcpy.mp` symbology API has several known quirks that this script deliberately bypasses using the CIM (Cartographic Information Model):

1. **Manual Interval Enum String**: `arcpy.mp` uses `"ManualInterval"`, but the CIM API requires the exact string `"Manual"`. Using the wrong string causes Pro to silently fall back to *Defined Interval* or *Natural Breaks*. The script handles this correctly in the CIM pass.
2. **Stale `numberFormat` Property**: Mutating an existing CIM colorizer often leaves a stale `numberFormat` object that causes Pro to discard custom range labels and auto-generate bare numbers. This script explicitly clears `numberFormat` and `deviationInterval`.
3. **Color Ramp Sampling Bugs**: `MappingColorRampObject` in some Pro versions does not support `readColors()` and will throw an error. This script allows `arcpy.mp` to apply the ramp and generate the colors internally, then mutates the bounds/labels while *preserving* the generated colors.
4. **Layer Blend Mode**: The CIM property is `"blendingMode"` (not `"blendMode"`). Setting the wrong property silently does nothing. The script locks `Multiply` directly in the CIM XML.

## Output

- **Rasters**: Saves IDW interpolated rasters as `.tif` files in the specified `OUT_FOLDER`.
- **Map Layers**: Adds the rasters to the active map with the log-scaled symbology, custom labels, descending legend, and Multiply blend mode applied.
- **Exports** (Optional): If `EXPORT_LOG_MAPS = True`, high-resolution TIFFs for each element are saved to the `log_maps` subfolder.
