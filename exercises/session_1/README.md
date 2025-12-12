# Exercises for Session 1: Data Ingestion and Spark SQL

## Components

Find below the list components which we will be using:

| Component  | Description | Docker Service | Port | Credentials |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| **Airbyte** Cluster | Data integration platform for data ingestion | `airbyte/airbyte:latest` | [http://localhost:8000](http://localhost:8000) | depends on first setup, check via `abctl local credentials` |
| **JupyterLab with Spark** | JupyterLab environment with Apache Spark for data processing | `jupyter/pyspark-notebook:latest` | check container logs for url with token | works with token  |
| **Mage AI** | Open-source data pipeline tool for orchestration | `mageai/mageai:latest` | [http://localhost:6789](http://localhost:6789) | Email: admin@admin.com <br> Password: admin |
| PostgreSQL | Relational database for storing ingested data | `postgres:latest` | `postgres://mage_admin:mage_pass@localhost:5432/mage_db` | Username: mage_admin <br> Password: mage_pass <br> Database: mage_db |
| **Adminer** | Lightweight web-based DB UI | `adminer:latest` | [http://localhost:8080](http://localhost:8080) | System: PostgreSQL <br> Server: postgres <br> Username: mage_admin <br> Password: mage_pass <br> Database: mage_db |


## Exercises for Airbyte

## 1. Setting up Airbyte to ingest data into PostgreSQL

Make sure to have Airbyte installed and running. If not, follow the setup instructions in the [main README](../../README.md).

Also run the PostgreSQL and Adminer services by executing:

```bash
docker-compose up -d postgres adminer
```

We want to ingest the `tickets.csv` dataset (source connector) into the PostgreSQL database (destination connector) using Airbyte.

Please use the [GDrive link](https://drive.google.com/uc?export=download&id=129kvuZIfxB0qQAeYQWxl0lOAh2jvpNKA) as the source for the CSV file.


## 2. (Optional) Ingest data also into DuckDB and Qdrant

If you want to extend the exercise, you can also set up additional destination connectors to ingest the data into DuckDB and Qdrant vector database. This will allow you to build an end-to-end data pipeline that includes data ingestion, storage, and vector search capabilities (session 2 and session 4).


## Exercises for Mage AI

Same as before, we need the `tickets.csv` dataset for these exercises from the `data` folder. Make sure to mount the `data` folder as a volume in the Docker container. To do this, modify the `docker-compose.yaml` file to include the following line under the `volumes` section for the Mage service.

Then, launch Mage AI by running:

```bash
docker-compose up -d mage postgres adminer
```

### 3. Get familiar with Mage AI

Open Mage AI in your web browser by navigating to [http://localhost:6789](http://localhost:6789). Log in using the credentials provided in the components table above.

Explore the Mage AI interface and familiarize yourself with its features. Look for the sample pipelines provided in the environment.

What do you see? How are they structured? What components do they use?

Run the sample pipelines to see what happens.

### 4. Extend the `example_pipeline` by inserting the data into PostgreSQL.

Open the `example_pipeline` provided in the Mage AI environment. Your task is to extend this pipeline to insert the processed data into the PostgreSQL database.

Make sure to configure the PostgreSQL connection settings in the pipeline. Which file do you need to modify to set the connection parameters?

### 5. Add a trigger to the pipeline

Mage AI allows you to schedule pipelines to run at specific intervals or by a trigger.

See how to add a trigger to the pipeline so that it runs whenever you make an API call.

Verify that the pipeline runs successfully when triggered. Check the PostgreSQL database to ensure that the data has been inserted correctly.

### 6. Create a new pipeline to process the Tickets dataset

Create a new pipeline in Mage AI to process the `tickets.csv` dataset. The pipeline should perform the following tasks:

- Read the `tickets.csv` file.
- Filter for tickets in English language only and save the result into a new table called `tickets_en` in PostgreSQL.
- Create a custom transformation that retrieves the top tags across all tag columns used in the tickets and their counts. Save the result into a new local file.
- Extract the number of tickets by priority and create a bar chart visualization.



## Exercises for Spark SQL
Launch the JupyterLab with Spark environment by running:

```bash
docker-compose up -d spark
```

### 7. Getting started - Recap on PySpark

Once opened, navigate to the `notebooks` folder and open the `01-getting-started.ipynb` notebook. Follow the instructions provided in the notebook to complete the exercises. This notebook contains exercises to help you learn PySpark fundamentals operations. Follow the instructions in each cell to complete the tasks.


### 8. Working with Spark SQL

Now, open the `02-spark-sql.ipynb` notebook. Follow the instructions provided in the notebook to complete the exercises. This notebook contains exercises to help you learn Spark SQL operations. Follow the instructions in each cell to complete the tasks.

> **Please note:**
>
> We need the `tickets.csv` dataset for these exercises from the `data` folder. Make sure to place the dataset in the appropriate directory so that the notebook can access it. You can either:
>
> a. upload it directly to the Jupyter environment or
>
> b. better, mount the `data` folder as a volume in the Docker container. To do this, modify the `docker-compose.yaml` file to include the following line under the `volumes` section for the Spark service.
