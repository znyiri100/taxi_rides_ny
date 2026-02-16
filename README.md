# NYC Taxi Rides DBT Project

This dbt project transforms and analyzes NYC Taxi & Limousine Commission (TLC) data. It supports both local development using DuckDB and cloud deployment using Google BigQuery.

## Prerequisites

- **Python**: Version 3.11 or higher
- **dbt**: Version 1.7.0 or higher (`dbt-duckdb` for local, `dbt-bigquery` for cloud)
- **DuckDB**: For local development
- **Google Cloud Platform Account**: For cloud deployment (BigQuery)

## Setup

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd taxi_rides_ny
    ```

2.  **Install dependencies:**
    It is recommended to use a virtual environment.
    ```bash
    python -m venv .venv
    source .venv/bin/activate
    pip install -r requirements.txt  # If requirements.txt exists, otherwise install dbt adapters manually
    # pip install dbt-duckdb dbt-bigquery
    ```
    *Note: The project specifies `require-dbt-version: [">=1.7.0", "<2.0.0"]` in `dbt_project.yml`.*

## Local Deployment (DuckDB)

Local development uses DuckDB to process data without needing a cloud provider.

1.  **Download Data:**
    Use text script `local_download.py` to download Yellow and Green taxi data and convert it to Parquet format.
    ```bash
    python local_download.py
    ```
    This will create a `data/` directory and a `taxi_rides_ny.duckdb` database file.

2.  **Run dbt:**
    Execute dbt commands using the `dev` target (default).
    ```bash
    dbt deps
    dbt build --target dev
    ```

    You can query the results directly in DuckDB:
    ```bash
    duckdb taxi_rides_ny.duckdb "SELECT * FROM prod.fct_monthly_zone_revenue LIMIT 10"
    ```

## Cloud Deployment (BigQuery)

Cloud deployment targets Google BigQuery for scalable data warehousing.

1.  **GCP Configuration:**
    - Ensure you have a Google Cloud Project set up.
    - specialized environment variable `GCP_PROJECT_ID` must be set to your project ID.
    ```bash
    export GCP_PROJECT_ID="your-gcp-project-id"
    ```
    - Configure authentication (e.g., using `gcloud auth application-default login` or setting `GOOGLE_APPLICATION_CREDENTIALS`).

2.  **dbt Configuration:**
    Ensure your `profiles.yml` is configured for the `prod` target using the `bigquery` adapter.
    
    Example `profiles.yml` entry:
    ```yaml
    taxi_rides_ny:
      target: dev
      outputs:
        dev:
          type: duckdb
          path: 'taxi_rides_ny.duckdb'
        prod:
          type: bigquery
          method: oauth
          project: "{{ env_var('GCP_PROJECT_ID') }}"
          dataset: nytaxi
          threads: 4
    ```

3.  **Run dbt:**
    Execute dbt commands using the `prod` target.
    ```bash
    dbt deps
    dbt build --target prod
    ```

## Project Structure

- `models/staging`: Raw data transformation and cleaning.
- `models/marts`: Business logic and reporting tables.
- `macros`: Custom dbt macros (including dialect-specific logic for DuckDB/BigQuery).
- `local_download.py`: Script to seed local DuckDB with data.
