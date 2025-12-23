# UPV - Inteligencia Artificial & Big Data Analytics - State of the art tools for Big Data solutions

## Introduction
This repository contains hands-on exercises for the "State of the Art Tools for Big Data" course at UPV's master *Inteligencia Artificial & Big Data Analytics*. The course provides practical experience with cutting-edge Big Data technologies and industry-leading tools.

Throughout this course, you will learn to:
- **Ingest data** using [Airbyte](https://airbyte.com/)
- **Orchestrate workflows** using [Mage AI](https://www.mage.ai/)
- **Process data** using [Apache Spark SQL](https://spark.apache.org/sql/)
- **Perform single-node processing** using [Polars](https://pola.rs/) and store data in [DuckDB](https://duckdb.org/)
- **Transform data** using [dbt](https://www.getdbt.com/)
- **Implement semantic search** by storing data in [Qdrant](https://qdrant.tech/) and [PostgreSQL (pg_vector)](https://github.com/pgvector/pgvector) vector databases
- **Develop a chatbot** using [n8n](https://n8n.io/)

![Architecture Diagram](./img/architecture.png)

Each session will cover some of the technologies and has its own exercises:

* **Session 1: Data Ingestion and Spark SQL**:
  * Technologies: Airbyte, Mage and SparkSQL
  * [Exercises](exercises/session_1)
* **Session 2: Single-node processing**:
  * Technologies: Polars, NVIDIA RAPIDS and DuckDB
  * [Exercises](exercises/session_2)
* **Session 3: dbt**:
  * Technologies: dbt and DuckDB
  * [Exercises](exercises/session_3)
* **Session 4: Vector databases**:
  * Technologies: PostgreSQL (pg_vector), Qdrant and n8n
  * [Exercises](exercises/session_4)

## 🚀 Setup Instructions

### Software Requirements

* Docker (https://docs.docker.com/desktop/)
* Python (https://www.python.org/downloads/)
* Visual Studio Code (https://code.visualstudio.com/docs/setup/setup-overview)

### Initial Setup

1. Clone this repository
1. Each exercise has its own docker compose (if required). Change to that folder and follow the instructions.