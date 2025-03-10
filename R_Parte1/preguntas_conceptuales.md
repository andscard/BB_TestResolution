## Parte 1: Preguntas Conceptuales (20%)

### 1.1. Arquitectura Medallion

#### Describa detalladamente los niveles de la arquitectura Medallion en Databricks (Bronze, Silver, Gold), explicando:

-   **Bronze:** Almacena datos no validados y sin procesar de todas las fuentes, lo cual permite llevar una trazabilidad de la información real y sin modificar que se extrajo. Se le suelen adicionar metadados y pueden ser guardados en formatos JSON, CSV. En esta capa no se realizan validaciones ni limpieza de los datos, por lo que pueden estar duplicados o con valores nulos.
-   **Silver:** Se realiza limpieza y validaciones de datos para. Se suelen aplicar normalizaciones en loda formatos y aplicar reglas de negocios, con ele objetivo de que estos sean mas estructurados. Es aquí donde se corrigen errores e inconsistencias, de modo que se tengan datos más integros y coherentes.
-   **Gold:** Proporciona datos que están listos para ser consumidos en análisis avanzados, reportes y modelos de IA. Usualmente en esta capa se crean agregaciones y cálculo derivado, tambien se pueden crear vistas optimizadas para integrases con herrramientas de BI o aprendizaje automatico. En este punto los datos deben ser altamente confiables, no deben existir incosistencias y se alinean perfectamente as los requisitos del negocio.

Esta arquitectura organiza la información en niveles (bronce, plata y oro), asegurando que los datos evolucionen desde su estado más crudo hasta una versión confiable y optimizada. Esto es gracias a la gestión estructurada y progresiva de los datos, que mejora su calidad en cada etapa.

### 1.2. Optimización en Spark

#### Explique las siguientes técnicas de optimización en Spark y cuándo usaría cada una:

-   **Particionamiento vs Bucketing**

    **Particionamiento** divide los datos de multiples archvios según el valor de de una columna, almacenándolos en diferentes carpetas. Se usa cuando se consultan frecuentemente datos filtrados por esa columna, reduciendo la lectura innecesaria.
    **Bucketing** agrupa los datos en un número fijo de archivos dentro de cada partición, distribuyéndolos de manera uniforme. Es útil cuando se requieren operaciones de join o group by eficientes, ya que reduce el shuffling.

-   **Cache vs Persist**

    **Cache** almacena los datos en memoria (RAM) y se usa cuando los datos se reutilizarán varias veces en la misma ejecución (consultas repetidas).
    **Persist** permite elegir dónde almacenar los datos (RAM, disco o ambos) dependiendo del nivel de persistencia configurado. Es usado cuando los datos son grandes y la memoria RAM es insuficiente.

-   **Broadcasting**
    Envía copias pequeñas de un dataframe a todos los nodos del clúster, evitando shuffling en joins. Se usa cuando se unen tablas grandes con otras más pequeñas.

-   **Repartitioning**
    Redistribuye los datos en n particiones, útil cuando hay un desbalance en el procesamiento. Se lo usa antes de operaciones costosas como joins o agregaciones para equilibrar la carga.

### 1.3. Delta Lake

#### ¿Qué ventajas ofrece Delta Lake sobre formatos tradicionales como Parquet? Explique conceptos como Time Travel, Schema Enforcement y Evolution, Transacciones ACID, Compactación y Z-Ordering:

Delta Lake ofrece capacidades avanzadas para mejorar la confiabilidad, gobernanza y rendimiento del almacenamiento de datos en comparación con otros formatos.

-   **Time Travel:** Permite acceder a versiones históricas de los datos mediante VERSION AS OF o TIMESTAMP AS OF. Es usado para recuperar datos eliminados o modificados accidentalmente, y para ver la trazabilidad de estos.

-   **Schema Enforcement y Evolution:** Enforcement impide que datos incompatibles se inserten en la tabla, mientras que evolution permite modificar el esquema sin afectar la integridad.

-   **Transacciones ACID:** conjunto de propiedades que garantizan Atomicidad, Consistencia, Aislamiento y Durabilidad (ACID) de los datos. Permiten procesamiento concurrente sin riesgo de corrupción y garantizan que las cargas y actualizaciones sean completas y eficientes.

-   **Compactación y Z-Ordering:** Compactación fusiona archivos pequeños en archivos mas grandes para mejorar la eficiencia de lectura, mientras que Z-Ordering optimiza la distribución de datos en disco, ordenando por una columna clave para mejorar el pruning en consultas.

### 1.4. Modelado Dimensional

#### Compare y contraste los esquemas Estrella y Copo de Nieve. ¿Cuándo emplearía cada uno en un entorno bancario y por qué?

| Característica               | Esquema Estrella                                                      | Esquema Copo de Nieve                                              |
| ---------------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **Estructura**               | Tabla de hechos conectada directamente a dimensiones desnormalizadas. | Tabla de hechos con dimensiones normalizadas en múltiples niveles. |
| **Rendimiento de consultas** | Más rápido, menos _joins_ (dimensiones más anchas).                   | Más lento, más _joins_ necesarios para recuperar datos.            |
| **Almacenamiento**           | Mayor redundancia de datos.                                           | Menos redundancia, pero más complejidad.                           |
| **Facilidad de uso**         | Más simple para analistas y herramientas de BI.                       | Más complejo, requiere mayor comprensión de la estructura.         |

En el entorno bancario el esquema estrella sería útil para reportes operativos, visualizaciones y análisis de transacciones en tiempo real, mientras que el esquema copo de nieve podría ser usado para análisis detallados como auditorías o modelos de caracter normativo.

### 1.5. Jobs y Workflows en Databricks

#### Explique cómo implementaría un workflow con dependencias en Databricks, las mejores prácticas para programar jobs, y cómo manejaría fallos y reintentos.

A continuación detallo los pasos para implementar un workflow con dependencias en Databricks:

1. Definir los Jobs: Crearía jobs que ejecuten notebooks, scripts Python, JARs o SQL, asegurando modularidad y reutilización.
2. Configurar dependencias: Usaría el Task Graph para establecer el orden de ejecución, garantizando que cada tarea dependa de la correcta finalización de sus predecesoras.
3. Programar el Workflow: Configuraría la ejecución en base a una programación periódica o activadores como cambios en tablas Delta.

**Mejores Prácticas**

-   Dividir las tareas en pasos independientes para facilitar la depuración y escalabilidad.
-   Pasar parámetros entre tareas para mejorar la flexibilidad del workflow.
-   Usar clusters optimizados para cada job, reduciendo costos y asegurando aislamiento.
-   Habilitar logs y monitoreo con alertas.

**Manejo de Fallos y reintentos**

-   Configurar reintentos automáticos en Databricks, estableciendo el número máximo de intentos y tiempo de espera.
-   Manejo de errores en notebooks, usando try-except en Python o IF ERROR THEN en SQL para capturar fallos y enviar notificaciones.
