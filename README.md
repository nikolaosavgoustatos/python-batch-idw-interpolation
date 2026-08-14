# ArcGIS Pro Batch IDW Interpolation & Percentile Symbology

This Python script automates the workflow for running Inverse Distance Weighted (IDW) interpolation on multiple geochemical elements and applying highly specific, standardized classified symbology in ArcGIS Pro. 

It is designed for geochemical or environmental spatial analysis where consistent map presentation across multiple variables (e.g., heavy metals in soil) is critical.

## Key Features

- **Batch IDW Processing**: Runs IDW interpolation across 14 elements (As, Ba, Ca, Cr, Cu, Fe, Mn, Ni, Pb, Rb, Sr, Ti, V, Zn) with customizable cell size, power, and search radius.
- **Percentile-Based Classification**: Automatically calculates 7 classes based on exact sample point percentiles:
  - `Min` → `5th` → `25th` → `50th` → `75th` → `90th` → `95th` → `Max`
- **Consistent Half-Up Rounding**: Overrides Python's default banker's rounding to ensure that classification break *values* and display *labels* are mathematically identical and visually consistent.
- **Custom Layer & Field Naming**: Automatically names map layers and symbology fields with units (e.g., `As mg/kg` instead of `IDW_As` and `Value`).
- **Advanced CIM Manipulation**: Bypasses standard `arcpy.mp` limitations by dropping into the CIM (Cartographic Information Model) API to lock class breaks, apply the "Prediction" color ramp, enforce descending legend order, and guarantee the `Multiply` layer blend mode.
- **Map Exporting**: Optional automated export of individual element maps to high-resolution TIFFs.

## Prerequisites

1. **ArcGIS Pro**: Must be installed and licensed (requires Spatial Analyst extension for IDW).
2. **Active Project**: The script must be run inside an open ArcGIS Pro project (`arcpy.mp.ArcGISProject("CURRENT")`).
3. **Input Data**: A point feature class containing the element fields (e.g., `As`, `Ba`, etc.).
4. **Color Ramp**: The "Prediction" color ramp must be available in the project's styles (it is included in the default "ArcGIS Colors" style).
5. **Python Environment**: Uses the default ArcGIS Pro Python environment (`arcgispro-py3`), which includes `numpy`.

## Configuration

Before running the script, open it in the ArcGIS Pro Python editor (or your preferred IDE) and modify the parameters in the **Inputs and parameters** section:

```python
IN_POINTS = "As mg/kg"               # Name of your input point feature layer
Z_FIELDS = ["As", "Ba", "Ca", ...]   # List of fields to interpolate
OUT_FOLDER = r"C:\Path\To\Output"    # Where raster TIFFs will be saved
OUT_CELL_SIZE = 50                   # Output raster cell size
POWER = 2                            # IDW power parameter
SEARCH_RADIUS = arcpy.sa.RadiusVariable(12, None) # 12 nearest points

# Run Options
RUN_IDW = True                       # Set to False to only fix symbology on existing layers
EXPORT_LOG_MAPS = False              # Set to True to export TIFFs
```

## Usage

Because this script interacts with the `CURRENT` ArcGIS Pro project, you should run it from inside ArcGIS Pro:

1. Open your ArcGIS Pro project.
2. Ensure your input points and extent/mask layers (e.g., `andros_municipality_community`) are loaded in the active map.
3. Open the **Python** pane (`View` -> `Python`).
4. Load the script into the pane or run it directly from a `.py` file using the `import` command or the "Run Script" button.
5. Monitor the progress in the Python console. The script will:
   - **Phase 1**: Run IDW and save rasters to `OUT_FOLDER`.
   - **Phase 2**: Add rasters to the map, apply the percentile symbology, and rename layers (e.g., `As mg/kg`).
   - **Phase 3** *(Optional)*: Export individual maps to the `log_maps` subfolder.

## How the Symbology Works

ArcGIS Pro's standard `arcpy.mp` symbology API can be finicky when it comes to manual class breaks. This script uses a two-step approach to guarantee perfect results:

1. **`arcpy.mp` Pass**: Applies the "Prediction" color ramp and sets the classification method to `ManualInterval` so Pro internally generates the 7 distinct colors.
2. **`arcpy.cim` Pass**: Accesses the layer's CIM definition to overwrite the break values and labels with precise, half-up rounded numbers. It also sets the legend to descending order (highest values at the top) and applies the `Multiply` blending mode.

### Label Formatting Logic
Labels are formatted as contiguous, non-overlapping ranges `(x, y]`:
- First lower bound: `round_half_up(min)`
- Subsequent lower bounds: `round_half_up(previous_break) + 0.1`
- Upper bounds: `round_half_up(exact_break)`

**Example Output:**
```text
32.3 - 39.0
26.3 - 32.2
17.6 - 26.2
10.4 - 17.5
7.6 - 10.3
5.9 - 7.5
5.4 - 5.8
```
