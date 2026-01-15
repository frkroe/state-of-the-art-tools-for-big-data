# dbt (Data Build Tool) — Practical Exercises 

This repo contains the code and instructions for practical exercises that will teach the basics of [**dbt (Data Build Tool)**](https://docs.getdbt.com/docs/introduction#dbt-core), a **transformation workflow tool** designed to help analysts and data engineers **transform and test their data as code**, following software engineering best practices.

For the exercises, we will use **dbt Core**, the open-source command-line tool offered by dbt as a product. dbt Core allows for the quick installation of dbt and [dbt adapters](https://docs.getdbt.com/docs/connect-adapters) as Python libraries, meaning that **having a compatible version of Python is the only requirement to start using dbt**.

By completing these exercises, you will learn about the essential components of dbt's workflow:

- **Models**: these are SQL-based files that define your transformations.
- **Tests**: these can be written to validate assumptions about your data, such as checking for null values, uniqueness, or referential integrity.
- **Documentation**: dbt offers features for documenting your data models and their relationships.
  
## Technologies used

The exercises are based on:

- **dbt Core**
- **DuckDB** as an embedded analytical database
- **duckcli** as a command-line tool to inspect DuckDB

## Execution model

Everything runs locally inside a **single Docker container**.

- No external database
- No cluster
- No scheduler
- No additional services

This setup is intentionally simple to focus entirely on **dbt concepts and workflows**.

## What you will learn

By completing these exercises, you will learn:

- What dbt is and which problems it solves
- How dbt models are defined using SQL files
- How dbt executes SQL and materializes tables and views
- Why the staging layer exists and how to design it
- How to build analytics-ready models with joins and aggregations
- How to add data quality tests using `schema.yml`
- How dbt documentation works and how it is generated
- How to reuse SQL logic using macros
- How dbt is used in a real workflow with `dbt build`, `dbt test`, and model selection

## Project overview

This project uses the fictional **Jaffle Shop** dataset, which contains:

- **customers**: customer information  
- **orders**: orders placed by customers  
- **payment**: payments made for orders  

The dbt project follows a **layered approach**:

Raw data (CSV seeds)  
→ Staging models (views)  
→ Core models (tables)  
→ Tests and documentation  
To learn about the dbt core elements, we will use an example with data for a fictional Jaffle Shop.

The Jaffle Shop dataset has data about its **customers**, **orders** and **payments**. The columns for each table, as well as the relationships between them, can be seen in the following Entity Relationship Diagram:

<div style="text-align: center;">
  <img src="./images/jaffle_shop_erd.png" alt="Jaffle Shop ERD" width="750px" height="250px">
</div>

We will create models that will be **materialized as views and tables** in a [**DuckDB**](https://duckdb.org/) database, as seen in the following diagram:

<div style="text-align: center;">
  <img src="./images/dbt-diagram-background.png" alt="Jaffle Shop dbt diagram" width="750px" height="300px">
</div>

## Exercise setup

### Requirements

- Docker installed and running
- Repository cloned locally
- Visual Studio Code is recommended

### Step 1 — Move to the exercise directory

    cd exercises/session_3/

You should see a `src/` directory.

### Step 2 — Run the container (single container)

    docker run --name dbt_container -it -v ${PWD}/src:/app/src -p 8080:8080 python:3.10-slim /bin/bash

This starts a single container with Python installed.  
DuckDB runs as an embedded database inside this container.

Here's a breakdown of the options used in this command:

- `docker run`: The main Docker command to run a container from an image.
- `--name dbt_container`: This option names the container `dbt_container` so it can be easily referenced later (e.g., for stopping, starting, or inspecting).
- `-it`: This option attaches an interactive terminal to the container.
- `-v ${PWD}/src:/app/src`: This option mounts a volume from the host machine to the container.
  - `${PWD}/src`: Refers to the current directory's `src` folder on the host machine.
  - `/app/src`: Specifies the target directory inside the container where the folder is mounted. This allows your local files to be accessible within the container.
- `-p 8080:8080`: maps port `8080` on the local machine to port `8080` in the container.
- `python:3.10-slim`: This specifies the image to use. In this case, it pulls the `python:3.10-slim` image, which is a lightweight version of Python 3.10, suitable for installing dbt Core.
- `/bin/bash`: This is the command that runs inside the container, opening a Bash shell for further interaction.



### Step 3 — Install dependencies inside the container

    cd /app/src
    pip install -r requirements.txt

Within the installed libraries, the following are the most important in the context of this setup:

dbt-duckdb~=1.5.0: This is the DuckDB adapter for dbt (Data Build Tool). By installing dbt-duckdb, you also install dbt-core.
duckcli~=0.2.1: A command-line interface (CLI) tool for interacting with DuckDB. It provides a convenient way to query, inspect, and manage DuckDB databases directly from the terminal. This tool is especially useful for quickly testing SQL queries or exploring your database.

### Step 4 — Move into the dbt project

    cd /app/src/jaffle_shop

## Exercise 0 — Sanity checks

### Goal

Verify that dbt is installed, DuckDB can be created, and raw data loads correctly.

### Check dbt installation

    dbt --version

### Load raw CSV files (seeds)

    dbt seed

This creates raw tables in DuckDB:

- customers
- orders
- payment

A file named `jaffle_shop.duckdb` is created.

### (Optional) Inspect raw tables

    duckcli jaffle_shop.duckdb
    show tables;
    select * from customers limit 5;
    select * from orders limit 5;
    select * from payment limit 5;
    exit

If these queries work, the environment is ready.

## Exercise 1 — Staging layer (views)

### Goal

Create staging models that clean raw data, rename columns consistently, apply very small transformations, and contain no business logic.

All staging models are materialized as views.

### Create the staging folder

    models/staging/

### Create the following staging models

#### stg_customers.sql

This model renames the primary key and selects only relevant columns.

Hint:
- Start from the raw `customers` table.
- Rename the `id` column to `customer_id`.
- Select only the columns that make sense for downstream models (avoid `select *`).

#### stg_orders.sql

This model normalizes foreign keys and keeps relevant order attributes.

Hint:
- Start from the raw `orders` table.
- Rename `id` to `order_id`.
- Rename `user_id` to `customer_id` to standardize the join key name.
- Keep the columns needed later (e.g., date and status).

#### stg_payments.sql

This model converts amounts from cents to the main currency unit and normalizes column names.

Hint:
- Start from the raw `payment` table.
- Rename `id` to `payment_id`.
- Convert `amount` from cents to currency by dividing by 100 (use a decimal like `100.0`).
- Keep method and status columns for later analysis.

### Configure staging models to be materialized as views

Edit `dbt_project.yml` and configure the `staging` folder so models are materialized as `view`.

Hint:
- Use the `models:` section and scope the config to the `jaffle_shop` project and the `staging` directory.

### Add staging schema file

Create a `schema.yml` file inside `models/staging/` to describe the staging models and their columns.

This file does not execute SQL.  
It is used for documentation and tests.

Hint:
- Add entries under `models:` for each staging model.
- Include at least the model name and a minimal columns list.

### Build staging models

    dbt build -s staging

## Exercise 2 — Core models (tables)

### Goal

Create analytics-ready tables using staging models only.

Models to build:
- customers_orders
- customers_payments
- customers_orders_payments

### Configure core models to be materialized as tables

Update `dbt_project.yml` so that:
- models are materialized as `table` by default
- staging models remain materialized as `view`

Hint:
- You will need a `models:` section scoped to `jaffle_shop`.
- Set `+materialized: table` at the project level and override `staging` to `view`.

### customers_orders.sql

This model creates customer-level order metrics.

Hints:
- Always read from staging models (use `ref()`), not raw tables.
- Create a `customers` CTE reading from the staging customers model.
- Create an `orders` CTE reading from the staging orders model.
- Join customers to orders using `customer_id` with a LEFT JOIN.
- Build these fields:
  - customer_id
  - customer_full_name (concatenate first_name and last_name)
  - first_order (min order_date)
  - most_recent_order (max order_date)
  - number_of_orders (count orders)
- Group by customer fields.

### customers_payments.sql

This model calculates customer-level payment metrics based on successful payments.

Hints:
- Start from staging orders and staging payments using `ref()`.
- Filter payments to only the successful ones (based on `status`).
- Join payments to orders by `order_id` to connect payments to customers.
- Produce:
  - customer_id
  - number_of_orders (count distinct orders)
  - total_amount (sum payment amount)
- Group by customer_id.

### customers_orders_payments.sql

This model combines order and payment metrics.

Hints:
- Build this model by joining the outputs of the two previous core models.
- Join on `customer_id`.
- Keep all customers from the orders model and bring payment totals when available (LEFT JOIN).

### Add core schema file

Create a `schema.yml` file in `models/` to describe these core models.

Hint:
- Add each model under `models:`
- Include minimal descriptions and a basic column list (can be expanded later).

### Build all models

    dbt build

## Exercise 3 — Tests (data quality)

### Goal

Add tests to enforce basic data quality assumptions.

The tests are **intentionally not fully provided**.  
Students are expected to **define most of the tests themselves** based on their understanding of the data and the models built so far.

What to test (high level):
- Primary keys are **not null** and **unique**
- Foreign keys reference valid records (**relationships**)
- Order status values belong to an **accepted set**

Tests must be defined in the corresponding `schema.yml` files.

### Example test (provided as guidance)

Below is **one example** of how a test can be declared in a `schema.yml` file.

This example enforces that the primary key of the `stg_customers` model is not null and unique:

    models:
      - name: stg_customers
        columns:
          - name: customer_id
            tests:
              - not_null
              - unique

This example is meant only as a reference.  
Students should add additional tests for other models and columns following the same pattern.

### Run tests

    dbt test

If tests fail, inspect the failures and decide whether:
- the data is incorrect,
- the transformation logic should be adjusted, or
- the test definition is too strict or incorrect.

The goal of this exercise is not to have all tests pass immediately, but to understand **how dbt enforces data quality and assumptions**.


## Exercise 4 — Documentation

### Goal

Generate and improve dbt documentation.

Generate documentation:

    dbt docs generate

Serve documentation:

    dbt docs serve --port 8080

Open the documentation site in the browser at:
http://localhost:8080

Improve documentation by:
- Adding richer descriptions in `schema.yml`
- Creating Markdown documentation blocks
- Customizing the overview page
## Exercise 5 — Macros

### Goal

Reuse SQL logic using macros.

Task:
- Create a macro that converts cents to currency units.
- Reuse that macro in your staging models (where amounts are converted).
- After updating models to use the macro, rebuild staging models.

Where to start:
- Create a new folder named `macros/` in the dbt project if it does not exist.
- Create a macro file (e.g., something like `macros/money.sql`) and define a macro inside it.

Hints:
- A macro is a small reusable SQL snippet written with Jinja.
- The macro should accept **one argument** (a column name or expression) and return a SQL expression that performs the conversion.
- To use a macro inside a model, you call it with Jinja syntax: `{{ ... }}`.
- Replace the hardcoded conversion in your staging model with the macro call.

Suggested mini-checkpoints:
1. Run `dbt build` before changing anything (baseline).
2. Create the macro file and rebuild (should still work).
3. Update **only** the relevant staging model to use the macro.
4. Rebuild only that model (fast iteration), then rebuild all staging.


## Extra — dbt workflow and selection

### Goal

Practice how dbt is used in a real workflow.

Examples:
- Build a model and everything downstream from it
- Build a model and everything upstream from it
- Run the full pipeline

Where to start:
- Pick one model in the middle of the DAG (for example, a staging model or a core model).
- Try running only that model first, then expand the selection.

Hints:
- dbt selection syntax allows you to run only what you need while still respecting dependencies.
- The `+` modifier is used to include related models in the graph:
  - downstream (children)
  - upstream (parents)
- Use selection patterns to answer questions like:
  - “If I change this staging model, what else needs to run?”
  - “If I want this final model, what dependencies must run first?”

Suggested mini-checkpoints:
1. Run a single model only.
2. Run the same model plus everything downstream.
3. Run a final model plus everything upstream.
4. Run the full pipeline and compare runtime with the partial runs.

## Extra — Practical Exercise: EventSpark

### Goal

Put into practice the concepts learned in the main exercise by working on a **second, more open-ended dataset**.

This exercise is intentionally less guided and focuses on applying dbt concepts independently, especially **modeling and transformations**.

---

### Dataset overview

The `extra_seeds/` folder contains a dataset for a fictional online event management platform called **EventSpark**.

EventSpark allows users to:
- create events
- attend events
- purchase tickets
- track event participation

The dataset includes the following tables:
- `users`
- `events`
- `tickets`

These tables are related through a simple event-based domain model (see the EventSpark ER Diagram provided in the repository).

---

### Proposed exercises

The new data can be used to practice dbt modeling concepts.  
Below are some suggested exercises.

---

### 1. Users model

Objective:  
Create a `users.sql` model to clean and transform user data.

Hints:
- Start from the raw `users` seed.
- Apply light cleaning and normalization.
- Think about what fields may be useful downstream.

Example transformations:
- Clean or standardize email domains.
- Flag users based on region or country.
- Normalize identifiers or timestamps.

---

### 2. Events model

Objective:  
Create an `events.sql` model to analyze event characteristics and capacity.

Hints:
- Start from the raw `events` seed.
- Focus on event-level attributes and metrics.

Example transformations:
- Calculate remaining tickets per event.
- Derive event attendance rates.
- Categorize events by type or size.

---

### 3. Tickets model

Objective:  
Create a `tickets.sql` model that joins ticket sales with users and events.

Hints:
- Use joins to connect tickets to users and events.
- Treat this model as the main “fact” table of the dataset.

Example transformations:
- Calculate total revenue per event.
- Aggregate ticket sales by payment method.
- Group metrics by user region or event type.

---

### Notes

This extra exercise is intentionally flexible:
- There is no single correct solution.
- Focus on applying dbt modeling patterns (staging, refs, joins).
- Prioritize clarity and consistency over complexity.


## Extra: dbt project creation

This section shows how the dbt project `jaffle_shop` included in this repo was created for those looking to recreate it or learn how a dbt project is initialized.

First, it is necessary to **install dbt Core and an adapter**, in this case the DuckDB adapter.

To do that, you can **create and activate a Python virtual environment**:

```
python -m venv venv
source venv/Scripts/activate
```

And then, **install all dependencies** in the [`requirements.txt`](./src/requirements.txt) file:

```
pip install -r requirements.txt
```

After that, the dbt project can be created with the `dbt init` command:

```
dbt init jaffle_shop
```

After choosing the adapter to create the project for, the project is created with its initial structure:

<div style="text-align: center;">
  <img src="./images/initial_project.png" alt="Initial project" width="270px" height="400px">
</div>

A project is usually created with a `profiles.yml` file if there is a default one for the adapter you choose. Since there isn't one for DuckDB, the [`profiles.yml`](./src/jaffle_shop/profiles.yml) file was created with this content:

```yml
jaffle_shop:
  outputs:
    dev:
      type: duckdb
      path: 'jaffle_shop.duckdb'
      threads: 4
  target: dev
```

Indicating that a profile with name *jaffle_shop* is of type *duckdb* and will write data to the path *jaffle_shop.duckdb*.

To finish the setup, the CSV files with the Jaffle Shop data were copied to the [seeds](./src/jaffle_shop/seeds/) directory to be used as the input data for the dbt models.
