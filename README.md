# NRW Residential Building Stock — Clustering Analysis

## Overview

This repository contains the clustering analysis of 4.1 million residential buildings
in NRW (North Rhine-Westphalia, Germany).

The goal is to reduce 4.1 million buildings into representative archetypes (clusters)
that can be fed into the SESMG energy optimization model.

---

## ⚠️ Before You Start — Data Files Required

The notebooks require large data files that are **not included in this repository** due to size.
You need to obtain these separately and place them in the **same folder as the notebooks**:

| File | Size | Required by |
|------|------|-------------|
| `DEA_method3b_final.parquet` | ~1 GB | All clustering notebooks |
| `lod2_nrw_roofs.geoparquet` | ~170 MB | `roof_clustering_by_type.ipynb` |
| `wohnflaechen_nrw.parquet` | ~200 MB | `residential_filter_and_lawn_area.ipynb` |
| `final_tabula.xlsx` | ~13 MB | `pipeline_method3b.ipynb` |
| `DEA_combined.parquet` | ~718 MB | `pipeline_method3b.ipynb` only (optional — see note below) |

> **Note on DEA_combined.parquet:**
> Only needed if you want to rebuild `DEA_method3b_final.parquet` from scratch.
> If you already have `DEA_method3b_final.parquet` you can skip this file
> and skip `pipeline_method3b.ipynb` entirely.

---

## How to Open and Run

### Requirements
Install the required Python packages:
```
pip install pandas geopandas numpy scikit-learn matplotlib openpyxl pyarrow demandlib shapely
```

### ⚠️ Important — Open VS Code from Inside the Folder
All notebooks use `os.getcwd()` to find data files automatically.
This only works if VS Code is opened **from inside the folder**.

**Correct:** Right-click the folder → **Open with Code**
**Wrong:** Dragging a single `.ipynb` file into VS Code

If you open a notebook directly, the paths will not resolve and you will get a `FileNotFoundError`.

---

## Notebooks

### Pipeline (optional — only if rebuilding from scratch)
| Notebook | Description |
|----------|-------------|
| `pipeline_method3b.ipynb` | Builds `DEA_method3b_final.parquet` from raw `DEA_combined.parquet` |

### Main Analysis
| Notebook | Description |
|----------|-------------|
| `clustering_m3b_by_type.ipynb` | K-Means clustering per building type using Method 3b |
| `roof_clustering_by_type.ipynb` | Roof area clustering from LOD2 dataset |
| `outlier_threshold_comparison.ipynb` | Comparison of P95 vs P99.5 outlier removal |

### Geothermal Analysis
| Notebook | Description |
|----------|-------------|
| `geothermal_lookup_table.ipynb` | Borehole lookup table (heat power → boreholes → area) |
| `annual_to_peak_heat_power.ipynb` | Converts annual heat demand to peak power using BDEW profiles |
| `residential_filter_and_lawn_area.ipynb` | Residential filter + lawn area + Boolean geothermal flag |

---

## Recommended Run Order

```
1. clustering_m3b_by_type.ipynb
2. roof_clustering_by_type.ipynb
3. geothermal_lookup_table.ipynb
4. annual_to_peak_heat_power.ipynb
5. residential_filter_and_lawn_area.ipynb
6. outlier_threshold_comparison.ipynb   ← optional, for reference
```

Each notebook saves outputs to a `clustering_results/` subfolder created automatically.

---

## Key Results

### Method 3b Clustering — Per Building Type (P95 outlier removal)

| Building type | k Electricity | k Heat | k PV | Combinations |
|---------------|:---:|:---:|:---:|:---:|
| SFH | 2 (hardcoded) | 4 | 4 | 32 |
| TH  | 2 (hardcoded) | 4 | 4 | 32 |
| MFH | 4 | 5 | 4 | 80 |
| AB  | 4 | 4 | 4 | 64 |
| **Total** | | | | **208** |

> SFH and TH electricity is hardcoded to k=2 — only 2 possible values exist
> from the CO2online table (2,700 kWh and 3,500 kWh) since these building types
> always have exactly 1 dwelling.

### Roof Area Clustering (LOD2 dataset)

| Building type | Gabled | Flat | Total |
|---------------|:---:|:---:|:---:|
| SFH | 4×4=16 | 4 | 20 |
| TH  | 5×4=20 | 4 | 24 |
| MFH | 4×4=16 | 4 | 20 |
| AB  | 5×4=20 | 5 | 25 |
| **Total** | | | **89** |

### Total Archetypes for SESMG
```
M3b combinations × Roof combinations = 208 × 89 = 18,512 archetypes
```

### Geothermal Feasibility

| Building type | Feasible | Total matched | % feasible |
|---------------|:---:|:---:|:---:|
| SFH | 2,461,069 | 2,485,862 | 99.0% |
| TH  | 471,733   | 492,569   | 95.8% |
| MFH | 677,025   | 724,106   | 93.5% |
| AB  | 4,058     | 4,300     | 94.4% |
| **Overall** | **3,613,885** | **3,706,837** | **97.5%** |

Residential buildings matched to parcel: **3,705,834 (89.7%)**
Excluded (hotels, garages, industrial, mixed use): **427,489 (10.3%)**

---

## CO2online Lookup Table (Method 3b)
Values used: **without electric water heating** (SESMG optimizes DHW separately)
Source: CO2online Stromspiegel via Statista, Mai 2025

| Persons | SFH/TH (kWh/yr) | MFH/AB (kWh/yr) |
|---------|:-:|:-:|
| 1 | 1,800 | 1,200 |
| 2 | 2,700 | 1,900 |
| 3 | 3,500 | 2,400 |
| 4 | 4,000 | 2,900 |
| 5 | 4,500 | 3,100 |

---

## Outlier Removal — P95

The top 5% of buildings by footprint area are removed per building type before clustering.
This removes misclassified industrial and farm buildings that distort cluster means.

| Threshold | Buildings removed | SFH threshold | AB threshold |
|-----------|:-:|:-:|:-:|
| P99.5 (old) | 20,669 (0.5%) | 441 m² | 2,199 m² |
| P95 (current) | 206,668 (5.0%) | 217 m² | 940 m² |
