## Module 3 Homework

<b><u>Important Note:</b></u> <p> For this homework we will be using the Yellow Taxi Trip Records for **January 2024 - June 2024 NOT the entire year of data** 
Parquet Files from the New York
City Taxi Data found here: </br> https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page </br>

## Question 1:
Question 1: What is count of records for the 2024 Yellow Taxi Data?
- [ ] 65,623
- [ ] 840,402
- [X] 20,332,093
- [ ] 85,431,289

```sql
SELECT COUNT(*) FROM `taxi_project-123456.taxi_data.external_yellow_taxi_2024` LIMIT 10
f0_
20332093
```

## Question 2:
Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.</br> 
What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

- [ ] 18.82 MB for the External Table and 47.60 MB for the Materialized Table
- [x] 0 MB for the External Table and 155.12 MB for the Materialized Table
- [ ] 2.14 GB for the External Table and 0MB for the Materialized Table
- [ ] 0 MB for the External Table and 0MB for the Materialized Table

```sql
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_locations
FROM `taxi_project-123456.taxi_data.external_yellow_taxi_2024`; -- Table externe
Octets traités
0 octets (résultats mis en cache)
```

```sql
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_locations
FROM `taxi_project-123456.taxi_data.internal_yellow_taxi_2024`; -- Table interne
Octets traités
155,12 Mo
```

## Question 3:
Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
- [X] BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires 
reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.
- [ ] BigQuery duplicates data across multiple storage partitions, so selecting two columns instead of one requires scanning the table twice, 
doubling the estimated bytes processed.
- [ ] BigQuery automatically caches the first queried column, so adding a second column increases processing time but does not affect the estimated bytes scanned.
- [ ] When selecting multiple columns, BigQuery performs an implicit join operation between them, increasing the estimated bytes processed

```sql
SELECT
  PULocationID AS distinct_pu_locations
FROM `taxi_project-123456.taxi_data.internal_yellow_taxi_2024`;
Octets traités
155,12 Mo
```

```sql
SELECT
  PULocationID AS distinct_pu_locations,
  DOLocationID AS distinct_do_locations
FROM `taxi_project-123456.taxi_data.internal_yellow_taxi_2024`;
Octets traités
310,24 Mo
```

## Question 4:
How many records have a fare_amount of 0?
- [ ]  128,210
- [ ] 546,578
- [ ]  20,188,016
- [X] 8,333

```sql
SELECT COUNT(*)
FROM `taxi_project-123456.taxi_data.internal_yellow_taxi_2024`
WHERE fare_amount = 0
f0_
8333
```

## Question 5:
What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
- [X] Partition by tpep_dropoff_datetime and Cluster on VendorID
- [ ] Cluster on by tpep_dropoff_datetime and Cluster on VendorID
- [ ] Cluster on tpep_dropoff_datetime Partition by VendorID
- [ ] Partition by tpep_dropoff_datetime and Partition by VendorID

```sql
CREATE OR REPLACE TABLE `taxi_project-123456.taxi_data.optimized_table`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `taxi_project-123456.taxi_data.external_yellow_taxi_2024`;
```

## Question 6:
Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
2024-03-01 and 2024-03-15 (inclusive)</br>

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values? </br>

Choose the answer which most closely matches.</br> 

- [ ] 12.47 MB for non-partitioned table and 326.42 MB for the partitioned table
- [X] 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table
- [ ] 5.87 MB for non-partitioned table and 0 MB for the partitioned table
- [ ] 310.31 MB for non-partitioned table and 285.64 MB for the partitioned table

```sql
SELECT DISTINCT VendorID
FROM `taxi_project-123456.taxi_data.internal_yellow_taxi_2024`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';

Cette requête traitera 310,24 Mo lors de son exécution.
```

```sql
SELECT DISTINCT VendorID
FROM `taxi_project-123456.taxi_data.optimized_table`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';

Cette requête traitera 26,84 Mo lors de son exécution.
```
## Question 7: 
Where is the data stored in the External Table you created?

- [ ] Big Query
- [ ]  Container Registry
- [X] GCP Bucket
- [ ] Big Table

## Question 8:
It is best practice in Big Query to always cluster your data:
- [ ] True
- [X] False
