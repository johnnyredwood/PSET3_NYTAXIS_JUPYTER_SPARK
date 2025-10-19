Universidad San Francisco de Quito
Data Mining
Proyecto 03
John Ochoa Abad 345743

#Github Link:
https://github.com/johnnyredwood/PSET3_NYTAXIS_JUPYTER_SPARK/tree/main

#Descripción del proyecto:
Implementé una infraestructura analítica completa utilizando Docker Compose que integra Jupyter+Spark con Snowflake para procesar el dataset 
NYC TLC Trips 2015-2025. El proyecto replica el proceso de ingesta de datos Parquet de taxis Yellow y Green hacia un esquema raw en Snowflake, 
construyendo posteriormente una tabla analítica unificada (One Big Table) en el esquema analytics.

Desarrollé cinco notebooks parametrizados que gestionan todo el flujo de datos: desde la ingesta inicial y enriquecimiento con Taxi Zones, 
hasta la construcción de la OBT y el análisis de 20 preguntas de negocio. Implementé controles de idempotencia, auditoría de cargas y 
validaciones de calidad, asegurando que la reingesta de meses no duplique registros. La infraestructura se configura completamente mediante 
variables de ambiente, manteniendo las credenciales seguras y el proceso reproducible.


#Checklist de aceptación
[x] Docker Compose levanta Spark y Jupyter Notebook.
[x] Todas las credenciales/parámetros provienen de variables de ambiente (.env).
[x] Cobertura 2015–2025 (Yellow/Green) cargada en raw con matriz y conteos por lote.
[x] analytics.obt_trips creada con columnas mínimas, derivadas y metadatos.
[x] Idempotencia verificada reingestando al menos un mes.
[x] Validaciones básicas documentadas (nulos, rangos, coherencia).
[x] 20 preguntas respondidas (texto) usando la OBT.
[x] README claro: pasos, variables, esquema, decisiones, troubleshooting.


#Variables de ambiente: listado y propósito; guía para .env.

Para la ejecución de los notebooks se definieron las siguientes variables de ambiente de Snowflake:

*SNOWFLAKE_URL
*SNOWFLAKE_ACCOUNT
*SNOWFLAKE_USER
*SNOWFLAKE_PASSWORD
*SNOWFLAKE_WAREHOUSE
*SNOWFLAKE_DATABASE
*SNOWFLAKE_SCHEMA_RAW
*SNOWFLAKE_SCHEMA_ANALYTICS
*SNOWFLAKE_ROLE

Adicionalmente, se configuraron parámetros de origen de datos y entorno:

*SOURCE_PATH: URL base del dataset NYC TLC
*YEARS: Rango de años a procesar (2015-2025)
*MONTHS: Meses a incluir en el procesamiento
*SERVICES: Tipos de servicio (yellow, green)

Con estas variables siguiendo el ejemplo del .env.example incluido en el proyecto se puede reproducir el mismo con credenciales propias
de esta manera se gestiona correctamente los datos sensibles.

#Arquitectura (diagrama/tabla): Spark/Jupyter → Snowflake (raw → analytics.obt_trips).

┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   FUENTE        │    │   PROCESAMIENTO  │    │   DESTINO        │
│                 │    │   SPARK/JUPYTER  │    │   SNOWFLAKE      │
│                 │    │                  │    │                  │
│  NYC TLC Data   │──> │   Docker         │    │                  │
│  Parquet Files  │    │   Container      │    │                  │
│  (2015-2025)    │    │                  │    │ ┌──────────────┐ │
│                 │    │ ┌──────────────┐ │    │ │ Esquema Raw  │ │
│  Yellow/Green   │    │ │ Notebook 01  │ │    │ │              │ │ 
│  Taxi Trips     │    │ │ Ingesta      │─┼────────> Taxis Raw  │ │
└─────────────────┘    │ └──────────────┘ │    │ │              │ │
                       │ │ Notebook 02  │ │    │ │  Joins con   │ │
                       │ │ Unificar     │─┼────────> Taxi Zones │ │
 					   │ │ Datos        │ │    │ └───────│──────┘ │
                       │ └──────────────┘ │    │ ┌───────V──────┐ │
                       │ │ Notebook 03  │ │    │ │    Esquema   │ │
                       │ │ Construcción │      │ │  Analytics   │ │
                       │ │ OBT          │─┼────────> OBT con    │ │              
                       │ └──────────────┘ │    │ │  derivadas   │ │           
                       │ │ Notebook 04  │ │    │ │              │ │
                       │ │ Validación   │─┼────────> Datos      │ │
                       │ └──────────────┘ │    │ │   depurados  │ │
                       │ │ Notebook 05  │ │    │ │   y lógicos  │ │
                       │ │ Análisis     │<┼────────  en OBT     │ │
					   │ │   de negocio │ │    │ └──────────────┘ │
                       │ └──────────────┘ │    │                  │
                       └──────────────────┘    └──────────────────┘

#Matriz de cobertura 2015–2025 por servicio/mes (ok/falta/fallido).

Se leyeron los datos directamente desde https://d37ci6vzurychx.cloudfront.net y tras el consumo de los mismos en Jupyter se registro 
la siguiente matriz de datos por servicio y mes:

| AÑO  | MES   | YELLOW_COUNT | GREEN_COUNT | ESTADO_LOAD |
| ---- | ----- | ------------ | ----------- | ----------- |
| 2015 | 1     | 12741035     | 1508493     | OK          |
| 2015 | 2     | 12442394     | 1574830     | OK          |
| 2015 | 3     | 13342951     | 1722574     | OK          |
| 2015 | 4     | 13063758     | 1664394     | OK          |
| 2015 | 5     | 13157677     | 1786848     | OK          |
| 2015 | 6     | 12324936     | 1638868     | OK          |
| 2015 | 7     | 11559666     | 1541671     | OK          |
| 2015 | 8     | 11123123     | 1532343     | OK          |
| 2015 | 9     | 11218122     | 1494927     | OK          |
| 2015 | 10    | 12307333     | 1630536     | OK          |
| 2015 | 11    | 11305240     | 1529984     | OK          |
| 2015 | 12    | 11452996     | 1608297     | OK          |
| 2016 | 1     | 10905067     | 1445292     | OK          |
| 2016 | 2     | 11375412     | 1510722     | OK          |
| 2016 | 3     | 12203824     | 1576393     | OK          |
| 2016 | 4     | 11927996     | 1543926     | OK          |
| 2016 | 5     | 11832049     | 1536979     | OK          |
| 2016 | 6     | 11131645     | 1404727     | OK          |
| 2016 | 7     | 10294080     | 1332510     | OK          |
| 2016 | 8     | 9942263      | 1247675     | OK          |
| 2016 | 9     | 10116018     | 1162373     | OK          |
| 2016 | 10    | 10854626     | 1252572     | OK          |
| 2016 | 11    | 10102128     | 1148214     | OK          |
| 2016 | 12    | 10446697     | 1224158     | OK          |
| 2017 | 1     | 9710820      | 1069565     | OK          |
| 2017 | 2     | 9169775      | 1022313     | OK          |
| 2017 | 3     | 10295441     | 1157827     | OK          |
| 2017 | 4     | 10047135     | 1080844     | OK          |
| 2017 | 5     | 10102127     | 1059463     | OK          |
| 2017 | 6     | 9656993      | 976467      | OK          |
| 2017 | 7     | 8588486      | 914783      | OK          |
| 2017 | 8     | 8422153      | 867407      | OK          |
| 2017 | 9     | 8945421      | 882464      | OK          |
| 2017 | 10    | 9768672      | 925737      | OK          |
| 2017 | 11    | 9284803      | 874173      | OK          |
| 2017 | 12    | 9508501      | 906016      | OK          |
| 2018 | 1     | 8760687      | 792744      | OK          |
| 2018 | 2     | 8492819      | 769197      | OK          |
| 2018 | 3     | 9431289      | 836246      | OK          |
| 2018 | 4     | 9306216      | 799383      | OK          |
| 2018 | 5     | 9224788      | 796552      | OK          |
| 2018 | 6     | 8714667      | 738546      | OK          |
| 2018 | 7     | 7851143      | 684374      | OK          |
| 2018 | 8     | 7855040      | 675815      | OK          |
| 2018 | 9     | 8049094      | 682032      | OK          |
| 2018 | 10    | 8834520      | 731888      | OK          |
| 2018 | 11    | 8155449      | 673287      | OK          |
| 2018 | 12    | 8195675      | 719654      | OK          |
| 2019 | 1     | 7696617      | 672105      | OK          |
| 2019 | 2     | 7049370      | 615594      | OK          |
| 2019 | 3     | 7866620      | 643063      | OK          |
| 2019 | 4     | 7475949      | 567852      | OK          |
| 2019 | 5     | 7598445      | 545452      | OK          |
| 2019 | 6     | 6971560      | 506238      | OK          |
| 2019 | 7     | 6310419      | 470743      | OK          |
| 2019 | 8     | 6073357      | 449695      | OK          |
| 2019 | 9     | 6567788      | 449063      | OK          |
| 2019 | 10    | 7213891      | 476386      | OK          |
| 2019 | 11    | 6878111      | 449500      | OK          |
| 2019 | 12    | 6896317      | 455294      | OK          |
| 2020 | 1     | 6405008      | 447770      | OK          |
| 2020 | 2     | 6299367      | 398632      | OK          |
| 2020 | 3     | 3007687      | 223496      | OK          |
| 2020 | 4     | 238073       | 35644       | OK          |
| 2020 | 5     | 348415       | 57361       | OK          |
| 2020 | 6     | 549797       | 63110       | OK          |
| 2020 | 7     | 800412       | 72258       | OK          |
| 2020 | 8     | 1007286      | 81063       | OK          |
| 2020 | 9     | 1341017      | 87987       | OK          |
| 2020 | 10    | 1681132      | 95120       | OK          |
| 2020 | 11    | 1509000      | 88605       | OK          |
| 2020 | 12    | 1461898      | 83130       | OK          |
| 2021 | 1     | 1369769      | 76518       | OK          |
| 2021 | 2     | 1371709      | 64572       | OK          |
| 2021 | 3     | 1925152      | 83827       | OK          |
| 2021 | 4     | 2171187      | 86941       | OK          |
| 2021 | 5     | 2507109      | 88180       | OK          |
| 2021 | 6     | 2834264      | 86737       | OK          |
| 2021 | 7     | 2821746      | 83691       | OK          |
| 2021 | 8     | 2788757      | 83499       | OK          |
| 2021 | 9     | 2963793      | 95709       | OK          |
| 2021 | 10    | 3463504      | 110891      | OK          |
| 2021 | 11    | 3472949      | 108229      | OK          |
| 2021 | 12    | 3214369      | 99961       | OK          |
| 2022 | 1     | 2463931      | 62495       | OK          |
| 2022 | 2     | 2979431      | 69399       | OK          |
| 2022 | 3     | 3627882      | 78537       | OK          |
| 2022 | 4     | 3599920      | 76136       | OK          |
| 2022 | 5     | 3588295      | 76891       | OK          |
| 2022 | 6     | 3558124      | 73718       | OK          |
| 2022 | 7     | 3174394      | 64192       | OK          |
| 2022 | 8     | 3152677      | 65929       | OK          |
| 2022 | 9     | 3183767      | 69031       | OK          |
| 2022 | 10    | 3675411      | 69322       | OK          |
| 2022 | 11    | 3252717      | 62313       | OK          |
| 2022 | 12    | 3399549      | 72439       | OK          |
| 2023 | 1     | 3066766      | 68211       | OK          |
| 2023 | 2     | 2913955      | 64809       | OK          |
| 2023 | 3     | 3403766      | 72044       | OK          |
| 2023 | 4     | 3288250      | 65392       | OK          |
| 2023 | 5     | 3513649      | 69174       | OK          |
| 2023 | 6     | 3307234      | 65550       | OK          |
| 2023 | 7     | 2907108      | 61343       | OK          |
| 2023 | 8     | 2824209      | 60649       | OK          |
| 2023 | 9     | 2846722      | 65471       | OK          |
| 2023 | 10    | 3522285      | 66177       | OK          |
| 2023 | 11    | 3339715      | 64025       | OK          |
| 2023 | 12    | 3376567      | 64215       | OK          |
| 2024 | 1     | 2964624      | 56551       | OK          |
| 2024 | 2     | 3007526      | 53577       | OK          |
| 2024 | 3     | 3582628      | 57457       | OK          |
| 2024 | 4     | 3514289      | 56471       | OK          |
| 2024 | 5     | 3723833      | 61003       | OK          |
| 2024 | 6     | 3539193      | 54748       | OK          |
| 2024 | 7     | 3076903      | 51837       | OK          |
| 2024 | 8     | 2979183      | 51771       | OK          |
| 2024 | 9     | 3633030      | 54440       | OK          |
| 2024 | 10    | 3833771      | 56147       | OK          |
| 2024 | 11    | 3646369      | 52222       | OK          |
| 2024 | 12    | 3668371      | 53994       | OK          |
| 2025 | 1     | 3475226      | 48326       | OK          |
| 2025 | 2     | 3577543      | 46621       | OK          |
| 2025 | 3     | 4145257      | 51539       | OK          |
| 2025 | 4     | 3970553      | 52132       | OK          |
| 2025 | 5     | 4591845      | 55399       | OK          |
| 2025 | 6     | 4322960      | 49390       | OK          |
| 2025 | 7     | 3898963      | 48205       | OK          |
| 2025 | 8     | 3574091      | 46306       | OK          |


#Pasos para Docker Compose y ejecución de notebooks (orden y parámetros).

Prerrequisitos
*Docker instalado
*Docker Compose instalado
*Archivo .env configurado con las credenciales de Snowflake

1. Descargar de repositorio y Configuración del Ambiente

- Descargar el repositorio a su entorno local con
git clone https://github.com/johnnyredwood/PSET3_NYTAXIS_JUPYTER_SPARK.git

Crear archivo de variables de ambiente:

- Copiar el template y configurar con valores reales
cp .env.example .env

- Editar el archivo .env con tus credenciales
nano .env

2. Verificar estructura de directorios:

proyecto/
├── docker-compose.yml
├── .env
├── notebooks/
│   ├── 01_ingesta_parquet_raw.ipynb
│   ├── 02_enriquecimiento_y_unificacion.ipynb
│   ├── 03_construccion_obt.ipynb
│   ├── 04_validaciones_y_exploracion.ipynb
│   └── 05_data_analysis.ipynb
└── work/

3. Inicialización de la Infraestructura
Levantar los servicios con Docker Compose:

- Ejecutar en el directorio del proyecto el siguiente comando para levantar el contenedor con variables de entorno de .env
docker-compose --env-file .env up -d

- Verificar que el contenedor esté corriendo
docker-compose ps

4. Acceder a Jupyter Notebook:
Acceder al Jupyter Notebook del contenedor con el puerto y token indicados en su .env

URL: http://localhost:puerto
Token: [valor de JUPYTER_TOKEN en .env]

5. Ejecución Secuencial de Notebooks
Orden de ejecución obligatorio:

Notebook 01 - Ingesta de Datos RAW
Parámetros esperados:
- Años: 2015-2025 (configurado en .env)
- Meses: 1-12 (configurado en .env)  
- Servicios: yellow, green (configurado en .env)
Genera:
-Tabla RAW de datos de taxi por servicio en Snowflake

Notebook 02 - Enriquecimiento y Unificación
-Depende de: Notebook 01 completado
-Procesa: Datos RAW
-Incluye: Integración con Taxi Zones, normalización de catálogos
Genera:
-Tabla enriquecida y unificada de datos de taxis con zonas de taxis en Snowflake

Notebook 03 - Construcción OBT
- Depende de: Notebook 02 completado  
- Procesa: Datos unificados hacia analytics
- Incluye: Cálculo de derivadas, aplicación de idempotencia
Genera:
-Primera versión de la tabla OBT de Taxis

Notebook 04 - Validaciones y Exploración
-Depende de: Notebook 03 completado
-Verifica: Calidad de datos, rangos lógicos, consistencia
Genera: 
-Tabla OBT lista para consultas de negocio

Notebook 05 - Análisis de Datos
-Depende de: Notebook 04 completado
-Consulta: tabla obt validada y hosteada en Snowflake
-Responde: 20 preguntas de negocio desde tabla obt

#Diseño de raw y OBT (columnas, derivadas, metadatos, supuestos).

*Esquema RAW
El esquema raw funciona como capa de aterrizaje donde se preservan los datos en su 
formato original con metadatos de ingesta. Se implementaron tablas particionadas por servicio 
y período para optimizar el manejo de los volúmenes de datos.

Estructura de tablas RAW:

NY_TAXI_RAW_YELLOW - Viajes de taxi amarillo RAW
NY_TAXI_RAW_GREEN - Viajes de taxi verde RAW
NY_TAXI_RAW_TAXI_ZONES - Zonas de Taxis de New York RAW

Columnas base preservadas del origen:

Datos temporales: pickup/dropoff datetime
Ubicaciones: PULocationID, DOLocationID
Métricas de viaje: trip_distance, passenger_count
Tarifas: fare_amount, tip_amount, tolls_amount, total_amount
Identificadores: VendorID, RatecodeID, payment_type

Metadatos de ingesta agregados:

run_id - Identificador único de la ejecución
source_year / source_month - Período de origen
ingested_at_utc - Timestamp de ingesta
service_type - Tipo de servicio (yellow/green)

*Esquema ANALYTICS - OBT
La OBT consolida todos los datos de viajes de taxis de New York en una tabla que junta toda 
la información necesaria validada y depurada a manera de ejecutar consultas de negocio
sobre la misma

Columnas de la OBT:

Temporales:
pickup_datetime, dropoff_datetime - Timestamps originales
pickup_date, pickup_hour - Componentes temporales
dropoff_date, dropoff_hour - Componentes temporales
trip_duration_min - Duración calculada en minutos

Ubicaciones:
pu_location_id, do_location_id - IDs originales
pu_zone, pu_borough - Nombres desnormalizados
do_zone, do_borough - Nombres desnormalizados

Servicio y Códigos:
vendor_id, vendor_name - Desnormalizado
rate_code_id, rate_code_desc - Desnormalizado
payment_type, payment_type_desc - Desnormalizado

Métricas y Tarifas:
passenger_count, trip_distance
fare_amount, extra, mta_tax, tip_amount
tolls_amount, improvement_surcharge
congestion_surcharge, airport_fee, total_amount

Derivadas Calculadas:
trip_duration_min - Duración del viaje en minutos
avg_speed_mph - Velocidad promedio (solo viajes válidos)
tip_pct - Porcentaje de propina sobre tarifa base

Metadatos:
run_id - Trazabilidad de la ejecución
ingested_at_utc - Fecha de procesamiento
source_service - Servicio de origen
source_year, source_month - Período origen

Supuestos de Diseño
Clave Natural: Se define basada en pickup_datetime, PULocationID, DOLocationID y VendorID para garantizar identificación única de viajes en merges

Estrategia de Idempotencia: Implementación de UPSERT basado en clave natural, permitiendo reingesta sin duplicados.

Manejo de Datos: Se han filtrado nulos en campos obligatorios y se ha definido validaciones lógicas para datos númericos de forma que los mismos
cumplan con rangos lógicos

Cálculo de Derivadas:

tip_pct: tip_amount / fare_amount (solo cuando fare_amount > 0)

avg_speed_mph: trip_distance / (trip_duration_min/60) (solo viajes con duración y distancia válidas)

trip_duration_min: (dropoff_datetime - pickup_datetime) en minutos

#Calidad/auditoría: qué se valida y dónde se ve.

*Validación de Conectividad con Snowflake desde Spark:
Inicio sesión de Spark y posteriormente genero una conexión con Snowflake con mis credenciales y ejecuto una query simple de SELECT current_version()
esto lo valido en todos los notebooks antes de proceder con el consumo, procesamiento y/o lectura de datos

*Validación de Ingesta de datos:
En todos los notebooks he implementado logs en forma de prints y manejo de excepciones para ir monitoreando el proceso de consumo de todos los datos.
A su vez una vez los mismos se iban consumiendo ingresaba en Snowflake a verificar que las tablas aumenten en cantidad de filas y monitoreaba los datos
recien ingresados con queries simples desde Snowflake

*Validación del contenedor de docker:
Al tener spark-notebook: Jupyter+Spark desde un contenedor de docker verificaba que el mismo estuviera funcionando correctamente con el comando docker ps,
con el Docker Desktop verificando que el contenedor este arriba e ingresando a localhost con el puerto definido y verificando que pudiera ingresar
sin problema a Jupyter

*Validación de datos consumidos en OBT:
En el Notebook 4 durante el proceso de generación de la tabla OBT se aplican múltiples validaciones de calidad de datos para garantizar 
la coherencia y consistencia de los registros. En primer lugar, se eliminan todas las filas que contienen valores nulos en campos 
críticos como DO_LOCATION_ID, PU_LOCATION_ID, PASSENGER_COUNT, PAYMENT_TYPE, RATE_CODE_ID, PICKUP_DATETIME, DROPOFF_DATETIME, TRIP_DISTANCE 
y VENDOR_ID. Posteriormente, se filtran los registros que presentan valores incoherentes, restringiendo el número de pasajeros entre 1 y 9, 
los montos monetarios (EXTRA, FARE_AMOUNT, METROPOLITAN_TAX, TIP_AMOUNT, TOLLS_AMOUNT, TOTAL_AMOUNT) a valores no negativos, las distancias 
de viaje (TRIP_DISTANCE) a valores mayores que cero, y la duración del viaje (TRIP_DURATION_MIN) entre 1 y 180 minutos. Además, se asegura 
que la velocidad promedio (AVG_SPEED_MPH) se mantenga entre 0 y 100 mph y que el porcentaje de propina (TIP_PCT) sea no negativo. 
Finalmente, se validan las ubicaciones de recogida y destino (PU_LOCATION_ID, DO_LOCATION_ID) para que estén dentro del rango 1 a 265, 
y las fechas (MONTH, YEAR) se restringen a meses válidos (1–12) y años comprendidos entre 2015 y 2025.