# ETL Pipeline on Azure for Mexico's COVID-19 Open Epidemiological Data (2020-2022)

Design and implementation of an end-to-end pipeline on Microsoft Azure for the processing and standardization of the historical annual open data files on COVID-19 from Mexico's Ministry of Health (SSA), as part of a paired comparative study against a local/legacy Python/pandas workflow.

## Context

This project is part of my Master's thesis in Data Science (Universidad Vasco de Quiroga), whose objective is to design, implement, and evaluate an automated pipeline on Azure for processing the SSA's annual files (`COVID19MEXICO2020.csv`, `COVID19MEXICO2021.csv`, `COVID19MEXICO2022.csv`), measuring differences in technical latency, compliance with the official schema, and data quality, against traditional local processing with pandas.

## Architecture

```
SSA (open data)
    │
    ▼
Azure Data Factory (automated extraction, pipeline parameterized by year)
    │
    ▼
Azure Data Lake Storage Gen2 — Bronze layer (raw ZIP files)
    │
    ▼
Azure Databricks / PySpark (cleansing: 9 validation rules)
    │
    ▼
Azure Data Lake Storage Gen2 — Silver layer (clean data + execution log)
    │
    ▼
Azure Data Lake Storage Gen2 — Gold layer (data ready for analytical consumption)
    │
    ▼
Power BI (visualization)
```

## Repository Components

| Folder / file | Description |
|---|---|
| `/notebooks/` | Databricks notebooks (PySpark) for the 9 processing runs (3 years × 3 repetitions): loading from Bronze, applying the 9 cleansing rules, computing quality and throughput metrics, and writing to Silver/Gold |
| `/data-factory/` | JSON definition of the Azure Data Factory `source_prep` pipeline, which automates the year-parameterized download of the SSA source files into the Bronze layer |
| `README.md` | This document |

## Technologies Used

- **Azure Data Factory** — automated, parameterized extraction of source files
- **Azure Data Lake Storage Gen2** — storage in a medallion architecture (Bronze / Silver / Gold), managed with Unity Catalog
- **Azure Databricks** (Serverless) — distributed processing with PySpark (Apache Spark 4.2.0)
- **Power BI** — visualization and analytical consumption of the Gold layer

## Cleansing Rules Applied (identical to the local comparison condition)

1. `FECHA_SINTOMAS` (symptom onset date) not null
2. `FECHA_ACTUALIZACION` (update date) not null
3. Valid `SECTOR` (4=IMSS, 6=ISSSTE, 12=SSA)
4. `CLASIFICACION_FINAL` (final classification) in {1, 3} (lab-confirmed or clinical-epidemiological determination)
5. Filter by the period of the evaluated year
6. Latency calculation (`FECHA_ACTUALIZACION − FECHA_SINTOMAS`)
7. Exclusion of negative latency
8. Exclusion of extreme latency (>1,095 days)
9. Deduplication by logical key, keeping the record with the highest completeness

## Metrics Recorded per Run

- Technical processing latency (`duracion_min`)
- Throughput (`registros_finales / duracion_min`)
- Completeness, duplicates, and consistency after cleansing
- Structural compliance (fields vs. the official 39-field catalog)
- Record compliance by institution (SECTOR)
- Data-to-visualization latency in Power BI

## Results

The complete results, covering the 9 processing runs (3 years × 3 repetitions) and their comparison against the local pandas workflow, are documented in the research paper associated with this project.

## Author

**Arturo Ramírez Cíntora**
Master's in Data Science — Universidad Vasco de Quiroga
Electronic Engineer

## Note

The processed data is publicly available, published by Mexico's Ministry of Health through its open data portal: https://www.gob.mx/salud/documentos/datos-abiertos-152127
