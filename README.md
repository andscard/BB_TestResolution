# ETL Pipeline en Databricks - Arquitectura y Diseño

## Descripción General

Este proyecto implementa un **pipeline ETL (Extract, Transform, Load)** en **Databricks**, siguiendo la **arquitectura Medallion** (Bronze, Silver, Gold). La solución gestiona la ingesta, transformación y almacenamiento de datos para su análisis, asegurando calidad e integridad en cada etapa.

---

## **Estructura de Archivos**

La organización de archivos en `FileStore` es la siguiente:

```
FileStore/
│── bronze/               # Almacén de datos crudos después de la ingesta
│── silver/               # Datos limpios y transformados
│── gold/                 # Datos listos para análisis y reporting
│── raw/                  # Archivos fuente en formato CSV
│── etl/
│   ├── utils/
│   │   ├── utils.ipynb       # Funciones compartidas para manejo de datos
│   ├── ingestion.ipynb   # Carga de datos y validaciones iniciales (Bronze Layer)
│   ├── transformations.ipynb  # Limpieza y procesamiento de datos (Silver Layer)
│   ├── final.ipynb       # Agregaciones y cálculos finales (Gold Layer)
│   ├── etl_master.ipynb  # Orquestación del pipeline ETL
```

---

## **Arquitectura Medallion**

El pipeline ETL está estructurado en tres capas:

### **Bronze Layer (Ingesta de Datos)**

-   **Notebook:** `ingestion.ipynb`
-   **Objetivo:** Extraer datos crudos desde archivos CSV y almacenarlos en formato **Parquet**.
-   **Validaciones:**  
    ✅ Carga de archivos CSV desde `raw/`

    ✅ Verificación de valores nulos en claves primarias

    ✅ Filtrado de registros inválidos

    ✅ Almacenamiento en `FileStore/bronze/`

### **Silver Layer (Limpieza y Normalización)**

-   **Notebook:** `transformations.ipynb`
-   **Objetivo:** Transformar y limpiar los datos para asegurar su calidad.
-   **Procesos:**  
    ✅ Conversión de tipos de datos

    ✅ Eliminación de duplicados

    ✅ Validación de integridad referencial (IDs de clientes y categorías de comercio)

    ✅ Normalización de valores categóricos

    ✅ Almacenamiento en `FileStore/silver/`

### **Gold Layer (Modelado y Agregaciones)**

-   **Notebook:** `final.ipynb`
-   **Objetivo:** Generar métricas y datos listos para análisis.
-   **Cálculos:**  
    ✅ Total de gasto por cliente

    ✅ Clasificación de riesgo crediticio

    ✅ Análisis de gasto por categoría

    ✅ Almacenamiento en `FileStore/gold/`

---

## **Orquestación del Pipeline**

Para ejecutar el pipeline ETL en orden, se usa el **notebook maestro** `etl_master.ipynb`, el cual corre los notebooks en secuencia:

```python
dbutils.notebook.run("etl/ingestion", 0)
dbutils.notebook.run("etl/transformations", 0)
dbutils.notebook.run("etl/final", 0)
```

---

## **Decisiones de Diseño**

### **Uso de `Parquet` en lugar de `CSV`**

-   **Ventaja:** Mejor rendimiento y compresión.
-   **Motivo:** Spark maneja `Parquet` más eficientemente que `CSV`, reduciendo tiempos de lectura y escritura.

### **Manejo de errores y logs**

-   Logs con `logging` en cada notebook para rastrear errores y progreso.
-   Manejo de errores en lectura de archivos usando `try-except`.
-   Verificación de claves primarias nulas para asegurar la calidad del dato.

### **Uso de `utils.ipynb` para reutilización de código**

-   **Objetivo:** Evitar código duplicado y centralizar funciones comunes.
-   **Funciones incluidas:**

    ```python
    def load_parquet(path):
        """Carga archivos Parquet con manejo de errores."""
        ...

    def upload_csv(ruta):
        """Carga archivos CSV con validaciones."""
        ...
    ```

-   **Modo de importación en los notebooks:**
    ```python
    dbutils.notebook.run("etl/utils", 0)
    ```

### **Uso de `dbutils.notebook.run()` para ejecución modular**

Cada notebook es ejecutado secuencialmente, permitiendo una arquitectura flexible y escalable.

---

## **Ejecución del Pipeline ETL**

Para ejecutar el proceso completo en Databricks:

1. **Abrir `etl_master.ipynb`** en Databricks.
2. **Ejecutar todas las celdas.**

Esto procesará los datos desde la ingesta hasta su almacenamiento final en `gold/`.

---

## **Posibles Mejoras Futuras**

-   **Particionamiento:** Dividir de forma eficiente columnas de fechas (year, month, day of week).

-   **Automatización:** Configurar Databricks Workflows para encadenar los notebooks y ejecutarlos automáticamente.

-   **Optimización:** Uso de Z-Ordering en columnas de alta cardinalidad para mejorar la compresión y las búsquedas en Delta Lake.

-   **Monitoring:** Usar API de Databricks para rastrear el estado y rendimiento de los pipelines. Envío de correos.

-   **Caching:** Usar df.cache() para evitar leer los datos varias veces.
