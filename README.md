# HW7 — Time-Series Features and JSONB in PostgreSQL

## Overview

This project demonstrates advanced PostgreSQL features for time-series analysis and semi-structured event data. It includes time-series feature engineering, JSONB queries, index optimization, query execution plan analysis, and a comparison between JSONB and normalized relational storage.

## Data Source

The notebook uses the official NYC TLC Yellow Taxi trip data for March 2024.

Source: [NYC Taxi & Limousine Commission — Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

The notebook attempts to download the Parquet file directly from the official TLC data distribution. If the download or loading process fails, it uses a deterministic synthetic dataset containing 60 days across five pickup zones.

The notebook reports which data source was used. If the fallback dataset is used, the results should be interpreted as demonstrations on synthetic data rather than real taxi activity.

## Technologies

- PostgreSQL 16
- Python
- SQLAlchemy
- psycopg2
- pandas
- PyArrow
- Google Colab

## Project Structure

- `HW7_TimeSeries_JSONB_PostgreSQL.ipynb` — implementation, SQL queries, validation results, and execution plans.
- `README.md` — project description and instructions.

## Tasks Implemented

### 1. Daily Taxi Aggregation

The notebook creates `t7_hw_taxi_daily`, aggregated by pickup zone and calendar day.

The table contains:

- `pickup_zone_id`
- `ts_day`
- `trip_count`
- `avg_fare`
- `total_distance`

A composite primary key on `(pickup_zone_id, ts_day)` ensures one daily record per pickup zone.

An optional typed trip table demonstrates timestamp handling using `TIMESTAMPTZ`.

### 2. Time-Series ML Features

The notebook implements four groups of features using PostgreSQL window functions:

- **Lag features:** `trips_lag_1d`, `trips_lag_7d`, and `trips_lag_30d`.
- **Rolling average:** seven-day rolling average fare.
- **Rolling volatility:** 30-day standard deviation of average fare.
- **Expanding features:** cumulative trips and expanding average trips.

All time-series windows partition by `pickup_zone_id` to prevent historical values from different zones from mixing.

The combined feature view is named `v_t7_hw_taxi_features`.

### 3. JSONB Event Log

The table `t7_hw_events` contains 50,000 synthetic events across five event types:

- `page_view`
- `click`
- `add_to_cart`
- `checkout`
- `recommendation_shown`

Each event type has its own payload structure stored in a PostgreSQL `JSONB` column.

The notebook demonstrates:

- JSONB containment using `@>`
- Top-level key existence using `?` and `?|`
- JSONPath queries using `jsonb_path_query` and `jsonb_path_exists`

### 4. Indexing and Query Performance

The notebook uses `EXPLAIN (ANALYZE, BUFFERS)` to inspect execution plans.

The experiments include:

- A JSONB containment query before creating a GIN index.
- A GIN index using `jsonb_path_ops`.
- The same containment queries after index creation.
- A text equality query on `payload ->> 'device'` before and after an expression B-tree index.

The observed plans are used to discuss scan operators, execution time, buffer usage, index selectivity, and index maintenance costs.

PostgreSQL may choose a sequential scan even when an index exists. Conclusions are based on the actual execution plans produced by the notebook.

### 5. JSONB Versus Normalized Storage

The notebook creates `t7_hw_checkouts_norm`, a normalized relational table populated from checkout events.

It compares an aggregate query over JSONB payloads with an equivalent query over typed relational columns using `EXPLAIN (ANALYZE, BUFFERS)`.

The comparison discusses schema flexibility, data validation, query complexity, and performance trade-offs.

A MongoDB aggregation equivalent is also included as explanatory pseudocode.

## How to Run

1. Open the notebook in [Google Colab](https://colab.research.google.com/).
2. Run the notebook cells from top to bottom.
3. Allow the notebook to install dependencies and initialize PostgreSQL.
4. Wait for the data-loading and table-creation steps to complete.
5. Review the time-series features, JSONB query outputs, and execution plans.
6. Complete the Reflection section using the actual results.
7. Save the notebook with its output cells before submitting.

The notebook attempts to use `pgserver` first and provides a PostgreSQL 16 system-service fallback if `pgserver` cannot start.

## Reflection

The final Reflection should be approximately 250–300 words and should be based on the actual query-plan outputs.

It should address:

1. The observed execution plans before and after the GIN index.
2. Why GIN and expression B-tree indexes serve different query patterns.
3. The read/write and storage trade-offs of indexing.
4. JSONB versus normalized relational storage.
5. Schema evolution when introducing a new event type.

Performance measurements should not be assumed in advance; they depend on the execution environment and the plans selected by PostgreSQL.

## Reproducibility

The notebook uses `DROP TABLE IF EXISTS` for the main assignment tables and a fixed reference date for the synthetic taxi dataset.

For submission, execute the notebook from a fresh runtime using **Restart & Run All**, verify that all cells complete successfully, and retain the SQL output cells.

If the synthetic fallback is used, this must remain clearly stated in the notebook and repository documentation.
