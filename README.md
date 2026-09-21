# Kineret-ETL

Preprocessing pipeline that turns raw **OMOP CDM** hospital data (Kineret / Rambam cohort) into the tabular input consumed downstream by the **Mediator** temporal-abstraction stage. This is the ETL half of the work described in the JAMIA paper and in the thesis — everything up to, but not including, the prediction phase.

## What this repo contains

- **`db_load.ipynb`** — reads raw OMOP tables (condition / observation / procedure / device / drug / measurement / visit / person / death), unifies schemas, filters invalid concepts, builds dimension tables (concepts, units, drug routes), and runs measurement-unit normalization rules. Output: cleaned per-table parquet files + dimension tables.
- **`data_exploration.ipynb`** — cohort assembly and mapping from OMOP concept IDs to the higher-level clinical concepts used by Mediator. Applies the cohort filters and the concept-mapping rules under `index/` to emit the final event tables in Mediator's expected shape.
- **`index/`** — code-to-concept mappings exported from the VM. These files are inputs to `data_exploration.ipynb` (`load_index_specs` + cohort loaders):
  - `index/cohort_filters/` — OMOP `concept_id` lists that define patient inclusion (`include_DD.txt` for diabetes diagnoses, `include_GLUCOSE_TEST.txt` for glucose measurements).
  - `index/explorations/{clinical_events,drug_exposure,measurement}/*.txt` — one file per Mediator concept (e.g. `HEMOGLOBIN-A1C_MEASURE`, `METFORMIN_HOME_BITZUA`, `RETINOPATHY`), each holding the OMOP `concept_id`s + rules that collapse into that concept.

## Pipeline position

```
OMOP CDM (raw)  →  [db_load]  →  cleaned OMOP + dimensions
               →  [data_exploration + index/]  →  Mediator input
               →  Mediator (temporal abstraction)  →  prediction models
```

Only the two boxed stages live here. Downstream Mediator and prediction code are in separate repos.

## Notes

- Paths inside the notebooks (`/home/jovyan/workspace/data/...`) reflect the JupyterHub VM where they were run; adjust to your environment.
- These notebooks are archived as-is for reproducibility. For methodology, cohort definitions, and rationale behind the concept groupings, see the thesis and the JAMIA paper.
