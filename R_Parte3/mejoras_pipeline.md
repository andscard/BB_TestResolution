## MEJORAS A FUTURO

### Particionamiento

-   Dividir de forma eficiente columnas de fechas (year, month, day of week).

### Manejo de errores

-   Usar sentencia try-except para capturar y registrar cualquier fallo que ocurra durante el proceso ETL.

### Automatización

-   Configurar Databricks Workflows para encadenar los notebooks y ejecutarlos automáticamente.

### Optimización

-   Uso de Z-Ordering en columnas de alta cardinalidad para mejorar la compresión y las búsquedas en Delta Lake.

### Monitoring y Logging

-   Usar API de Databricks para rastrear el estado y rendimiento de los pipelines. Envío de correos.

### Caching

-   Usar df.cache() para evitar leer los datos varias veces.
