# Exercises for Session 4: Vector storage and semantic search

These are the exercises for the last session of the "Herramientas de última generación" course, on vector databases.

This repository contains the following:
* Docker compose with all the infra required: **PostgreSQL**, **Qdrant**, **n8n** and, optionally, **Ollama** for running models
* List of execercises

## 🏗️ Architecture Overview

**n8n** serves as the central orchestrator for building AI agents and automations — it can pull or push data from all other services:
- Query or insert vectors into **Qdrant** or **pgvector**
- Call LLMs and embedding models via **OpenAI** or **Ollama**
- Manage workflows for automation and data enrichment

## Components

Find below the list components which we will be using:

| Component  | Description | Docker Service | Port | Credentials |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| **n8n**      | Workflow automation and orchestration engine  | `n8nio/n8n:latest`              | [http://localhost:5678](http://localhost:5678)  |
| **PostgreSQL** | Relational DB with `pgvector` extension     | `ankane/pgvector:latest`        | `postgres://root:passwd@localhost:5432/vector_db`               |
| **Adminer**  | Lightweight web-based DB UI                  | `adminer:latest`                |  [http://localhost:8080](http://localhost:8080)         | **User**: root <br/> **Passwd**: passwd <br/> **Database**: vector_db |
| **Qdrant**   | Vector database optimized for embeddings      | `qdrant/qdrant:latest`          | Dashboard: [http://localhost:6333/dashboard](http://localhost:6333/dashboard) </br> REST API: [http://localhost:6333](http://localhost:6333)                 |
| **Ollama** (disabled)  | Local LLM runtime (e.g. Llama 3, Mistral)    | `ollama/ollama:latest`          | [http://localhost:11434](http://localhost:11434)                 |


## 1. Generate embeddings via curl/Postman

In this exercise we will do our first tests with OpenAI endpoints for generating embeddings.

1. Get the API key from the teacher
1. Take a look at [OpenAI's documentation](https://platform.openai.com/docs/api-reference/introduction) and look for the embeddings models and their APIs
1. Use curl (or Postman, Bruno or Thunder Client) to hit the embeddings endpoint and verify the API key works.
1. **Question**: What embedding models are avilable? Whare are the main differences between them?

## 2. Generate embeddings with a Python script

In this exercise we will automate the generation of embeddings using a Python script.

1. Create a Python script named `exercise2.py` which takes a text and returns the embeddings vector.

**TIP**: Use [OpenAI's SDK](https://github.com/openai/openai-python)

**TIP2**: Set the `OPENAI_API_KEY` as an environment variable so it can be reused across scripts

## 3. Create an embeddings table in PostgreSQL

In this exercise we will prepare the database to store embeddings.

1. Start the enviornment (`docker compose up -d`)
1. Connect to the database using **Adminer**
1. Create a database for this exercise
1. Install the `pg_vector` extension
1. Create a table with just three columns:
   * **id**: Sequential identifier of the text.
   * **text**: Text for which we are generating the embedding.
   * **embedding**: Vector representation of the text.

**TIP**: Check OpenAI documentation to confirm the vector's size.

**TIP**: Take a look at the  [pg_vector documentation](https://github.com/pgvector/pgvector).

## 4. Insert multiple texts and embeddings from Python

In this exercise we will automate the load of texts and its embeddings in the database.

1. Look for a dataset which has a list of texts, or generate your own.
1. Based the original Python script create a new one (`exercise4.py`) and do the following:
   * Change the input parameter to take text file.
   * Generate an `.env` file with the database and OpenAI config.
   * Read the text file and iterate through the texts.
   * For each text, generate the embedding using OpenAI and store in the database.

**IMPORTANT**: keep the text file size small (10-15 texts) for the time being in order to reduce costs.

## 5. Run queries with different distance metrics (cosine, L2…)

In this exercise we will perform semantic searches in the database.

1. Connect to the database using **Adminer**
1. Using curl/Postman, generate an embedding with the query/question you want to search
1. Perform a query using the L2 distance
1. Test with the other distances
1. **Question**: What is the difference between the differente types of distances? Did you find any difference in the results?

Repeat the above with other queries and/or adding more data for better results.

## 6. (Optional) Using Ollama for embeddings

This is an optional exercise where you can run all the exercises using you local machine and... for free!!!

1. Uncomment the Ollama service in `docker-compose.yaml`.
1. Take a look at the [CLI documentation](https://docs.ollama.com/cli) and check how to download and list models.
1. Take a look at the [available models](https://ollama.com/library).
1. Download an embedding model (e.g. [all-minilm](https://ollama.com/library/all-minilm)).
1. Test the embeddings model using `curl` (or Postman, Bruno, Thunder Client).
1. Take a look at the [API documentation](https://docs.ollama.com/api/introduction).
1. Change the table (or create a new one) to take into account the new model (vector size)
1. Change the Python code to call Ollama instead of OpenAI to generate the embeddings.

## 7. Store the same workflow in Qdrant

In this exercise we will build real use case, where the user will be able to search for similar tickets using a chatbot.
We will start by using a specific dataset, and fixing the Python code to read from it and store the embeddings in Qdrant.

But let's start by learning a little bit from Qdrant:

1. Go to the Qdrant dashboard: http://localhost:6333/dashboard
1. Take a look at the Quickstart
1. Now load some real data
   1. In the "Welcome" tab click on "Load Sample Data" or go to the "Datasets" tab.
   1. Import one of them, play around and learn by going to the collection (under "Collections" tab)

Now, back to work:

1. Download the [dataset](https://www.kaggle.com/datasets/tobiasbueck/multilingual-customer-support-tickets?select=dataset-tickets-multi-lang3-4k.csv) from Kaggle.
1. Take a look at the CSV file and decided which data you would like to embed. (TIP: think about your use case or how you want to use the data).
1. Create a new script (```exercise7.py```) based on exercise4.py with the following updates:
   1. Read from the downloaded file (found under ```data``` in the root folder)
   1. Include the data you want into the payload
   1. Store in Qdrant (More info on Python SDK: https://github.com/qdrant/qdrant-client)
1. Run the script to load the dataset into Qdrant.
1. Go to the collection in Qdrant UI and explore the data, similarities, etc.
1. Perform a search in the UI ("Console" tab). You will need the embeddings for a new query here (as in exercise 5).

**TIP**: Feel free to reduce the size of the tickets file, or just stop when you think there is enough info loaded.

**TIP**: The collection should automatically be created, but you can create it beforehand with custom parameters.

Qdrant Documentation: https://qdrant.tech/documentation/

## 8. Test the LLM from OpenAI

In this exercise we will test the OpenAI's GPT large language model.

1. Take a look at [OpenAI's documentation for responses](https://platform.openai.com/docs/api-reference/responses).
1. Take a look at the [models avaiable](https://platform.openai.com/docs/models).
1. We will be using [GPT-5 nano](https://platform.openai.com/docs/models/gpt-5-nano), which is fast, cheap and more than enough for our tests.
1. Test the model using curl (or Postman, Bruno, Thunder client)
1. Take a look and understand the response (more info in OpenAI's documentation)


## 9. Build a conversational agent with n8n

In this exercise we will complete the use case by building a chatbot agent in n8n.

1. Open n8n: http://localhost:5678
1. The teach will explain a little bit the UI (creating a new workflow, renaming, adding components, etc.)
1. In the UI, create a new workflow and do the following:
   * Add the "Chat Trigger"
   * Add an AI Agent to the chat, and configure with the following:
     * Add the "Chat Model". Use OpenAI chat model and configure the "Credential to connect with" (just place de API Key) and select the model (gpt-5-nano).
     * Add a tool to "Answer questions with a vector store". Add the description of the data.
     * Add Qdrant as a "Vector Store" and configure the "Credential to connect with" (just the URL http://qdrant:6333), add the description (what the collection has) and select the collection (tickets).
     * Add "Embedding" to Qdrant.
1. Test by running the workflow. Just type something in the chat. Think about what data is in Qdrant.
1. By the chat window, take a look at the "reasoning" and steps done. Check what was retrieved from Qdrant.

## 10. (Optional) Using Ollama for LLM

In this optional exercise we will do the chatbot using Ollama (instead of OpenAI) so you can have it for free.

1. Download an LLM which fits in your PC (e.g. gemma3 is 3.3GB)
1. Change the chat and embedding models in n8n to use Ollama, and configure accordingly
1. Test by running the workflow

## 11. (Optional) Hybrid search

In this exercise we will try another type of query. Not just doing a semantic search using embeddings, but also doing a keyword search. Combining everything together in a so call **hybrid search**.

1. Test in Qdrant directly
1. Change in n8n
1. **Question**: Do you get better results?
