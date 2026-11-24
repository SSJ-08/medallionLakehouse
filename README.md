# medallionLakehouse
An automated, incremental data pipeline on Databricks that turns raw NYC Taxi trip data into clean, analytics-ready tables using the medallion architecture (Bronze → Silver → Gold).

1. **Project Overview**

This project implements the medallion architecture on the Databricks Lakehouse platform. Raw NYC Taxi (TLC) trip records are landed as files, ingested incrementally, cleaned and validated, and aggregated into business-level tables. The pipeline runs on a schedule with no manual notebook reruns, and every run is logged for monitoring.

Key features

Incremental processing: only new files and records are processed on each run.
Data quality checks: invalid records are routed to a quarantine table with rejection reasons.
Idempotent, fault-tolerant runs: rerunning a failed job does not create duplicates or lose data.
Pipeline monitoring: a run-log table tracks row counts, rejected records, status and duration.
Automation: scheduled with Databricks Jobs, including retries and failure alerts.


2. **Problem Statement**

Raw trip data arrives continuously and is messy: it contains duplicate records, null values, invalid timestamps, negative fares and inconsistent types. Analysts cannot trust or efficiently query it in that form, and reprocessing the entire dataset on every run is slow and wasteful.

This project solves that by building a pipeline that:

Preserves raw data for auditing and replay (Bronze).
Cleans, validates and standardizes it (Silver).
Produces analytics-ready datasets for business questions (Gold).
Processes only new data, runs automatically, and reports on its own health.

4. **Technology Stack**
Area	Technology
Platform	Databricks (Unity Catalog, Volumes, Jobs/Workflows)
Processing	PySpark, Spark SQL
Storage	Delta Lake
Ingestion	Auto Loader (cloudFiles) / Structured Streaming
Language	Python
Version control	Git, GitHub
Quality and monitoring	Custom data quality checks, quarantine table, run-log table
Data source	NYC Taxi & Limousine Commission (TLC) Trip Record Data


5. **Pipeline Workflow**

Source data: NYC TLC Yellow Taxi trip records (monthly Parquet files) and the Taxi Zone Lookup table.

Bronze: raw ingestion
Auto Loader reads new files from the landing Volume.
Raw records are stored unchanged in a Delta table.
Metadata columns are added: _ingest_time, _source_file, _batch_id.
A checkpoint tracks processed files so nothing is ingested twice.

Silver: cleaned data
Cast columns to correct data types and standardize column names.
Remove duplicates using a generated trip key.
Handle missing values.
Validate records, for example: dropoff after pickup, fare and distance within valid ranges, valid location IDs.
Valid rows are upserted into Silver with MERGE; invalid rows go to the quarantine table with a rejection reason.

Gold: analytics
Daily revenue
Trip trends over time
Peak-hour demand
Popular pickup → dropoff routes
Location-based statistics (joined with the zone lookup)

Automation and monitoring
A Databricks Job chains Bronze → Silver → Gold as tasks, with a schedule, retries and failure alerts.
Each run writes to pipeline_run_log: layer, rows read, rows written, rows rejected, duration and status.
Quality thresholds (for example, rejection rate) can fail a run.


6. **Setup Instructions**
Prerequisites
A Databricks workspace 
A GitHub account
Python 3.x (only needed locally for downloading the data)

Step 1: Clone the repository
bash
git clone https://github.com/<your-username>/<repo-name>.git

In Databricks, add the repo under Workspace → Repos / Git folders to sync the notebooks.

Step 2: Create the catalog, schemas and Volume
sql
CREATE CATALOG IF NOT EXISTS taxi;
CREATE SCHEMA IF NOT EXISTS taxi.raw;
CREATE SCHEMA IF NOT EXISTS taxi.bronze;
CREATE SCHEMA IF NOT EXISTS taxi.silver;
CREATE SCHEMA IF NOT EXISTS taxi.gold;
CREATE SCHEMA IF NOT EXISTS taxi.monitoring;
CREATE VOLUME IF NOT EXISTS taxi.raw.landing;

Step 3: Download the data
Go to the NYC TLC Trip Record Data page.
Download several consecutive months of Yellow Taxi Parquet files from the same year, plus the Taxi Zone Lookup CSV.
Step 4: Upload to the landing Volume
/Volumes/taxi/raw/landing/yellow/      # monthly trip files (upload one month first)
/Volumes/taxi/raw/landing/reference/   # taxi_zone_lookup.csv
Step 5: Run the pipeline

Run the notebooks in order, or trigger the Job:

01_bronze_ingestion
02_silver_transformation
03_gold_aggregations
Step 6: Simulate new data and schedule
Upload the next month's file to yellow/ and rerun. Only the new file should be processed.
Create a Databricks Job with the three notebooks as dependent tasks, then add a schedule, retries and failure notifications.
