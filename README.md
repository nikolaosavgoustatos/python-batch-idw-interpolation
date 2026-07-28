# ArcGIS Pro Batch IDW Interpolation with Percentile Symbology

A Python script for **ArcGIS Pro** that automates **Inverse Distance Weighting (IDW)** interpolation for multiple geochemical elements while applying consistent, publication-quality classified symbology.

The script performs batch interpolation, calculates percentile-based class breaks, and configures raster symbology so that ArcGIS Pro displays **Manual Interval** classification with the **Red–Yellow–Blue (7 Classes)** color ramp.

---

## Features

* Batch IDW interpolation for multiple fields
* Automatic percentile-based classification
* Seven-class color scheme
* Exact (unrounded) classification values
* Rounded integer labels for map legends
* Automatic raster statistics calculation
* Automatic raster loading into the active map
* Manual Interval classification preserved in ArcGIS Pro
* Automatic correction if ArcGIS reverts raster symbology
* Saves class breaks to JSON for reproducible results
* Supports running interpolation or applying symbology only

---

## Classification Scheme

The script creates **7 classes** using the following breakpoints:

Minimum → 5th → 25th → 50th → 75th → 90th → 95th → Maximum

Important:

* Actual classification values remain exact floating-point numbers.
* Only the displayed labels are rounded using normal "round half up" rounding.
* This preserves numerical accuracy while producing clean map legends.

---

## Color Ramp

The script uses the ArcGIS Pro color ramp:

**Red–Yellow–Blue (7 Classes)**

with colors reversed so that

* High values → Red
* Low values → Blue

---

## Supported Elements

By default the script interpolates:

* As
* Ba
* Ca
* Cr
* Cu
* Fe
* Mn
* Ni
* Pb
* Rb
* Sr
* Ti
* V
* Zn

Additional fields can easily be added by modifying the `Z_FIELDS` list.

---

## Requirements

* ArcGIS Pro
* Spatial Analyst Extension
* Python environment included with ArcGIS Pro
* NumPy
* arcpy

---

## Input

Point feature class containing geochemical measurements.

Each selected field should contain numeric concentration values.

---

## Output

For every element the script generates:

* IDW raster (.tif)
* Classified raster layer
* Manual Interval symbology
* Rounded legend labels
* Stored class breaks in `idw_breaks.json`

---

## Usage

### Run interpolation and symbology

```python
RUN_IDW = True
```

### Apply symbology only to existing rasters

```python
RUN_IDW = False
```

---

## How It Works

1. Reads point measurements.
2. Computes percentile break values.
3. Runs IDW interpolation.
4. Calculates raster statistics.
5. Adds rasters to the active ArcGIS Pro map.
6. Applies Manual Interval classified symbology.
7. Locks classification using the ArcGIS CIM.
8. Performs a second verification pass to correct any reverted layers.

---

## Key Technical Details

The script addresses several ArcGIS Pro symbology behaviors, including:

* Difference between `ManualInterval` (arcpy.mp) and `Manual` (CIM)
* Explicit class label assignment
* Persistent manual class breaks
* Automatic recovery if ArcGIS reverts layers to Stretch or Standard Deviation rendering
* Consistent symbology across multiple rasters

---

## Example Applications

* Geochemical mapping
* Environmental contamination studies
* Soil chemistry visualization
* Mineral exploration
* Heavy metal distribution mapping
* Environmental baseline surveys

---

## License

MIT License

---

## Author

Nikolaos Avgoustatos
