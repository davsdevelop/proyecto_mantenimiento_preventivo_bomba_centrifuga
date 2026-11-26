# Contexto del proyecto — Mantenimiento preventivo de bomba centrífuga

## Objetivo

Construir un flujo ETL reproducible que extraiga, limpie, transforme y valide datos operativos de una **bomba centrífuga**, con el fin de preparar una base de datos analítica para el estudio de su mantenimiento preventivo.

## Alcance del Volumen 1

El Volumen 1 cubre exclusivamente el proceso ETL:

1. Extracción de las tablas desde una base PostgreSQL alojada en un contenedor Docker.
2. Inspección inicial de cada dataset con pandas.
3. Limpieza y transformación justificadas.
4. Validación del resultado final.
5. Persistencia en archivos Parquet listos para los siguientes volúmenes.

El análisis exploratorio profundo, el modelado y la visualización quedan fuera de este volumen.

## Fuentes de datos

| Tabla | Tipo | Ubicación | Estado |
|---|---|---|---|
| `catalogo_fallas` | PostgreSQL (Docker) | `data/raw/catalogo_fallas_raw.csv` | Procesada |
| `equipos` | PostgreSQL (Docker) | `data/raw/equipos_raw.csv` | Procesada |
| `lecturas_sensores` | PostgreSQL (Docker) | `data/raw/lecturas_sensores_raw.csv` | Procesada |
| `ordenes_mantenimiento` | PostgreSQL (Docker) | `data/raw/ordenes_mantenimiento_raw.csv` | Procesada |

Los parámetros de conexión se definen en `.env`. El rango temporal de las lecturas va del `2026-08-01` al `2026-10-31`.

## Estructura del proyecto

```text
proyecto/
├── data/
│   ├── raw/              # Snapshots CSV de las tablas
│   ├── interim/          # Datos intermedios, si aplica
│   └── processed/        # Parquet finales del ETL
├── notebooks/            # EDA adicional
├── src/
│   ├── data/
│   │   ├── ETL_catalogo_fallas.ipynb
│   │   ├── ETL_equipos.ipynb
│   │   ├── ETL_lecturas_sensores.ipynb
│   │   └── ETL_ordenes_mantenimiento.ipynb
│   └── models/           # Reservado para volúmenes posteriores
├── .env.example
├── requirements.txt
├── README.md
└── project_context.md
```

## Arquitectura y decisiones

- **Un notebook por tabla:** cada tabla de la fuente tiene su propio notebook en `src/data/`.
- **Acceso a datos:** `SQLAlchemy` como motor de conexión y `psycopg2-binary` como driver de PostgreSQL; la extracción se ejecuta con `pandas.read_sql`.
- **Snapshot en raw:** las tablas se guardan como CSV en `data/raw/` para conservar trazabilidad y reprocesar sin volver a consultar la base.
- **Salida en Parquet:** un archivo por tabla en `data/processed/`, con escritura atómica mediante archivo temporal.
- **Nombres de columnas normalizados** (minúsculas, sin espacios) para facilitar el trabajo posterior.
- **Fechas y números normalizados:** en `lecturas_sensores` se corrigen los dos formatos de fecha y los valores de temperatura con coma decimal y símbolo `°C`.
- **Sin imputación automática:** los nulos se reportan y solo se tratan cuando exista una regla de negocio definida.
- **Configuración por entorno:** credenciales y parámetros de conexión fuera del código, en `.env`.

## Resultados actuales

- Los cuatro notebooks ejecutan de principio a fin sin errores.
- Parquet generados en `data/processed/`:

| Archivo | Filas | Nulos |
|---|---:|---:|
| `catalogo_fallas_clean.parquet` | 4 | 0 |
| `equipos_clean.parquet` | 1 | 0 |
| `lecturas_sensores_clean.parquet` | 135,130 | 16,034 |
| `ordenes_mantenimiento_clean.parquet` | 1 | 0 |

- En `lecturas_sensores` los nulos restantes provienen de la fuente (`temp_carc`, `vibracion_mms`, `pres_desc`, `t_amb`); no se introdujeron nulos por errores de parseo.
- No se detectaron duplicados en los datasets procesados.

## Observaciones y pendientes

- Definir reglas de negocio para los nulos de las lecturas de sensores.
- Revisar valores atípicos y rangos de las variables numéricas.
- Confirmar si las tablas deben integrarse en un dataset analítico único para los siguientes volúmenes.

## Próximos volúmenes

- Análisis exploratorio de los datos preparados.
- Estudio de patrones operativos y de mantenimiento.
- Modelado predictivo orientado al mantenimiento preventivo.
