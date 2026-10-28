# Python and PostgreSQL ETL Pipeline

## Project Overview

This project implements an end-to-end batch ETL pipeline using Python and PostgreSQL. It ingests sales data from CSV files, stores raw records in a Bronze layer, cleans and validates data in a Silver layer, and generates customer-level sales summaries in a Gold layer.

The project focuses on data quality, idempotent ingestion, database upserts, data reconciliation, and audit logging.

## Architecture

```text
CSV Sales Data
      |
      v
Python Batch Ingestion
      |
      v
Bronze Layer
Raw sales records
      |
      v
Silver Layer
Cleaning, validation,
deduplication and upserts
      |
      v
Gold Layer
Customer sales aggregation
      |
      v
Pipeline Validation
      |
      v
Audit Logging
```

## Technology Stack

- Python
- PostgreSQL
- pandas
- NumPy
- psycopg
- Jupyter Notebook
- SQL
- Git and GitHub

## Project Structure

```text
etl_project/
├── 01_bronze_ingestion.ipynb
├── 02_silver_transformation.ipynb
├── 03_gold_aggregation.ipynb
├── 04_pipeline_validation.ipynb
├── 05_audit_logging.ipynb
├── data/
│   └── sales_2026_10_28.csv
├── .gitignore
└── README.md
```

Note: The local CSV file is excluded from Git tracking. Generate or provide the sample data locally before running the pipeline.

## ETL Workflow

### 1. Bronze Layer — Raw Ingestion

- Reads sales data from CSV files in configurable chunks.
- Generates SHA-256 record hashes.
- Stores raw records in PostgreSQL.
- Uses a unique constraint and `ON CONFLICT DO NOTHING` to prevent duplicate ingestion on reruns.

### 2. Silver Layer — Data Transformation

- Reads records from the Bronze layer.
- Standardizes customer IDs and converts data types.
- Validates required fields and rejects invalid records.
- Deduplicates records by order ID.
- Uses PostgreSQL upserts to update records when a newer version is available.

### 3. Gold Layer — Aggregation

- Aggregates sales by customer.
- Calculates total orders and total sales.
- Loads the results into the Gold summary table using upserts.

### 4. Pipeline Validation

- Checks record counts across layers.
- Validates duplicate order IDs.
- Reconciles total sales between Silver and Gold.
- Tests whether rerunning the pipeline produces consistent results.

### 5. Audit Logging

- Uses the `audit.etl_batch_log` table to track batch execution.
- Records batch ID, source file, status, start time, completion time, and error details.
- Supports `STARTED`, `SUCCESS`, and `FAILED` status tracking.

## Database Design

The project uses three PostgreSQL schemas:

| Schema | Table | Purpose |
|---|---|---|
| `bronze` | `sales_raw` | Stores raw ingested records |
| `silver` | `sales` | Stores cleaned and validated sales records |
| `gold` | `sales_summary` | Stores customer-level sales aggregates |
| `audit` | `etl_batch_log` | Tracks ETL batch execution |

Create the database and tables using the SQL definitions maintained with your project.

## Prerequisites

- Python 3.10 or later
- PostgreSQL
- Git
- Jupyter Notebook or VS Code

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/ajaypatil-gif/sales_etl_pipeline.git
cd sales_etl_pipeline
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on Windows:

```powershell
.\.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
pip install pandas numpy "psycopg[binary]" jupyter
```

### 4. Configure PostgreSQL

Create the `sales_etl` database and required schemas and tables.

Configure your database connection using environment variables or a local configuration file that is excluded from Git.

Example PowerShell environment variables:

```powershell
$env:PGHOST="localhost"
$env:PGDATABASE="sales_etl"
$env:PGUSER="postgres"
$env:PGPASSWORD="your_local_password"
```

Update your Python connection code to read these environment variables. Do not commit real passwords or credentials.

### 5. Prepare the input data

Generate the sample CSV using the data-generation code in the Bronze notebook, or place a compatible CSV file in the expected local data directory.

### 6. Run the notebooks

Open Jupyter:

```bash
jupyter notebook
```

Execute the notebooks in this order:

1. `01_bronze_ingestion.ipynb`
2. `02_silver_transformation.ipynb`
3. `03_gold_aggregation.ipynb`
4. `04_pipeline_validation.ipynb`
5. `05_audit_logging.ipynb`

Ensure database settings, file paths, and batch IDs are configured correctly before execution.

## Key Engineering Concepts

- Batch processing and chunk-based ingestion
- Bronze, Silver, and Gold data architecture
- Data cleansing and validation
- SHA-256 record hashing
- Idempotent ingestion
- PostgreSQL `INSERT ... ON CONFLICT`
- Deduplication and incremental updates
- Aggregation and reconciliation
- ETL audit logging
- Git version control

## Future Enhancements

- Add persistent quarantine storage for rejected records.
- Integrate audit logging directly into the complete ETL execution.
- Add automated pipeline orchestration.
- Implement structured logging and retry mechanisms.
- Add automated tests and continuous integration
