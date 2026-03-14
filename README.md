# Weather and Energy Pipeline Project

## GIK2Q3 Applied Big Data and Cloud Computing
**Grade Target:** VG (Distinction)  
**Deadline:** March 24, 2026

---

## Project Description
A scalable big data pipeline that analyses the correlation 
between weather and electricity prices across Sweden's four 
price zones (SE1-SE4) using Apache Spark and Dask.

---

## Team Members
- Saba — Data Engineer (Ingestion + Bronze Layer)
- Renuka — Spark Developer (Silver + Gold Layer)
- Shahed — Dask Developer (Dask Pipeline + Benchmark)
- Aveen — DevOps Engineer (Docker + Kubernetes)
- Abeer — Analyst and Reporter (Charts + Report + Presentation)

---

## Data Sources
- SMHI Open Data API (weather observations across Sweden)
- Nordpool (hourly electricity spot prices SE1-SE4)

---

## How to Run the Project

### Step 1 - Clone the repository
git clone https://github.com/AbeerAman/weather_energy_pipeline_.git
cd weather_energy_pipeline_

### Step 2 - Install requirements
pip install -r requirements.txt

### Step 3 - Download the data
See data/README.md for download instructions

### Step 4 - Run the notebooks in order
1. notebooks/01_ingestion.ipynb      (Saba)
2. notebooks/02_spark_processing.ipynb  (Renuka)
3. notebooks/03_dask_processing.ipynb   (Shahed)
4. notebooks/04_analysis.ipynb          (Abeer)

---

## Technologies Used
- Apache Spark (PySpark)
- Dask
- Docker
- Kubernetes (Minikube)
- Python 3.10
- Pandas, PyArrow, Matplotlib, Seaborn

---

## Project Structure
weather_energy_pipeline/
├── README.md
├── requirements.txt
├── data/
│   └── README.md
├── notebooks/
│   ├── 01_ingestion.ipynb
│   ├── 02_spark_processing.ipynb
│   ├── 03_dask_processing.ipynb
│   └── 04_analysis.ipynb
├── src/
│   ├── ingest.py
│   ├── spark_pipeline.py
│   └── dask_pipeline.py
├── k8s/
│   ├── Dockerfile
│   └── pipeline-job.yaml
└── docs/
    └── figures/
