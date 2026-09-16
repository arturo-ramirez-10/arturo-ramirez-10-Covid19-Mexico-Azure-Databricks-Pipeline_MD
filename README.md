# Pipeline ETL en Azure para Datos Epidemiológicos Abiertos de COVID-19 México (2020-2022)

Diseño e implementación de un pipeline end-to-end en Microsoft Azure para el procesamiento y estandarización de los archivos históricos anuales de datos abiertos de COVID-19 de la Secretaría de Salud de México (SSA), como parte de un estudio comparativo pareado frente a un flujo local/legado con Python/pandas.

## Contexto

Este proyecto forma parte de mi tesis de Maestría en Ciencia de Datos (Universidad Vasco de Quiroga), cuyo objetivo es diseñar, implementar y evaluar un pipeline automatizado en Azure para el procesamiento de los archivos anuales de la SSA (`COVID19MEXICO2020.csv`, `COVID19MEXICO2021.csv`, `COVID19MEXICO2022.csv`), midiendo diferencias en latencia técnica, conformidad con el esquema oficial y calidad del dato, frente al procesamiento local tradicional con pandas.

## Arquitectura

```
SSA (datos abiertos)
    │
    ▼
Azure Data Factory (extracción automatizada, pipeline parametrizado por año)
    │
    ▼
Azure Data Lake Storage Gen2 — capa Bronze (archivos ZIP crudos)
    │
    ▼
Azure Databricks / PySpark (depuración: 9 reglas de validación)
    │
    ▼
Azure Data Lake Storage Gen2 — capa Silver (datos limpios + bitácora de ejecución)
    │
    ▼
Azure Data Lake Storage Gen2 — capa Gold (datos listos para consumo analítico)
    │
    ▼
Power BI (visualización)
```

## Componentes del repositorio

| Carpeta / archivo | Descripción |
|---|---|
| `/notebooks/` | Notebooks de Databricks (PySpark) para las 9 corridas de procesamiento (3 años × 3 repeticiones): carga desde Bronze, aplicación de las 9 reglas de depuración, cálculo de métricas de calidad y throughput, y escritura en Silver/Gold |
| `/data-factory/` | Definición JSON del pipeline `source_prep` de Azure Data Factory, que automatiza la descarga parametrizada por año de los archivos fuente de la SSA hacia la capa Bronze |
| `README.md` | Este documento |

## Tecnologías utilizadas

- **Azure Data Factory** — extracción automatizada y parametrizada de los archivos fuente
- **Azure Data Lake Storage Gen2** — almacenamiento en arquitectura medallón (Bronze / Silver / Gold), gestionado con Unity Catalog
- **Azure Databricks** (Serverless) — procesamiento distribuido con PySpark (Apache Spark 4.2.0)
- **Power BI** — visualización y consumo analítico de la capa Gold

## Reglas de depuración aplicadas (idénticas a la condición de comparación local)

1. `FECHA_SINTOMAS` no nula
2. `FECHA_ACTUALIZACION` no nula
3. `SECTOR` válido (4=IMSS, 6=ISSSTE, 12=SSA)
4. `CLASIFICACION_FINAL` en {1, 3} (confirmado por laboratorio o dictaminación clínico-epidemiológica)
5. Filtro por periodo del año evaluado
6. Cálculo de latencia (`FECHA_ACTUALIZACION − FECHA_SINTOMAS`)
7. Exclusión de latencia negativa
8. Exclusión de latencia extrema (>1,095 días)
9. Deduplicación por llave lógica, conservando el registro de mayor completitud

## Métricas registradas por corrida

- Latencia técnica de procesamiento (`duracion_min`)
- Throughput (`registros_finales / duracion_min`)
- Completitud, duplicados y consistencia post-depuración
- Conformidad estructural (campos vs. catálogo oficial de 39 campos)
- Conformidad de registros por institución (SECTOR)
- Latencia dato → visualización en Power BI

## Resultados

Los resultados completos, con las 9 corridas de procesamiento (3 años × 3 repeticiones) y su comparación frente al flujo local con pandas, se documentan en el artículo de investigación asociado a este proyecto.

## Autor

**Arturo Ramírez Cíntora**
Maestría en Ciencia de Datos — Universidad Vasco de Quiroga
Ingeniero Electrónico

## Nota

Los datos procesados son de acceso público, publicados por la Secretaría de Salud de México a través de su portal de datos abiertos: https://www.gob.mx/salud/documentos/datos-abiertos-152127
