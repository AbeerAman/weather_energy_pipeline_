# Data

This folder is intentionally empty in the repository. Do not commit raw data files.

---

## How to get the data
cannot push data in github because it was more than 25 MB. So, zip file is attached for data.

### 1. SMHI Weather Data (9.10 GB)

The weather data is downloaded automatically by running the ingestion notebook.

**Option A — Run the notebook (recommended)**

Open and run `notebooks/01_ingestion-checkpoint.ipynb` from start to finish.

- It will connect to the SMHI Open Data API automatically
- Downloads air temperature records from 995 weather stations across Sweden
- Covers January 2019 – December 2024
- Takes approximately 90 minutes (rate-limited to 0.3s per request)
- Saves files to `data/raw/smhi/`

**Option B — Manual download**

If you prefer to download manually:

- Go to: https://opendata.smhi.se/apidocs/metobs/index.html
- Parameter: 1 (Air temperature)
- Period: corrected-archive
- Download CSV files for all stations and place them in `data/raw/smhi/`

---

### 2. Energy Market Data (12.8 MB)

The energy dataset is synthetic and generated automatically by the pipeline.

It is created when you run `notebooks/01_ingestion-checkpoint.ipynb`.

No manual download is needed.

---

## Expected folder structure after running the pipeline

```
data/
├── raw/
│   └── smhi/               ← 6,756 CSV files from SMHI API
├── bronze/
│   ├── weather_raw.parquet ← 0.45 GB, 73M records
│   └── energy_raw.parquet  ← 12.8 MB, 210,340 records
├── silver_spark/
│   └── weather_clean/      ← Spark-cleaned Parquet
├── silver_dask/
│   └── weather_clean/      ← Dask-cleaned Parquet
└── gold/
    └── gold_weather_energy.parquet ← Final analytical dataset
```

---

## Data sources

| Source | URL | License |
|---|---|---|
| SMHI Open Data API | https://opendata.smhi.se/apidocs/metobs/index.html | CC0 (public domain) |
| Nord Pool (synthetic) | https://www.nordpoolgroup.com | Synthetic — see report Section 2.3 |
