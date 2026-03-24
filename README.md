# Weather & Energy Correlation Pipeline

**GIK2Q3 Applied Big Data and Cloud Computing_Group B5**
Dalarna University | March 2026

**Team:** Saba · Renuka · Shahed · Aveen · Abeer

---

## Overview

A scalable big data pipeline that analyses the correlation between weather conditions and electricity spot prices across Sweden's four electricity price zones (SE1–SE4). The pipeline ingests 9.19 GB of SMHI weather observations (73 million records) alongside hourly electricity market data, processes them through a Medallion Architecture (Bronze → Silver → Gold), and produces analytical insights on the weather–energy relationship.

**Key findings:** Strong negative correlation of −0.5962 between temperature and spot price, increasing to −0.6467 with a 24-hour rolling average. Spark completed the full pipeline in 589.4 seconds vs Dask's 1,093.4 seconds (1.85× faster).

---

## Project Structure

```
weather_energy_pipeline/
├── README.md                        ← This file
├── requirements.txt                 ← Python dependencies
├── data/
│   ├── README.md                    ← Data download instructions
│   ├── raw/smhi/                    ← Downloaded SMHI CSVs (not in repo)
│   ├── bronze/                      ← Parquet after ingestion (not in repo)
│   ├── silver_spark/                ← Spark-cleaned Parquet (not in repo)
│   ├── silver_dask/                 ← Dask-cleaned Parquet (not in repo)
│   └── gold/                        ← Final analytical dataset (not in repo)
├── notebooks/
│   ├── 01_ingestion-checkpoint.ipynb   ← SMHI API download + Bronze layer
│   ├── 02_spark_processing.ipynb       ← Spark Silver + Gold pipeline
│   ├── 03_dask_processing.ipynb        ← Dask Silver + Gold pipeline
│   └── 04_analysis.ipynb               ← Correlation analysis + visualisations
├── k8s/
│   └── pipeline-job.yaml            ← Kubernetes Job manifest (Minikube)
├── Dockerfile                       ← Container definition for Spark pipeline
├── docs/
│   ├── architecture.png             ← System architecture diagram
│   └── benchmark.json               ← Spark vs Dask performance results
└── aws/
    └── AWS_Cloud_Deployment_Guide.docx  ← Step-by-step AWS deployment guide
```

> **Note:** `data/` is excluded from version control. See `data/README.md` for download instructions. Do not commit raw data, credentials, or generated output files.

---

## Quickstart

### 1. Prerequisites

| Requirement | Version tested |
|---|---|
| Python | 3.10 – 3.12 |
| Java (JDK) | 11 or 17 (required for Spark) |
| Docker | 24+ |
| Minikube | 1.32+ |
| kubectl | 1.28+ |

### 2. Clone the repository

```bash
git clone https://github.com/AbeerAman/weather_energy_pipeline_Project.git
cd weather_energy_pipeline_Project
```

### 3. Create a virtual environment and install dependencies

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Download the data

See [`data/README.md`](data/README.md) for the direct download link and instructions. Place the downloaded files into `data/raw/smhi/` before running any notebooks.

Alternatively, you can run `notebooks/01_ingestion-checkpoint.ipynb` end-to-end to re-download the data directly from the SMHI Open Data API (takes ~90 minutes; rate-limited to 0.3 s per request).
⚠️ IMPORTANT: Always open Jupyter Lab from inside the `notebooks/` folder — NOT from the project root! This ensures all `../data/` paths work correctly.

### 5. Run the pipeline

Execute the notebooks in order:

```
01_ingestion-checkpoint.ipynb   → creates data/bronze/
02_spark_processing.ipynb       → creates data/silver_spark/ and data/gold/
03_dask_processing.ipynb        → creates data/silver_dask/ and data/gold/
04_analysis.ipynb               → reads data/gold/, produces plots and stats
```

Launch Jupyter:

```bash
jupyter notebook
```

---

## Pipeline Architecture

The pipeline follows the **Medallion Architecture** pattern:

| Layer | Description | Tool |
|---|---|---|
| **Bronze** | Raw ingestion — 6,781 SMHI CSV files → Parquet (0.45 GB, 73M rows) | Pandas + fastparquet |
| **Silver** | Cleaning, validation, timestamp parsing, price-zone enrichment | Spark / Dask |
| **Gold** | Hourly aggregation, weather–energy join, 24h rolling window | Spark / Dask |
| **Analysis** | Correlation analysis, seasonal breakdowns, visualisations | Pandas + matplotlib |

The pipeline is implemented **twice** — once in Apache Spark (`02_spark_processing.ipynb`) and once in Dask (`03_dask_processing.ipynb`) — enabling a direct performance comparison.

---

## Data Sources

### SMHI Weather Data

- **Source:** [SMHI Open Data API](https://opendata.smhi.se/apidocs/metobs/index.html)
- **Parameter:** Air temperature (parameter 1), corrected-archive period
- **Coverage:** 995 weather stations across all of Sweden, January 2019 – December 2024
- **Volume:** 9.19 GB (6,781 CSV files), 73,355,669 records
- **License:** Creative Commons CC0 (public domain)

Station latitude is used to assign each measurement to a Swedish electricity price zone:

| Zone | Latitude range | Region |
|---|---|---|
| SE1 | ≥ 66.0° | Northern Sweden |
| SE2 | 61.0° – 66.0° | North-central |
| SE3 | 56.5° – 61.0° | South-central (incl. Stockholm) |
| SE4 | < 56.5° | Southern Sweden (incl. Malmö) |

### Energy Market Data

- **Source:** Synthetic dataset based on published characteristics of the Nord Pool Swedish market
- **Coverage:** January 2019 – December 2024, hourly, all four price zones
- **Volume:** ~50 MB, 210,340 records
- **Disclosure:** Real bulk historical data from Nord Pool requires commercial registration and was not available. The synthetic dataset reproduces seasonal price variation, hour-of-day patterns, and zone-specific offsets consistent with published market data. This is disclosed in full in the technical report.

---

## Framework Comparison (Spark vs Dask)

| Metric | Apache Spark | Dask |
|---|---|---|
| Total pipeline time | **589.4 s** | 1,093.4 s |
| Lines of code | ~120 | ~115 |
| Setup complexity | High — JVM + cluster config | Low — pure Python, `pip install` |
| Fault tolerance | Built-in (RDD lineage + checkpointing) | Limited (task graph retry only) |
| Recommendation | **Preferred at 10 GB+ scale** | Suitable for smaller workloads |

Spark is 1.85× faster for this workload due to its optimised shuffle engine and in-memory execution model. Full benchmark data is in `docs/benchmark.json`.

---

## Quality and Operations

The pipeline meets the VG Quality and Operations requirements:

- **Structured logging** — all stages use Python's `logging` module with named loggers (not `print` statements), producing timestamped execution traces.
- **Data validation** — null checks, schema enforcement, and value range assertions (−60°C to 45°C) applied at the Bronze layer before any processing.
- **Basic monitoring** — execution time tracked with `time.time()` for every stage; record counts logged at Bronze, Silver, and Gold layers.
- **Edge case handling** — malformed SMHI CSV files (~50–80 out of 995 stations) detected and skipped with warning log messages.

---

## Deployment

### Local Kubernetes (Minikube)

The Spark pipeline is containerised and deployed as a Kubernetes Job on a local Minikube cluster.

```bash
# 1. Build the Docker image
docker build -t weather-energy-pipeline:latest .

# 2. Start Minikube
minikube start --memory=8192 --cpus=4

# 3. Load the image into Minikube
minikube image load weather-energy-pipeline:latest

# 4. Deploy the Kubernetes Job
kubectl apply -f k8s/pipeline-job.yaml

# 5. Monitor the job
kubectl get pods
kubectl logs job/weather-energy-pipeline -f
```

### AWS Cloud Deployment (VG)

The pipeline was also deployed to AWS (eu-north-1, Stockholm region) using free tier resources. See [`aws/AWS_Cloud_Deployment_Guide.docx`](aws/AWS_Cloud_Deployment_Guide.docx) for the full step-by-step guide with screenshots.

**AWS resources used:**

| Component | Service | Detail |
|---|---|---|
| Data storage | S3 | Bucket: `weather-energy-b5data` |
| Compute | EC2 | `t3.micro`, Amazon Linux 2023 |
| Region | eu-north-1 | Europe (Stockholm) |
| Data uploaded | — | 433 MB Bronze + 5.8 MB Gold |

**Cloud-ready architecture (GCP):** The pipeline is also designed to scale to Google Cloud Platform with minimal code changes — GKE Autopilot for orchestration, Cloud Storage for Parquet data, and Cloud Build for CI/CD. See Section 3.5 of the technical report for the full architecture diagram and migration path.

> **Security reminder:** Never commit AWS Access Keys, Secret Keys, or any credentials to version control. Use environment variables or AWS IAM roles.

---

## Results Summary

| Metric | Value |
|---|---|
| Temperature–price correlation (hourly) | −0.5962 |
| Temperature–price correlation (24h rolling avg) | −0.6467 |
| Records processed (weather) | 73,355,669 |
| Records after Silver cleaning | 72,176,442 (98.4% retained) |
| Final Gold dataset records | 210,316 |
| Parquet compression ratio | ~14:1 (9.19 GB → 0.45 GB) |

The analysis confirms that as temperature falls, electricity spot prices rise significantly across all four Swedish price zones. SE4 (south) shows the weakest correlation; SE1 (north) the strongest.

---

## Requirements

Key dependencies (`requirements.txt`):

```
pyspark>=3.5
dask[distributed]>=2024.1
pandas>=2.0
fastparquet>=2023.10
pyarrow>=14.0
requests>=2.31
matplotlib>=3.8
seaborn>=0.13
jupyter>=1.0
```

> Install with: `pip install -r requirements.txt`

---

## Academic Integrity

All code is the original work of Group B5. External resources are cited in the technical report. AI tools (Claude ai, GitHub Copilot) were used for code assistance and are disclosed in the report declaration as required by course policy.

---

## Contact

Dalarna University | GIK2Q3 Applied Big Data and Cloud Computing | Spring 2026
