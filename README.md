# data-warehousing-homework-3
Homework for Data Warehousing

## Homework 3

## Cloud Shell Commands


```bash
# Clone the repository
git clone https://github.com/DataTalksClub/data-engineering-zoomcamp.git

# Navigate to the relevant directory
cd data-engineering-zoomcamp/cohorts/2026/03-data-warehouse

# List files in the directory
ls

# Open the file for editing
nano load_yellow_taxi_data.py


# Run the Python script
python load_yellow_taxi_data.py

```

## SQL Queries

```sql
-- Create an external table for yellow taxi trip data
CREATE OR REPLACE EXTERNAL TABLE `data-warehousing-486717.homework_3.external_yellow_tripdata`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://dw_homework_3/yellow_tripdata_2024-*.parquet']
);

-- Create a materialized table from the external table
CREATE OR REPLACE TABLE `data-warehousing-486717.homework_3.yellow_tripdata`
AS
SELECT *
FROM `data-warehousing-486717.homework_3.external_yellow_tripdata`;

-- Count total trips in 2024
SELECT COUNT(*) AS trip_count_2024
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- result: 20,332,093

-----------------------------------------------------------------------------------------------------------

-- Count distinct pickup locations in external table
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_location_count
FROM `data-warehousing-486717.homework_3.external_yellow_tripdata`;

-- Count distinct pickup locations in materialized table
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_location_count
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- result: 0 MB for the External Table and 155.12 MB for the Materialized Table

-------------------------------------------------------------------------------------------------------------

-- Query pickup locations
SELECT PULocationID
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- Query pickup and dropoff locations
SELECT PULocationID, DOLocationID
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- result: BigQuery is a columnar database; scanning two columns requires reading more data than one column,
-- leading to a higher estimated number of bytes processed.

----------------------------------------------------------------------------------------------------------------

-- Count trips with zero fare
SELECT COUNT(*) AS zero_fare_count
FROM `data-warehousing-486717.homework_3.yellow_tripdata`
WHERE fare_amount = 0;

-- result: 8,333

-------------------------------------------------------------------------------------------------------------------

-- Create an optimized table partitioned by dropoff date and clustered by VendorID
CREATE OR REPLACE TABLE `data-warehousing-486717.homework_3.yellow_tripdata_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT *
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- result: Partition by tpep_dropoff_datetime and cluster on VendorID

--------------------------------------------------------------------------------------------------------------------

-- Query distinct VendorID for first half of March 2024 from non-partitioned table
SELECT DISTINCT VendorID
FROM `data-warehousing-486717.homework_3.yellow_tripdata`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15 23:59:59';

-- Query distinct VendorID from partitioned table
SELECT DISTINCT VendorID
FROM `data-warehousing-486717.homework_3.yellow_tripdata_optimized`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15 23:59:59';

-- result: 310.24 MB for non-partitioned table and 26.84 MB for partitioned table

--------------------------------------------------------------------------------------------------------------------

-- result: GCP Bucket

--------------------------------------------------------------------------------------------------------------------

-- result: False

--------------------------------------------------------------------------------------------------------------------

-- Count rows in the original materialized table
SELECT count(*)
FROM `data-warehousing-486717.homework_3.yellow_tripdata`;

-- result: 0 B
-- Explanation: When a materialized table is created from an external table, BigQuery still tracks the source as an external table in its metadata.
-- BigQuery’s query size estimator looks at the original source to estimate bytes scanned.
-- External tables don’t have size metadata available in BigQuery, so it estimates 0 B.

```


