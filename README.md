# FoMoLess

A daily pipeline that ingests tech articles from 9 open sources (dev.to, Medium, Towards Data Science, BBC, Hacker News, Stack Overflow Blog, Stack Exchange, KDnuggets, The Guardian), scores them for trending momentum, and serves the results via MongoDB and an email digest.

## Downstream 
downstream will be viewed as a website with the link: https://fomoless.streamlit.app/ 
## Architecture

Sources → Azure Data Factory → ADLS Gen2 (raw) → Databricks / PySpark / Delta Lake (Bronze → Silver → Gold, medallion architecture) → MongoDB Atlas → daily email digest

## Repo structure

```
fomoless-pipeline/
  adf/                    Exported ADF pipelines, datasets, linked services (ARM JSON)
  databricks_notebooks/   The pipeline itself — see stages below
data-exploration/         Early exploration/prototyping, superseded by fomoless-pipeline
```

## Pipeline stages (`fomoless-pipeline/databricks_notebooks/`)

1. `00_config` — shared config, source list, and helper functions used by every other notebook
2. `01_setup` — catalog/schema setup and landing zone check
3. `02_bronze_ingestion` — incremental raw ingestion into ADLS
4. `03_Silver_transform` — harmonize, deduplicate, and validate raw records
5. `04_gold_aggregation` — incremental trend scoring (Change Data Feed-driven)
6. `05_data_quality` — validation gate; fails the run if checks don't pass
7. `06_load_mongodb` — incremental sink to MongoDB Atlas
8. `07_send_digest` — sends the daily email digest

## Setup

```
pip install -r requirements.txt
pip install -r fomoless-pipeline/databricks_notebooks/requirements.txt
```

Set these as environment variables (or Databricks widgets) — never hardcode them:

| Variable | Used for |
|---|---|
| `AZURE_STORAGE_CONNECTION_STRING` | ADLS Gen2 read/write |
| `MONGO_URI` | MongoDB Atlas connection (should include the target database name) |
| `MONGO_COLLECTION_NAME` | Target MongoDB collection |
| `GUARDIAN_API_KEY` | The Guardian API |

Run the notebooks in order, `00` through `07`, either manually in Databricks or via the ADF pipeline in `fomoless-pipeline/adf/`.
