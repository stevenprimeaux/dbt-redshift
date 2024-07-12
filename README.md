# DBT Fundamentals Course Project

For a visualization built with Looker Studio using a dbt production schema deployed on AWS Redshift, please see:

- [Total Amount Spent by Customer](https://lookerstudio.google.com/reporting/a263be35-a243-4da4-b315-cdb1378b30d5)

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

In dbt, these three tables were intialized as "sources", which represent raw data from the underlying business process. A benefit of sources is being able to use Jinja templates like

`select * from {{ source("nice_source_name", "nice_table_name") }}`

in model queries to automatically track the underlying database and schema, rather than having to hardcode that information in several places each time the warehouse changes. Minimal data tests were applied to these models, including tests for non-missingness and uniqueness on the primary keys, and a freshness warning was applied to the `payment` source for data older than 24 hours.

### Staging Models

Next, a few staging models were created to lightly clean and standardize the raw source data, but without applying any major transformations.

Under the jaffle_shop staging models, `stg_jaffle_shop__customers` and `stg_jaffle_shop__orders` were used to standardize the primary and foreign key column names. Several data tests were also applied at this stage. First, all primary keys were tested for non-missingness and uniqueness, and essential foreign keys were also tested for non-missingness (for example, every order should be associated with a customer). Foreign key relationships were also tested to ensure that a corresponding entry exists in the foreign table for any foreign key in the home table. Categorical variable columns were tested for accepted values.

Under the stripe staging models, `stg_stripe__payments` was used to clean up several column names, as well as converting the payment amount from cents to dollars. Similar tests were applied to this model, including validation of primary keys, checking for essential foreign keys and relationships, and specifying accepted values for categorical columns. In addition to the generic data tests included in the YAML file, a singular test was also added to the `tests/` directory to validate that no payment amounts are negative.

Definitions of accepted values were included in the dbt documentation as well using Jinja markdown blocks within the column description.

### Marts Models

With the source and staging models completed, models could now be added to fulfill analysis needs for specific stakeholders, like the Finance and Marketing teams in this example. One helpful heuristic for when to group a model under marts rather than staging is when joins are involved, since joins often involve adding new information to understand a particular dimension of an ongoing business process.

First, an orders fact table was materialized as `fct_orders` to capture the essential measurements related to a given order (customer, date, amount), as well as foreign keys to access the various dimensions. Data tests were applied at this stage, with the foreign key relationship tests being particularly important for this table structure. And with `fct_orders` now available in the DAG, a customers dimension model could finally be constructed to summarize each customer's lifetime order history, including first order date, last order date, total number of orders, and total amount spent.

### Deployment

Finally, a production environment was created in dbt to deploy the finalized analytical models under the `dbt_prod` schema, rather than the `dbt_sprimeaux` schema used in development. Within this production environment, a recurring weekly job was created to execute `dbt build`, which incrementally materializes and tests each model along the DAG to ensure there are no errors as the production schema is being refreshed with new data. From this deployed production schema, a Looker visualization was built on the `dim_customers` marts model.  
