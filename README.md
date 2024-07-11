# DBT Fundamentals Course Project

For examples of visualizations built with Looker Studio using a dbt production schema deployed on AWS Redshift, please see:

- [Bar plot from customers dimension table](https://lookerstudio.google.com/reporting/a263be35-a243-4da4-b315-cdb1378b30d5)

## Directory Structure

Note that some advanced dbt features were not needed for this project, including analyses, macros, seeds, and snapshots. As a result some default project directories were not used.

- models
  - marts
    - finance
    - marketing
  - staging
    - jaffle_shop
    - stripe
- tests

## Project Walkthrough

### Warehouse Setup

For this project, an AWS Redshift database was used as the underlying warehouse, but the steps would be similar for BigQuery or Snowflake. Within the `dev` database, schemas for `jaffle_shop` and `stripe` were created, and a few tables were loaded from an S3 bucket, including `customers`, `orders`, and `payment`.

### Source Models

In dbt, these three tables were intialized as "sources", which represent raw data before transformations have been applied. A benefit of sources is being able to use Jinja templates like

`select * from {{ source("nice_source_name", "nice_table_name") }}`

in model queries to automatically track the underlying database and schema, rather than having to hardcode that information in several places each time the warehouse changes. Minimal data tests were applied to these models, including tests for non-missingness and uniqueness on the primary keys, and a freshness warning was applied to the `payment` source for data older than 24 hours.

### Staging Models



### Marts Models



### Deployment


