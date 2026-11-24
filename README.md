# medallionLakehouse
An automated, incremental data pipeline on Databricks that turns raw NYC Taxi trip data into clean, analytics-ready tables using the medallion architecture (Bronze → Silver → Gold).

1. Project Overview

This project implements the medallion architecture on the Databricks Lakehouse platform. Raw NYC Taxi (TLC) trip records are landed as files, ingested incrementally, cleaned and validated, and aggregated into business-level tables. The pipeline runs on a schedule with no manual notebook reruns, and every run is logged for monitoring.

Key features

Incremental processing: only new files and records are processed on each run.
Data quality checks: invalid records are routed to a quarantine table with rejection reasons.
Idempotent, fault-tolerant runs: rerunning a failed job does not create duplicates or lose data.
Pipeline monitoring: a run-log table tracks row counts, rejected records, status and duration.
Automation: scheduled with Databricks Jobs, including retries and failure alerts.
