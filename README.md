# Graph Repairing under Neighborhood Constraints

This repository contains code, notebooks and data for experiments and utilities used in the paper "Graph repairing under neighborhood constraints" (Song et al., 2017). The project implements and compares several label-repair algorithms (APOC-based greedy repair, contraction variants, etc.) on small graph datasets (restaurants, Cora).

Contents
- Notebooks: example experiments and import/cleaning pipelines (`restaurants_pipeline.ipynb`, `restaurants_pipeline_apoc.ipynb`, `cora_*.ipynb`, `algorithms.ipynb`).
- Data: original and processed data under `datasets/`.
- Code: helper functions embedded in notebooks (repair algos, Neo4j import helpers, evaluation metrics).

Prerequisites
- Python 3.12+ (pyproject specifies >=3.12)
- Neo4j 5.x (or compatible with the `neo4j` Python driver used here)
- APOC procedures installed on the Neo4j instance if you intend to run APOC-powered notebooks (`restaurants_pipeline_apoc.ipynb`).

Recommended local environment setup
1. Create and activate a virtual environment:

```powershell
python -m venv .venv
# Graph Repairing under Neighborhood Constraints

This repository contains notebooks, data and helper code used to reproduce experiments and utilities from research on label-repair under neighborhood constraints (Song et al., 2017). The notebooks implement and evaluate repair strategies (APOC greedy, contraction variants) on two example datasets: `Restaurant` and `Cora`.

## Contents
- Notebooks: import/cleaning pipelines and experiments (`restaurants_pipeline.ipynb`, `restaurants_pipeline_apoc.ipynb`, `cora_import.ipynb`, `cora_data_cleaning.ipynb`, `algorithms.ipynb`).
- Data: original and preprocessed data under `datasets/`.
- Helpers: functions embedded in notebooks (Neo4j import, repair algorithms, evaluation metrics).

## Prerequisites
- Python 3.12+
- Neo4j 5.x (or compatible with the `neo4j` Python driver)
- (Optional) APOC procedures installed on Neo4j to run APOC-based notebooks

## Quick environment setup
1. Create and activate a virtual environment:

```powershell
python -m venv .venv
.\.venv\Scripts\activate
```

2. Install dependencies:

```powershell
pip install -e .
# or
pip install easydict ipykernel neo4j networkx pandas python-dotenv matplotlib python-Levenshtein
```

3. Environment variables
- Copy `example.env` to `.env` and edit values. Key variables used by the notebooks:
  - `NEO4J_URI` (e.g. `bolt://localhost:7687`)
  - `NEO4J_USERNAME`, `NEO4J_PASSWORD`
  - `NEO4J_CONSTRAINT_DB`, `NEO4J_INSTANCE_DB`
  - `FRAUD_NUMBER` (optional; used by Cora notebooks to set swap count)

## Datasets — Homogeneous Documentation
Below each dataset section follows the same structure for clarity: Overview, Files, Generated artifacts, Notebooks, Quick run, Node/edge mapping, Troubleshooting.

**Restaurant**

- Overview
  - Small example dataset derived from Sheila Tejada's restaurant records. Used as a compact, illustrative dataset for repair experiments (label = `area_code`).

- Files
  - `datasets/restaurant/original/` — original raw files (fodors.txt, match-pairs.txt, zagats.txt).
  - `datasets/restaurant/fz.arff`, `datasets/restaurant/fz-nophone.arff` — ARFF variants included for historical experiments.

- Generated artifacts (in `datasets/temp/`)
  - `restaurants_YYYYMMDD-HHMMSS.txt` — cleaned restaurant records (id, name, area_code, addr, city).
  - `restaurant_similarities_YYYYMMDD-HHMMSS.txt` — similarity/pair files used to build SIMILAR edges.
  - `*_cleaned_*.txt` — cleaned versions used by import pipelines.

- Notebooks
  - `restaurants_pipeline.ipynb` — full non-APOC pipeline: import GT and INSTANCE databases, inject label noise, run contraction-based repair algorithms (`contraction_internal`, `contraction_smart`), evaluate results.
  - `restaurants_pipeline_apoc.ipynb` — APOC-based greedy repair variant; requires APOC plugin in Neo4j.

- Quick run
  1. Configure `.env` (see `example.env`) with Neo4j credentials and DB names.
  2. Start Jupyter and open `restaurants_pipeline.ipynb`.
  3. Run cells in order: imports → load artifacts (or generate via cleaning notebook) → import GT and INSTANCE → inject noise → run repair algos → evaluate.

- Node / Edge mapping
  - Nodes: `Restaurant` nodes with properties `id`, `name`, `area_code`, `addr`, `city`, and `label` (set to area code in GT).
  - Edges: undirected similarity stored as two directed `:SIMILAR` relationships.

- Troubleshooting
  - If import cells fail due to missing files, re-run the cleaning steps or check `datasets/temp/` for timestamped artifacts.
  - APOC cells require APOC installed and the user to have permission to run procedures.

**Cora**

- Overview
  - The Cora bibliographic dataset (papers/authors/co-authorship) used to demo graph import, perturbation and repair evaluation on a citation/co-author graph.

- Files
  - `datasets/cora/cora.arff` — original ARFF file (attributes + `class`).
  - `datasets/cora/cora.txt` — plain-text export of bibliographic lines used during preprocessing.
  - `datasets/cora/cora_modified.txt` — cleaned/compact form expected by the notebooks (produced by `cora_data_cleaning.ipynb`).

- Generated artifacts (in `datasets/temp/`)
  - `authors_YYYYMMDD-HHMMSS.txt` — list of author/paper identifiers produced by the cleaning notebook.
  - `author_connections_YYYYMMDD-HHMMSS.txt` — lines like `(authorA,authorB)` representing co-authorship pairs; the import notebook picks the latest by timestamp.

- Notebooks
  - `cora_data_cleaning.ipynb` — parse `cora.arff` / `cora.txt`, normalize fields, and write timestamped `authors_*` and `author_connections_*` artifacts to `datasets/temp/`.
  - `cora_import.ipynb` — auto-detects the latest generated artifacts, imports a canonical constraint graph and an instance graph into Neo4j, and can apply random swaps/perturbations to simulate noise.

- Quick run
  1. Configure `.env` (see `example.env`) and optionally set `FRAUD_NUMBER` for swap count.
  2. Run `cora_data_cleaning.ipynb` to generate `authors_*` and `author_connections_*` in `datasets/temp/`.
  3. Open `cora_import.ipynb` and run cells in order; it will print the `AUTHORS_PATH` and `CONNECTIONS_PATH` it uses.

- Node / Edge mapping
  - Nodes: authors (or paper IDs depending on the cleaning step) with identifying properties and optional `label` properties used in evaluation.
  - Edges: co-authorship pairs (imported as undirected connections represented by two directed edges where appropriate).

- Troubleshooting
  - If `cora_import.ipynb` raises `FileNotFoundError` for `AUTHORS_PATH` / `CONNECTIONS_PATH`, re-run `cora_data_cleaning.ipynb` to regenerate artifacts.
  - Ensure Neo4j user can create constraints and write to the chosen DB names.

## Notes on Neo4j & APOC
- APOC-based notebooks call `apoc.periodic.commit(...)`. If you plan to run them:
  - Install and enable APOC for your Neo4j instance.
  - Ensure procedure permissions are available for the DB user.

## Common pitfalls & troubleshooting (general)
- KeyError for missing columns: notebooks sometimes produce `repair_cost` or `repair_cost_num_relabels`; updated code now handles both.
- Neo4j connectivity: verify `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD` in `.env` and that databases exist.

Enjoy exploring the experiments!

