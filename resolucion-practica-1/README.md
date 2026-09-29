# Entrega · 01_ingesta_bronze

- **Nombre:** Jeremías
- **`student_id`:** `jmena`
- **Namespace:** `workspace.bigdata_jmena`

## Resultados de la ingesta

| Tabla Bronze | Formato de origen | Filas |
|---|---|---:|
| `workspace.bigdata_jmena.bronze_customers` | CSV | 5,000 |
| `workspace.bigdata_jmena.bronze_products` | Parquet | 500 |
| `workspace.bigdata_jmena.bronze_transactions` | CSV | 50,011 |
| `workspace.bigdata_jmena.bronze_events` | JSON | 200,000 |

Diagnóstico de calidad sobre `bronze_transactions`:

| rows | distinct_ids | invalid_amounts |
|---:|---:|---:|
| 50,011 | 50,000 | 52 |

Existen 11 filas sobrantes respecto de los IDs distintos y 52 valores de `amount` que la sentencia `try_cast(amount AS DECIMAL(12,2))` no logra convertir.

---

## 1. Tres observaciones sobre CSV/JSON, Parquet y Delta

**1. CSV y JSON no traen un esquema confiable** 
En el formato CSV todo se lee como texto. Por eso `amount` queda como `string`, y el diagnóstico lo confirma: 52 valores no son convertibles a número, así que ninguna inferencia logra tipar esta columna como numérica sin antes perder datos. El parámetro `inferSchema=true` obliga a Spark a hacer otra pasada sobre los datos y produce un resultado que depende de lo que haya en ellos, por lo que puede variar si estos datos se ven afectados de alguna forma. En el formato JSON el esquema también se infiere recorriendo los registros. Con estos formatos conviene declarar el esquema explícitamente.

**2. Parquet guarda el esquema y los datos en formato columnar** 
`products_raw` se leyó con sus tipos correctos sin inferir nada porque el esquema ya viene dentro del archivo. Además permite leer solo las columnas necesarias y usa estadísticas para saltear datos. Su límite es que sigue siendo un directorio de archivos sin registro de transacciones: no ofrece historial de versiones, garantías de atomicidad ni control de cambios de esquema.

**3. Delta es Parquet más un log de transacciones** 
Ese log agrega lo que un directorio Parquet no tiene: `DESCRIBE HISTORY` muestra cada operación sobre la tabla, `DESCRIBE DETAIL` nos brinda información sobre el formato, la cantidad de archivos y el tamaño, y hay transacciones ACID, control de esquema y posibilidad de volver a versiones anteriores. Delta no mejora la calidad del dato: las tablas Bronze conservan `amount` como texto y los 11 duplicados tal como llegaron, pero agrega trazabilidad a las tablas.

---

## 2. Dónde aparece cada una de las cinco V

**Volumen** 
Con la escala usada, el volumen se reparte de forma muy desigual: `bronze_events` tiene 200,000 filas, `bronze_transactions` 50,011, `bronze_customers` 5,000 y `bronze_products` 500. Las tablas de hechos (events y transactions) concentran casi todo el volumen y son las que más crecen.

**Velocidad** 
Aparece en `events_json` y en `transactions`, datos que un sistema real genera de forma continua. En este notebook la ingesta es por lotes, por lo que la velocidad es más conceptual y no tan real. Lo que sí queda registrado es la columna `_ingested_at`, que marca cuándo llegó cada dato.

**Variedad** 
Son cuatro fuentes en tres formatos: CSV (customers y transactions), Parquet (products) y JSON (events). Hay datos estructurados y semi-estructurados. La ingesta los unifica como tablas Delta con metadatos comunes (`_source`, `_source_file`, `_ingested_at`).

**Veracidad** 
Es la V que el notebook mide de forma explícita: `bronze_transactions` tiene 50,011 filas pero solo 50,000 `transaction_id` distintos y 52 importes no convertibles a número. Esta estapa (Bronze) registra el problema sin corregirlo y los metadatos de origen e ingesta permiten rastrear de dónde vino cada registro afectado.

**Valor** 
En esta estapa no encuentra el valor como tal. Bronze aporta valor como base auditable y reproducible, porque conserva el dato original y permite reprocesar, pero aún falta la parte analítica.