# MLflow + Pinecone Homework

Airflow stack for two DATA 226 assignments:

- **HW9** — IMDB sentiment classification logged to MLflow
- **HW10** — Medium article search engine using Pinecone vector DB

## Stack

- **Airflow 2.10.1** — pipeline orchestration (http://localhost:8082)
- **MLflow 2.13.0** — experiment tracking (http://localhost:5001)
- **Postgres 13** — Airflow metadata DB
- **Pinecone (serverless)** — vector index for semantic search
- **sentence-transformers** — `all-MiniLM-L6-v2` embeddings (384-dim)

## Setup

1. Start the stack:
   ```bash
   docker compose -f docker-compose-mlflow.yaml up -d
   ```
   First boot takes a few minutes — `_PIP_ADDITIONAL_REQUIREMENTS` installs
   torch, sentence-transformers, pinecone, mlflow, and dbt-snowflake.

2. Open Airflow at http://localhost:8082 (login `airflow` / `airflow`).

3. **HW10 only** — create an Airflow Variable named `pinecone_api_key`
   with your Pinecone API token (Admin → Variables → +).

## DAGs

### `dags/build_pinecone_search.py` (HW10)

`Medium_to_Pinecone` — 5-task pipeline:

| Task | Purpose |
|------|---------|
| `download_data` | Fetch `medium_data.csv` from S3 |
| `preprocess_data` | Build `metadata` column (title + subtitle) |
| `create_pinecone_index` | Create `semantic-search-fast` (dim=384, dotproduct) |
| `generate_embeddings_and_upsert` | Encode titles with MiniLM, upsert to Pinecone |
| `test_search_query` | Top-5 similarity search for `"what is ethics in AI"` |

Trigger from the Airflow UI. Runs end-to-end in ~40 seconds after the
embedding model is cached.

### `dags/imdb_sentiment.py` (HW9)

IMDB sentiment training pipeline with MLflow tracking. Logs metrics,
parameters, and the trained model as MLflow artifacts.

## File layout

```
MLFLOW_pinecone_HW/
├── .gitignore
├── README.md
├── docker-compose-mlflow.yaml    # Airflow + MLflow + Postgres
└── dags/
    ├── build_pinecone_search.py  # HW10
    └── imdb_sentiment.py         # HW9
```

`mlflow-env/`, `logs/`, and `__pycache__/` are gitignored.

## Tear down

```bash
docker compose -f docker-compose-mlflow.yaml down
```

Add `-v` to also remove the Postgres and MLflow data volumes.
