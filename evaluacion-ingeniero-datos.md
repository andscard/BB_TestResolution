# Evaluación Técnica: Ingeniero de Datos Semi-Senior
## Banco Bolivari

### Instrucciones Generales
- La evaluación tiene una duración máxima de 3 horas.
- Se valorará tanto la funcionalidad como la calidad del código, así como la documentación y justificación de decisiones tomadas.
- Puede utilizar cualquier recurso online excepto IA generativa o solicitar ayuda a terceros.
- Entregue su solución en un repositorio Git (GitHub/GitLab/BitBucket) con instrucciones claras para su ejecución.
- Se requiere un README detallado que explique la arquitectura de la solución y las decisiones de diseño.

### Estructura de la Evaluación
- **Parte 1**: Preguntas conceptuales. (20%)
- **Parte 2**: Caso práctico de modelado de datos. (30%)
- **Parte 3**: Implementación de pipeline de datos con PySpark. (50%)

---

## Parte 1: Preguntas Conceptuales (20%)

### 1.1. Arquitectura Medallion
Describa detalladamente los niveles de la arquitectura Medallion en Databricks (Bronze, Silver, Gold), explicando:
- El propósito de cada nivel.
- Las transformaciones típicas en cada nivel.
- Consideraciones de calidad de datos en cada nivel.
- Ventajas de este enfoque frente a arquitecturas tradicionales.

### 1.2. Optimización en Spark
Explique las siguientes técnicas de optimización en Spark y cuándo usaría cada una:
- Particionamiento vs Bucketing.
- Cache vs Persist.
- Broadcasting.
- Repartitioning.

### 1.3. Delta Lake
¿Qué ventajas ofrece Delta Lake sobre formatos tradicionales como Parquet? Explique conceptos como:
- Time Travel.
- Schema Enforcement y Evolution.
- Transacciones ACID.
- Compactación y Z-Ordering.

### 1.4. Modelado Dimensional
Compare y contraste los esquemas Estrella y Copo de Nieve. ¿Cuándo emplearía cada uno en un entorno bancario y por qué?

### 1.5. Jobs y Workflows en Databricks
Explique cómo implementaría un workflow con dependencias en Databricks, las mejores prácticas para programar jobs, y cómo manejaría fallos y reintentos.

---

## Parte 2: Caso práctico de modelado de datos (30%)

### Contexto
El banco está diseñando un nuevo data mart para analizar el comportamiento de uso de tarjetas de crédito. Se busca segmentar clientes para campañas personalizadas y detectar patrones de gasto que permitan ofrecer productos adecuados a cada perfil.

### Datos disponibles
Se tiene acceso a las siguientes fuentes de datos:

1. **Transacciones de tarjetas de crédito** (actualización diaria, ~500,000 registros/día):
   - transaction_id
   - card_id
   - transaction_date
   - merchant_id
   - merchant_category
   - transaction_amount
   - transaction_currency
   - is_international
   - is_online

2. **Clientes** (actualización semanal, ~2,000,000 registros):
   - customer_id
   - customer_name
   - customer_age
   - customer_gender
   - customer_address
   - customer_city
   - customer_province
   - registration_date
   - income_segment

3. **Tarjetas** (actualización diaria, ~3,000,000 registros):
   - card_id
   - customer_id
   - card_type
   - card_limit
   - issue_date
   - expiry_date
   - credit_score
   - status

4. **Comercios** (actualización mensual, ~100,000 registros):
   - merchant_id
   - merchant_name
   - merchant_category
   - merchant_city
   - merchant_province

### Requerimientos

1. Diseñe un modelo dimensional (esquema estrella o copo de nieve) para analizar:
   - Patrones de gasto por categoría de comercio.
   - Comportamiento transaccional por segmento de cliente.
   - Uso de tarjetas con respecto a límites de crédito.
   - Análisis geográfico de transacciones.

2. Especifique para cada tabla:
   - Dimensiones o hechos.
   - Columnas y tipos de datos.
   - Claves primarias y foráneas.
   - Granularidad de los datos.
   - Estrategia de actualización (SCD Tipo 1, 2, etc.)

3. Diagrama el modelo propuesto (puede usar cualquier herramienta o incluso un dibujo a mano).

4. Explique cómo implementaría este modelo usando la arquitectura Medallion, detallando:
   - Estructura de tablas en cada nivel. (Bronze, Silver, Gold)
   - Transformaciones en cada nivel.
   - Frecuencia de actualización.
   - Estrategias de particionamiento.

---

## Parte 3: Implementación de pipeline de datos con PySpark (50%)

### Caso de uso: Análisis de riesgo crediticio para campañas personalizadas

El equipo de marketing necesita generar una campaña dirigida a clientes con potencial para aumentar su línea de crédito. Para esto, necesitan un análisis detallado del comportamiento de pago y uso de tarjetas.

### Datos de muestra
A continuación, se proporcionan datos de muestra con los que trabajará. En producción, estos datos serían mucho más voluminosos.

#### Archivo 1: `credit_transactions.csv` (Transacciones)
```
transaction_id,customer_id,transaction_date,amount,type,status,merchant_category
T1001,C5678,2023-01-05,120.50,purchase,completed,retail
T1002,C8925,2023-01-05,1500.00,cash_advance,completed,atm
T1003,C5678,2023-01-06,45.20,purchase,completed,restaurant
T1004,C1234,2023-01-06,800.00,purchase,completed,electronics
T1005,C7890,2023-01-07,60.75,purchase,declined,online
T1006,C1234,2023-01-08,20.00,payment,completed,service
T1007,C5678,2023-01-10,1000.00,payment,completed,service
T1008,C9876,2023-01-12,500.00,purchase,completed,travel
T1009,C7890,2023-01-15,125.30,purchase,completed,restaurant
T1010,C1234,2023-01-16,2000.00,payment,completed,service
T1011,C5678,2023-01-18,300.00,purchase,completed,retail
T1012,C8925,2023-01-20,75.00,purchase,completed,retail
T1013,C9876,2023-01-22,1200.00,purchase,completed,electronics
T1014,C7890,2023-01-25,450.00,purchase,completed,travel
T1015,C1234,2023-01-27,55.90,purchase,completed,restaurant
T1016,C8925,2023-01-28,2500.00,payment,completed,service
T1017,C7890,2023-01-30,80.25,purchase,completed,retail
T1018,C9876,2023-01-31,100.00,cash_advance,completed,atm
T1019,C5678,2023-02-01,75.60,purchase,completed,restaurant
T1020,C1234,2023-02-03,300.00,purchase,completed,retail
```

#### Archivo 2: `customer_accounts.csv` (Información de cuentas)
```
customer_id,credit_limit,account_open_date,credit_score,income,last_payment_date,min_payment,total_due
C1234,5000.00,2020-05-12,720,60000,2023-01-15,150.00,3000.00
C5678,8000.00,2018-10-23,680,75000,2023-01-20,200.00,4000.00
C7890,3000.00,2021-02-18,650,45000,2023-01-25,100.00,2000.00
C8925,10000.00,2019-07-30,750,90000,2023-01-28,250.00,5000.00
C9876,6000.00,2020-11-05,700,70000,2023-01-30,180.00,3600.00
```

#### Archivo 3: `customer_demographics.csv` (Demografía de clientes)
```
customer_id,age,gender,city,province,employment_status,time_at_address,time_at_employer
C1234,35,M,Guayaquil,Guayas,employed,48,60
C5678,42,F,Quito,Pichincha,employed,36,84
C7890,28,M,Cuenca,Azuay,employed,24,36
C8925,55,F,Quito,Pichincha,employed,120,180
C9876,31,M,Guayaquil,Guayas,self_employed,60,72
```

#### Archivo 4: `merchant_categories.csv` (Categorías de comercios)
```
category,description,risk_level
retail,Retail shops and department stores,medium
restaurant,Restaurants and dining,low
electronics,Electronics and appliance stores,high
travel,Travel agencies and services,medium
online,Online shopping platforms,high
atm,ATM withdrawals and cash advances,very_high
service,Service payments,low
```

### Tareas

#### 1. Ingesta y transformación de datos
Escriba código PySpark para:

a) Ingestar los datos desde los archivos CSV proporcionados (simule que están en una ubicación DBFS).
b) Aplicar la arquitectura Medallion:
   - **Bronze**: Almacenar los datos en su forma original (con validaciones mínimas)
   - **Silver**: Limpiar, normalizar y validar los datos
   - **Gold**: Crear tablas para análisis específicos

#### 2. Limpieza y validación de datos
Implemente las siguientes validaciones y transformaciones en la capa Silver:

a) Conversión de tipos de datos apropiados
b) Manejo de valores nulos y duplicados
c) Validaciones de integridad (ej. clientes que aparecen en transacciones pero no en la tabla de clientes)
d) Normalización de categorías y valores


### Entregable final
Entregue los siguientes componentes:

1. Notebooks de Databricks (o scripts equivalentes) para cada paso del pipeline.
2. Diagrama de flujo del proceso completo.
3. Documentación detallada de las decisiones tomadas.
4. Sugerencias para mejoras o extensiones futuras del pipeline.

---

## Criterios de evaluación

### Conocimientos técnicos (50%)
- Correcto uso de PySpark y Databricks.
- Implementación adecuada de la arquitectura Medallion.
- Calidad del modelado dimensional.
- Optimización de rendimiento.

### Calidad del código (30%)
- Legibilidad y organización.
- Manejo de errores.
- Modularidad y reusabilidad.
- Documentación.

### Visión de negocio (20%)
- Comprensión del dominio bancario. (Plus)
- Alineación de la solución técnica con necesidades de negocio.
- Pertinencia de las métricas y segmentos creados.
- Valor analítico de la solución final.

---

## Recursos adicionales

Durante la evaluación puede consultar:
- Documentación de Apache Spark: https://spark.apache.org/docs/latest/
- Documentación de Databricks: https://docs.databricks.com/
- Documentación de Delta Lake: https://delta.io/
- Documentación de PySpark: https://spark.apache.org/docs/latest/api/python/

¡Buena suerte!
Cualquier duda contactarse al: 0984993898