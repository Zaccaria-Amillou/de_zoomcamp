## Module 4 Homework

For this homework, i used the following datasets:
* [Green Taxi dataset (2019 and 2020)](https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/green)
* [Yellow Taxi dataset (2019 and 2020)](https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/yellow)
* [For Hire Vehicle dataset (2019)](https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/fhv)



### Question 1: Understanding dbt model resolution

Provided you've got the following sources.yaml
```yaml
version: 2

sources:
  - name: raw_nyc_tripdata
    database: "{{ env_var('DBT_BIGQUERY_PROJECT', 'dtc_zoomcamp_2025') }}"
    schema:   "{{ env_var('DBT_BIGQUERY_SOURCE_DATASET', 'raw_nyc_tripdata') }}"
    tables:
      - name: ext_green_taxi
      - name: ext_yellow_taxi
```

with the following env variables setup where `dbt` runs:
```shell
export DBT_BIGQUERY_PROJECT=myproject
export DBT_BIGQUERY_DATASET=my_nyc_tripdata
```

What does this .sql model compile to?
```sql
select * 
from {{ source('raw_nyc_tripdata', 'ext_green_taxi' ) }}
```

- [ ] `select * from dtc_zoomcamp_2025.raw_nyc_tripdata.ext_green_taxi`
- [ ] `select * from dtc_zoomcamp_2025.my_nyc_tripdata.ext_green_taxi`
- [ ] `select * from myproject.raw_nyc_tripdata.ext_green_taxi`
- [x] `select * from myproject.my_nyc_tripdata.ext_green_taxi`
- [ ] `select * from dtc_zoomcamp_2025.raw_nyc_tripdata.green_taxi`


### Question 2: dbt Variables & Dynamic Models

Say you have to modify the following dbt_model (`fct_recent_taxi_trips.sql`) to enable Analytics Engineers to dynamically control the date range. 

- In development, you want to process only **the last 7 days of trips**
- In production, you need to process **the last 30 days** for analytics

```sql
select *
from {{ ref('fact_taxi_trips') }}
where pickup_datetime >= CURRENT_DATE - INTERVAL '30' DAY
```

What would you change to accomplish that in a such way that command line arguments takes precedence over ENV_VARs, which takes precedence over DEFAULT value?

-[] Add `ORDER BY pickup_datetime DESC` and `LIMIT {{ var("days_back", 30) }}`
-[] Update the WHERE clause to `pickup_datetime >= CURRENT_DATE - INTERVAL '{{ var("days_back", 30) }}' DAY`
-[] Update the WHERE clause to `pickup_datetime >= CURRENT_DATE - INTERVAL '{{ env_var("DAYS_BACK", "30") }}' DAY`
-[x] Update the WHERE clause to `pickup_datetime >= CURRENT_DATE - INTERVAL '{{ var("days_back", env_var("DAYS_BACK", "30")) }}' DAY`
-[] Update the WHERE clause to `pickup_datetime >= CURRENT_DATE - INTERVAL '{{ env_var("DAYS_BACK", var("days_back", "30")) }}' DAY`


### Question 3: dbt Data Lineage and Execution

Considering the data lineage below **and** that taxi_zone_lookup is the **only** materialization build (from a .csv seed file):

![image](./homework_q2.png)

Select the option that does **NOT** apply for materializing `fct_taxi_monthly_zone_revenue`:

- [ ] `dbt run`
- [ ] `dbt run --select +models/core/dim_taxi_trips.sql+ --target prod`
- [ ] `dbt run --select +models/core/fct_taxi_monthly_zone_revenue.sql`
- [ ] `dbt run --select +models/core/`
- [x] `dbt run --select models/staging/+`


### Question 4: dbt Macros and Jinja

```sql
{% macro resolve_schema_for(model_type) -%}

    {%- set target_env_var = 'DBT_BIGQUERY_TARGET_DATASET'  -%}
    {%- set stging_env_var = 'DBT_BIGQUERY_STAGING_DATASET' -%}

    {%- if model_type == 'core' -%} {{- env_var(target_env_var) -}}
    {%- else -%}                    {{- env_var(stging_env_var, env_var(target_env_var)) -}}
    {%- endif -%}

{%- endmacro %}
```

And use on your staging, dim_ and fact_ models as:
```sql
{{ config(
    schema=resolve_schema_for('core'), 
) }}
```

select all statements that are true to the models using it:
- [x]Setting a value for  `DBT_BIGQUERY_TARGET_DATASET` env var is mandatory, or it'll fail to compile
- [ ] Setting a value for `DBT_BIGQUERY_STAGING_DATASET` env var is mandatory, or it'll fail to compile
- [x] When using `core`, it materializes in the dataset defined in `DBT_BIGQUERY_TARGET_DATASET`
- [x] When using `stg`, it materializes in the dataset defined in `DBT_BIGQUERY_STAGING_DATASET`, or defaults to `DBT_BIGQUERY_TARGET_DATASET`
- [x] When using `staging`, it materializes in the dataset defined in `DBT_BIGQUERY_STAGING_DATASET`, or defaults to `DBT_BIGQUERY_TARGET_DATASET`



### Question 5: Taxi Quarterly Revenue Growth

Considering the YoY Growth in 2020, which were the yearly quarters with the best (or less worse) and worst results for green, and yellow

- [ ] green: {best: 2020/Q2, worst: 2020/Q1}, yellow: {best: 2020/Q2, worst: 2020/Q1}
- [ ] green: {best: 2020/Q2, worst: 2020/Q1}, yellow: {best: 2020/Q3, worst: 2020/Q4}
- [ ] green: {best: 2020/Q1, worst: 2020/Q2}, yellow: {best: 2020/Q2, worst: 2020/Q1}
- [x]green: {best: 2020/Q1, worst: 2020/Q2}, yellow: {best: 2020/Q1, worst: 2020/Q2}
- [ ] green: {best: 2020/Q1, worst: 2020/Q2}, yellow: {best: 2020/Q3, worst: 2020/Q4}
```sql
WITH quarterly_revenue AS (
    -- Étape 1 : Calculer les revenus trimestriels pour chaque année et taxi type
    SELECT
        'yellow' AS service_type,
        EXTRACT(YEAR FROM tpep_pickup_datetime) AS year,
        EXTRACT(QUARTER FROM tpep_pickup_datetime) AS quarter,
        CONCAT(EXTRACT(YEAR FROM tpep_pickup_datetime), '/Q', EXTRACT(QUARTER FROM tpep_pickup_datetime)) AS year_quarter,
        SUM(total_amount) AS revenue
    FROM `noted-lead-448822-q9.zoomcamp_dbt_4.yellow_tripdata`
    WHERE total_amount > 0  -- On exclut les valeurs invalides
    GROUP BY 1, 2, 3, 4

    UNION ALL

    SELECT
        'green' AS service_type,
        EXTRACT(YEAR FROM lpep_pickup_datetime) AS year,
        EXTRACT(QUARTER FROM lpep_pickup_datetime) AS quarter,
        CONCAT(EXTRACT(YEAR FROM lpep_pickup_datetime), '/Q', EXTRACT(QUARTER FROM lpep_pickup_datetime)) AS year_quarter,
        SUM(total_amount) AS revenue
    FROM `noted-lead-448822-q9.zoomcamp_dbt_4.green_tripdata`
    WHERE total_amount > 0
    GROUP BY 1, 2, 3, 4
),

revenue_with_growth AS (
    -- Étape 2 : Calculer la croissance YoY (comparaison avec l’année précédente)
    SELECT 
        q1.service_type,
        q1.year,
        q1.quarter,
        q1.year_quarter,
        q1.revenue AS current_revenue,
        q2.revenue AS previous_revenue,
        ROUND((q1.revenue - q2.revenue) / q2.revenue * 100, 2) AS yoy_growth
    FROM quarterly_revenue q1
    LEFT JOIN quarterly_revenue q2
        ON q1.service_type = q2.service_type
        AND q1.year = q2.year + 1
        AND q1.quarter = q2.quarter
)

-- Étape 3 : Sélection finale des résultats
SELECT *
FROM revenue_with_growth
ORDER BY service_type, year, quarter;
```

### Question 6: P97/P95/P90 Taxi Monthly Fare

wWhat are the values of `p97`, `p95`, `p90` for Green Taxi and Yellow Taxi, in April 2020?

- [ ] green: {p97: 55.0, p95: 45.0, p90: 26.5}, yellow: {p97: 52.0, p95: 37.0, p90: 25.5}
- [x] green: {p97: 55.0, p95: 45.0, p90: 26.5}, yellow: {p97: 31.5, p95: 25.5, p90: 19.0}
- [ ] green: {p97: 40.0, p95: 33.0, p90: 24.5}, yellow: {p97: 52.0, p95: 37.0, p90: 25.5}
- [ ] green: {p97: 40.0, p95: 33.0, p90: 24.5}, yellow: {p97: 31.5, p95: 25.5, p90: 19.0}
- [ ] green: {p97: 55.0, p95: 45.0, p90: 26.5}, yellow: {p97: 52.0, p95: 25.5, p90: 19.0}
```sql
WITH valid_trips AS (
    SELECT
        'yellow' AS service_type,
        EXTRACT(YEAR FROM tpep_pickup_datetime) AS year,
        EXTRACT(MONTH FROM tpep_pickup_datetime) AS month,
        fare_amount
    FROM `noted-lead-448822-q9.zoomcamp_dbt_4.yellow_tripdata`
    WHERE fare_amount > 0
      AND trip_distance > 0
      AND payment_type IN (1, 2)  -- 1 = Credit Card, 2 = Cash

    UNION ALL

    SELECT
        'green' AS service_type,
        EXTRACT(YEAR FROM lpep_pickup_datetime) AS year,
        EXTRACT(MONTH FROM lpep_pickup_datetime) AS month,
        fare_amount
    FROM `noted-lead-448822-q9.zoomcamp_dbt_4.green_tripdata`
    WHERE fare_amount > 0
      AND trip_distance > 0
      AND payment_type IN (1, 2)
),

percentiles AS (
    SELECT
        service_type,
        year,
        month,
        APPROX_QUANTILES(fare_amount, 100)[SAFE_OFFSET(97)] AS p97,
        APPROX_QUANTILES(fare_amount, 100)[SAFE_OFFSET(95)] AS p95,
        APPROX_QUANTILES(fare_amount, 100)[SAFE_OFFSET(90)] AS p90
    FROM valid_trips
    GROUP BY service_type, year, month
)

-- Sélection des valeurs pour avril 2020
SELECT *
FROM percentiles
WHERE year = 2020 AND month = 4;
```

### Question 7: Top #Nth longest P90 travel time Location for FHV

For the Trips that **respectively** started from `Newark Airport`, `SoHo`, and `Yorkville East`, in November 2019, what are **dropoff_zones** with the 2nd longest p90 trip_duration ?

- [x] LaGuardia Airport, Chinatown, Garment District
- [ ] LaGuardia Airport, Park Slope, Clinton East
- [ ] LaGuardia Airport, Saint Albans, Howard Beach
- [ ] LaGuardia Airport, Rosedale, Bath Beach
- [ ] LaGuardia Airport, Yorkville East, Greenpoint
```sql
WITH trip_durations AS (
    SELECT
        EXTRACT(YEAR FROM pickup_datetime) AS year,
        EXTRACT(MONTH FROM pickup_datetime) AS month,
        PUlocationID,
        DOlocationID,
        APPROX_QUANTILES(TIMESTAMP_DIFF(dropoff_datetime, pickup_datetime, SECOND), 100)[91] AS p90_trip_duration
    FROM `noted-lead-448822-q9.zoomcamp_dbt_4.fhv_tripdata`
    WHERE EXTRACT(YEAR FROM pickup_datetime) = 2019 AND EXTRACT(MONTH FROM pickup_datetime) = 11
    GROUP BY year, month, PUlocationID, DOlocationID
),
ranked_trips AS (
    SELECT
        t.*,
        z_pu.zone AS pickup_zone,
        z_do.zone AS dropoff_zone,
        RANK() OVER (PARTITION BY PUlocationID ORDER BY p90_trip_duration DESC) AS rank
    FROM trip_durations t
    JOIN `noted-lead-448822-q9.dbt_yphamvan.dim_zones` z_pu 
        ON t.PUlocationID = z_pu.locationid
    JOIN `noted-lead-448822-q9.dbt_yphamvan.dim_zones` z_do 
        ON t.DOlocationID = z_do.locationid
    WHERE z_pu.zone IN ('Newark Airport', 'SoHo', 'Yorkville East')
)
SELECT pickup_zone, dropoff_zone, p90_trip_duration
FROM ranked_trips
WHERE rank = 2;
```

