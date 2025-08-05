# Global Historical Climatology Network (GHCN) Analysis with Dask

## Overview

This project leverages **Dask** to process and analyze large-scale climate datasets from the Global Historical Climatology Network (GHCN). The goal is to compute the **maximum daily temperature range (TMAX - TMIN)** for each weather station across multiple datasets using scalable parallel computation.

This work was completed as part of the NYU course **DSGA1004 - Big Data**.

## Contributors

- Kyeongmo Kang
- Nikolas Prasinos
- Alexander Pegot-Ogier

## Dataset Description

The GHCN dataset contains daily climate summaries from stations around the world. Each file contains:

- Station ID, year, month, day
- Climate metric (TMAX, TMIN, PRCP, etc.)
- Measurement values (tenths of °C) and flags

Data source: [https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc\:C00861](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc\:C00861)

### Datasets Used

- `/scratch/.../ghcnd_tiny/` (tiny sample for development)
- `/scratch/.../ghcnd_small/` (small subset)
- `/scratch/.../ghcnd_all/` (full dataset - \~37GB)

## Objective

Compute, for each station, the **largest observed difference between TMAX and TMIN** for any day with valid readings. Return a Dask DataFrame with:

- `station_id`
- `t_range` (maximum temperature range in tenths °C)

## Approach

### ✅ 1. Data Parsing

Used provided `load_daily()` function to convert `.dly` files into Python dictionaries per observation. Each record includes:

- `station_id`, `year`, `month`, `day`
- `element` (e.g. TMAX, TMIN), `value`
- Flag fields: `quality`, `measurement`, `source`

### ✅ 2. Data Cleaning

- Removed observations with `quality` ≠ `' '` (invalid)
- Dropped `value = -9999` (missing)
- Retained only `TMAX` and `TMIN`

### ✅ 3. Daily Max Temperature Range

- Grouped records by station and day
- Joined TMAX and TMIN
- Computed `TMAX - TMIN` and extracted maximum per station
- Saved to Parquet

## Results

| Dataset      | Output File           | Runtime (approx.) | Notes                          |
| ------------ | --------------------- | ----------------- | ------------------------------ |
| ghcnd\_tiny  | `tdiff-tiny.parquet`  | < 10 seconds      | Used for testing logic         |
| ghcnd\_small | `tdiff-small.parquet` | \~ 1–2 minutes    | Production test case           |
| ghcnd\_all   | Not completed         | ✘                 | Requires distributed scheduler |

## How to Run

Run analysis within Dask-enabled JupyterLab or from terminal with local cluster.

```bash
# Setup conda or venv environment and install dependencies
pip install -r requirements.txt

# Launch JupyterLab on Greene or local
jupyter lab

# Run analysis
python ghcn.py  # Adjust path to tiny/small/full directory
```

## Directory Structure

```
the-global-historical-climatology-netwrok-data-analysis-using-dask/
├── Dask starter.ipynb       # Sample intro notebook to Dask API
├── GHCN analysis.ipynb      # Full analysis pipeline in notebook
├── ghcn.py                  # Script version of processing pipeline
├── requirements.txt         # Python package dependencies
├── tdiff-tiny.parquet/      # Output from tiny dataset run
├── tdiff-small.parquet/     # Output from small dataset run
├── mydask.png               # Dask computation graph image
├── README.md                # Project overview (this file)
├── REPORT.md                # Summary of results and answers
└── dask-worker-space/       # Local worker artifacts
```

## Technologies Used

- Python 3.11
- Dask (DataFrame, Delayed)
- pandas, NumPy

## License

For academic and research use only. NYU DSGA1004, Spring 2025.

---

For full results and runtime benchmarks, see [`REPORT.md`](./REPORT.md)

