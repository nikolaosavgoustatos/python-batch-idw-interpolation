# ArcGIS Pro Batch IDW Interpolation & Percentile Symbology

This Python script automates the workflow of generating Inverse Distance Weighting (IDW) interpolation surfaces from point data in ArcGIS Pro. It applies a highly customized, 7-class percentile-based symbology using the "Prediction" color ramp, and ensures that the classification break values and legend labels are mathematically consistent using standard half-up rounding.

## Key Features

- **Batch IDW Processing:** Iterates through multiple element fields (e.g., As, Ba, Ca, Cr) to generate continuous prediction rasters.
- **Percentile-Based Classification:** Generates exactly 7 classes based on the data distribution: `Min -> 5th -> 25th -> 50th -> 75th -> 90th -> 95th -> Max`.
- **Consistent Rounding Logic:** Uses standard half-up rounding (`round_half_up`) for **both** the actual CIM classification bounds and the legend labels. This prevents visual mismatches where a label says `5.0` but the underlying break is `4.98`.
- **Contiguous Legend Labels:** Formats labels as non-overlapping `(x, y]` intervals with exactly 1 decimal place (e.g., `4.2 - 5.0`, `5.1 - 6.7`). Legend displays in descending order.
- **Color Ramp Application:** Pulls the "Prediction" color ramp from the project's ArcGIS Colors style and uses a hybrid `arcpy.mp` + CIM approach to ensure Pro generates the colors correctly while locking in custom bounds and labels.
- **Layer Blend Mode:** Automatically sets the layer blend mode to `Multiply` via the CIM API to guarantee it sticks.
- **Optional Map Export:** Can automatically toggle layer visibility and export high-resolution TIFFs for each element.

## Prerequisites

1. **ArcGIS Pro:** This script relies on `arcpy` and must be run inside an open ArcGIS Pro project.
2. **Spatial Analyst Extension:** Required for the IDW interpolation. The script checks out the extension automatically.
3. **Input Data:** A point feature class containing the fields to interpolate.
4. **Project Styles:** The "Prediction" color ramp must be available in the project's styles (it is included by default in ArcGIS Pro).

## Installation & Usage

1. Download or clone this repository.
2. Open your ArcGIS Pro project.
3. Open the **Python** pane or a Jupyter Notebook within ArcGIS Pro.
4. Run the script.

### Configuration

Before running, modify the configuration block at the top of the script to match your environment:

```python
# Path to your input point feature class
IN_POINTS = "Your_Point_Layer_Name"

# List of fields to interpolate
Z_FIELDS = ["As", "Ba", "Ca", "Cr", "Cu", "Fe", "Mn", "Ni", "Pb", "Rb", "Sr", "Ti", "V", "Zn"]

# Output folder for the generated rasters
OUT_FOLDER = r"C:\Path\To\Your\Output\Folder"

# IDW Parameters
OUT_CELL_SIZE = 50
POWER = 2
SEARCH_RADIUS = arcpy.sa.RadiusVariable(12, None)

# Toggle phases
RUN_IDW = True               # Set to False to only fix symbology on existing layers
EXPORT_LOG_MAPS = False      # Set to True to also export per-element TIFFs
```

> **Note on Environment Variables:** The script includes specific environment settings (`extent`, `mask`, `cellSizeProjectionMethod`) hardcoded to specific layer names (e.g., `andros_gys`, `andros_municipality_community`). If your project uses different names for your extent/mask layers, update the `env_kwargs` dictionary in the `main()` function.

## How It Works

### The Rounding Problem
Python's built-in `round()` function uses banker's rounding (half-to-even), meaning `round(5.05, 1)` returns `5.0`. ArcGIS Pro's default UI rounding can also sometimes cause mismatches between displayed labels and actual data bounds.

This script implements a custom `round_half_up()` function to ensure that `.05` always rounds up. 

### Classification & Label Generation
1. The script reads the point data using `numpy` to calculate exact percentile breaks.
2. The breaks are rounded using `round_half_up()`.
3. Labels are generated as contiguous ranges. The lower bound of a class is always `previous_upper_bound + 0.1` to ensure no overlapping ranges.

### Symbology Application
Applying raster symbology via Python in ArcGIS Pro can be notoriously finicky. This script uses a two-step approach:
1. **`arcpy.mp` API:** Sets the colorizer to `RasterClassifyColorizer`, applies the "Prediction" color ramp, and sets the break count. This forces Pro to internally generate the 7 colors.
2. **CIM API:** Accesses the layer's definition (`V2`) to forcefully lock the `upperBound` values, `minimumBreak`, custom labels, and the `Multiply` blend mode—while preserving the colors Pro generated in step 1.
