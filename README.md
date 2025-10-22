# Workshop: Production-Ready Data Ingestion for Recovering Pandas Users


Supporting materials:
[Colab 1: Managing schemas](https://colab.research.google.com/drive/1OpNx-oOIDk-cD5SQa8ipEni1bEnB2hpi?usp=sharing) 
[Colab 2: Managing hardware](https://colab.research.google.com/drive/1vt4blX1_icTNrtVmvZy8wPdUYmwmuUXD?usp=sharing) 
[Colab 3: Incremental loading](https://colab.research.google.com/drive/1c3GApTKWnlh-h5cwrJ7XPvTWOuEVlwZD?usp=sharing) 
[Colab 4: Moving faster with extra tools](https://colab.research.google.com/drive/1PbDDtd5rdZrNmcyuPbG6k71A2s2AcpDB?usp=sharing) 



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

## Core Technical Concepts (dlt)
Key Python decorators and primitives:

- `@dlt.resource`
  - Defines a data-producing function (e.g., `def coin_list()`)
- `@dlt.transformer`
  - A dependent resource that uses data from previous resource to get more data.
- `@dlt.source`
  - Groups multiple resources/transformers into a single source. Yielded resources get loaded.
- `pipeline.run(source)`
  - Executes the pipeline with a data source that can be a dlt source, dlt resource, or any iterable object (e.g., `pipeline.run(source)`)

Core function: dlt loads json "magically" into strongly typed flat unnested tables.

## Workshop Agenda (Deep Dive Topics)

Rapid-fire explanations across four areas, with self-paced examples in notebooks

### Topic 1: Schema Management

- [Colab notebook link](https://colab.research.google.com/drive/1C0Gt1dmJlkDa_TEqtzEHt1pmHNXDb2NW?usp=sharing)


- **Schema Inference**: Automatic schema is inferred on first run. Weakly typed JSON becomes strongly typed relational structures by flattening dictionaries and unpacking lists into sub-tables. [Colab link](https://colab.research.google.com/drive/1C0Gt1dmJlkDa_TEqtzEHt1pmHNXDb2NW#scrollTo=14e42bc3)
- **Schema Evolution**: Handles source changes by adding new columns/tables by default; can be configured to alert (e.g., Slack) on change. [Colab link](https://colab.research.google.com/drive/1C0Gt1dmJlkDa_TEqtzEHt1pmHNXDb2NW#scrollTo=acb6fb76)
- **Data Contracts**: When you "hate change", freeze the schema via `schema_contract` to control tables, columns, and `data_type` with modes like `evolve`, `freeze` (stop load), `discard_row`, or `discard_value`. Pydantic models can be used as well. [Colab link](https://colab.research.google.com/drive/1C0Gt1dmJlkDa_TEqtzEHt1pmHNXDb2NW#scrollTo=cf5897df)

### Topic 2: Hardware Bottleneck Management

Notebook 2:

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

Notebook 3:

- **Challenge**: Managing state (checkpoints) is complex.
- **Approach**:
  - Run 1: Full extract/load sets initial state
  - Run 2..n: Incremental extract since the last state
- **Write Dispositions**: `replace` (full load), `append` (stateless), `merge` (upsert with primary key), `scd2` (Type 2, `valid_from`/`valid_to`)
- **State Handling**: dlt state is a Python dict persisted in a separate destination table (more robust than orchestrator/implicit state)
- **Two Methods**:
  - Custom: `dlt.current.resource_state().setdefault(...)` and `dlt.current.resource_state()["last_updated"] = ...`
  - Cursor-based (Automated): `updated_at=dlt.sources.incremental("updated_at")`; dlt inspects yielded data, saves the last value to state, and exposes `updated_at.start_value` for the next run

### Topic 4: Bonus Round & dlt Ecosystem

- **REST API Source**: Templated source for many APIs without writing Python; configure client, auth, paginator, and resources in a dictionary.
    Docs link: 
- **Workspace Dashboard**: Browse schemas, debug pipelines, overall view.
    Docs link:
- **Ibis Integration**: Query datasets with a single, backend-agnostic API in ~20 systems (DuckDB, Snowflake, BigQuery, etc.).
    Docs link:
- **Marimo Notebooks**: Reactive notebooks saved as clean `.py` files (not `.ipynb`).
    Docs link:
- **LLM-native Scaffolding**: 4,100+ scaffolds generated from API docs to solve long-tail connectors.
    Link: 
- **LLM-native Workflow**: (1) Init scaffold, (2) Generate running code, (3) Debug in dashboard, (4) Explore in Marimo.
 - Docs link:

## Notebooks
Hands-on notebooks for each topic are in `notebooks/`:
- `notebooks/1_schema_management.ipynb`
- `notebooks/2_hardware_bottlenecks.ipynb`
- `notebooks/3_incremental_loading.ipynb`
- `notebooks/4_dlt_ecosystem.ipynb`

## Setup
1. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
2. Install dependencies:
   ```bash
   pip install -U pip
   pip install -r requirements.txt
   ```
3. Launch Jupyter:
   ```bash
   jupyter lab
   ```

