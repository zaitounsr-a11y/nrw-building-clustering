# NRW Residential Building Stock — Clustering Analysis


## What is in this folder

This folder contains everything needed to reproduce the clustering analysis of 4.1 million
residential buildings in NRW (North Rhine-Westphalia, Germany).

The goal is to reduce 4.1 million buildings into representative archetypes (clusters)
that can be fed into the SESMG energy optimization model.

---

## Files

### Data files
| File | Size | Description |
|------|------|-------------|
| `DEA_method1_final.parquet` | ~1 GB | 4.1M buildings with electricity (Method 1), heat, and PV demand |
| `DEA_method3b_final.parquet` | ~1 GB | Same buildings with electricity calculated via CO2online lookup (Method 3b) |
| `lod2_nrw_roofs.geoparquet` | ~170 MB | 3D LOD2 roof dataset — south, east, west, north areas per building |
| `final_tabula.xlsx` | ~13 MB | TABULA heat demand lookup table by building type and refurbishment state |

> **Note on DEA_combined.parquet:**
> This is the raw OSM building dataset (~10 GB) required only by the two pipeline notebooks
> (`pipeline_method1.ipynb` and `pipeline_method3b.ipynb`).
> If you only want to run the clustering notebooks, you do not need it —
> the pre-built parquets (`DEA_method1_final.parquet` and `DEA_method3b_final.parquet`) are already included.

### Pipeline notebooks (how the parquets were built)
| Notebook | Description |
|----------|-------------|
| `pipeline_method1.ipynb` | Builds `DEA_method1_final.parquet` from raw OSM data |
| `pipeline_method3b.ipynb` | Builds `DEA_method3b_final.parquet` from raw OSM data |

> **Note:** You do not need to run these. The parquets are already built and included.
> These notebooks are provided for transparency — they show exactly how the demand values were calculated.

### Clustering notebooks (the main analysis)
| Notebook | Description |
|----------|-------------|
| `clustering_m1_final.ipynb` | K-Means clustering per building type using Method 1 → **292 combinations** |
| `clustering_m3b_by_type.ipynb` | K-Means clustering per building type using Method 3b → **267 combinations** |
| `roof_clustering_by_type.ipynb` | Roof area clustering from LOD2 dataset → **89 combinations** |
| `silhouette_comparison_m1_vs_m3b.ipynb` | Compares silhouette behaviour between Method 1 and Method 3b |

---

## How to open and run the notebooks

### Requirements
You need Python 3.9+ with these packages installed:
```
pip install pandas geopandas numpy scikit-learn matplotlib openpyxl pyarrow
```

### Important — how to open VS Code correctly
All notebooks use `os.getcwd()` to find files automatically.
This only works if VS Code is opened **from inside the folder** — not by opening individual files.

If you open a notebook directly without opening the folder first,
the paths will not resolve and you will get a `FileNotFoundError`.

---

## Recommended order to run

1. **`clustering_m1_final.ipynb`** — start here, this is the main result
2. **`clustering_m3b_by_type.ipynb`** — comparison method
3. **`roof_clustering_by_type.ipynb`** — roof area clustering
4. **`silhouette_comparison_m1_vs_m3b.ipynb`** — method comparison figure

Each notebook saves its outputs to a `clustering_results/` subfolder
that is created automatically in the same folder.

---

## Key results summary


### Clustering combinations
| Variable | Method 1 | Method 3b |
|----------|----------|-----------|
| Per building type | 292 | 267 |
| Whole dataset | 64 | 64 |

### Roof area clustering
| Building type | Gabled combos | Flat combos | Total |
|---------------|:---:|:---:|:---:|
| SFH | 4×4=16 | 4 | 20 |
| TH | 5×4=20 | 4 | 24 |
| MFH | 4×4=16 | 4 | 20 |
| AB | 5×4=20 | 5 | 25 |
| **Total** | | | **89** |

### Total archetypes for SESMG
```
M1 combinations × Roof combinations = 292 × 89 = 25,988 archetypes
M3 combinations × Roof combinations = 267 × 89 = 23,763 archetypes

---

## CO2online lookup table (Method 3b)
Values used: **without electric water heating** (SESMG optimizes DHW separately)  
Source: CO2online Stromspiegel via Statista, Mai 2025

| Persons | SFH/TH (kWh/yr) | MFH/AB (kWh/yr) | Source |
|---------|:-:|:-:|--------|
| 1 | 1,800 | 1,200 | Statista Mai 2025 |
| 2 | 2,700 | 1,900 | Statista Mai 2025 |
| 3 | 3,500 | 2,400 | Statista Mai 2025 |
| 4 | 4,000 | 2,900 | Statista April 2023 |
| 5 | 4,500 | 3,100 | Statista Mai 2025 |

