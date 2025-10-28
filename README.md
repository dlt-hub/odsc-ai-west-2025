# Workshop: Production-Ready Data Ingestion for Recovering Pandas Users


Supporting materials:
- [Colab 1: Managing schemas](https://colab.research.google.com/drive/1OpNx-oOIDk-cD5SQa8ipEni1bEnB2hpi?usp=sharing) 
- [Colab 2: Managing hardware](https://colab.research.google.com/drive/1vt4blX1_icTNrtVmvZy8wPdUYmwmuUXD?usp=sharing) 
- [Colab 3: Incremental loading](https://colab.research.google.com/drive/1c3GApTKWnlh-h5cwrJ7XPvTWOuEVlwZD?usp=sharing) 
- [Colab 4: Moving faster with extra tools](https://colab.research.google.com/drive/1PbDDtd5rdZrNmcyuPbG6k71A2s2AcpDB?usp=sharing)
- [Slides](https://docs.google.com/presentation/d/16zBkcFDxRNq6Fpkq1cUoQek3ISzDk5J4EbZy5Qa2ErU/edit?usp=sharing)



**Presented by Adrian Brudaru, Co-founder @ dltHub**

## Presenter
- **Name**: Adrian Brudaru
- **Role**: Co-founder, dltHub; data engineer since 2012
- **Motivation**: Created `dlt` (data load tool) as the tool he wished he had for building data warehouses and data products

## Narrative
The workshop addresses the friction between Data Scientists ("Pandas Users") and Data Engineers.

- **Pandas / ML Perspective**:
  - `pandas` is used for everything: loading, transforming, memory management, unnesting, cleaning
  - Perfect for prototyping in local Jupyter notebooks
  - Problems begin when "moving to production"

- **Data Engineer Perspective**:
  - Focus on scale, efficiency, resilience, maintainability, and testing
  - A simple `pandas.load_json()` + `df.to_sql()` hides a long production checklist
  - Requirements include: schema management, atomicity, idempotency, state persistence, incremental loading, memory management, parallelism, retries, schema evolution, data normalization, and data contracts

The slides contrast a simple 4-step `pandas` flow with the complex, multi-component system required to be production-ready.

## The Proposed Solution: `dlt`
`dlt` is introduced as the solution: as easy to use as `df.to_sql()` while providing real-life production features.

- **Shallow learning curve**
- **Transparent**: not a black box; users own their code
- **Vendor-agnostic**: avoid lock-in; move between destinations easily

## Workshop Agenda (Deep Dive Topics)

Rapid-fire explanations across four areas, with self-paced examples in notebooks

### Topic 1: Schema Management

- **Schema Inference**: Automatic schema is inferred on first run. Weakly typed JSON becomes strongly typed relational structures by flattening dictionaries and unpacking lists into sub-tables. 
- **Schema Evolution**: Handles source changes by adding new columns/tables by default; can be configured to alert (e.g., Slack) on change. 
- **Data Contracts**: When you "hate change", freeze the schema via `schema_contract` to control tables, columns, and `data_type` with modes like `evolve`, `freeze` (stop load), `discard_row`, or `discard_value`. Pydantic models can be used as 

### Topic 2: Hardware Bottleneck Management


- **Memory (RAM)**: Use Python generators to yield data in chunks; configure `buffer_max_items` so dlt buffers to files after N items.
- **Disk**: For small disks (e.g., serverless):
  - Chunk a source using `chunk_size` and loop `pipeline.run()` (one-offs/backfills)
  - Mount storage and set `DLT_DATA_DIR` as an "infinite disk"
- **CPU (Async I/O)**: Speed up I/O-bound APIs:
  - Use `@dlt.resource(parallelized=True)` to parallelize
  - Use `async def` resources; dlt runs them in parallel automatically (configurable worker count)
- **CPU (Parallelism)**: Fully utilize CPU during normalize via a process pool; configure `[normalize] workers`.
- **Network (Retries)**: Use `from dlt.sources.helpers import requests` for retries, exponential backoff, and HTTP 429 handling (respecting `Retry-After`).

### Topic 3: Incremental Loading & State

- **Write Dispositions**: `replace` (full load), `append` (stateless), `merge` (upsert with primary key), `scd2` (Type 2, `valid_from`/`valid_to`)
- **State Handling**: dlt state is a Python dict persisted in a separate destination table (more robust than orchestrator/implicit state)


### Topic 4: Bonus Round & dlt Ecosystem

- **REST API Source**: Templated source for many APIs without writing Python; configure client, auth, paginator, and resources in a dictionary.
    Docs link: [rest api source](https://dlthub.com/docs/dlt-ecosystem/verified-sources/rest_api/basic)
- **Workspace Dashboard**: Browse schemas, debug pipelines, overall view.
    Docs link: [validate with dasbboard](https://dlthub.com/docs/dlt-ecosystem/llm-tooling/llm-native-workflow#validate-with-pipeline-dashboard)
- **Ibis Integration**: Query datasets with a single, backend-agnostic API in ~20 systems (DuckDB, Snowflake, BigQuery, etc.).
    Docs link: [Ibis](https://dlthub.com/docs/general-usage/dataset-access/ibis-backend)
- **Marimo Notebooks**: Reactive notebooks saved as clean `.py` files (not `.ipynb`).
    Docs link:
- **LLM-native Scaffolding**: 4,100+ scaffolds generated from API docs to solve long-tail connectors.
    Link: [marimo](https://dlthub.com/docs/general-usage/dataset-access/marimo)
- **LLM-native Workflow**: (1) Init scaffold, (2) Generate running code, (3) Debug in dashboard, (4) Explore in Marimo.
 - Docs link: [LLM native workflow](https://dlthub.com/docs/dlt-ecosystem/llm-tooling/llm-native-workflow) 



