# Talk Title: DuckDB: 20 problems, 20 minutes - one solution

Talk Summary: Need to find a good home within a school catchment area? Want to find the best TV episode of the Simpson’s? In a hurry to load hundreds of random parquet files with inconsistent schemas?  Give me 20 minutes of your time and I’ll show you how DuckDB - the fast, free & versatile analytical database - can be the solution for many a data challenge.


# Part A - DuckDB as a SQL OLAP database
As a baseline - let's get familiar with SQL commands at the DuckDB CLI

Let's launch DuckDB command line intervafe (CLI)
```bash
duckdb
```

## Loading CSV files
We'll be using the flexible [read_csv](https://duckdb.org/docs/data/csv/overview.html#csv-loading) function

```sql
-- A single file; note the type detection
SELECT *   
FROM read_csv('food_collection/pizza_1.csv');

-- Multiple files, adding a psudo-column filename
SELECT * 
FROM read_csv('food_collection/pizza*.csv', auto_detect=true, filename=true);

-- Mixed CSV schemas - this will fail as the schema changes
SELECT *  
FROM read_csv('food_collection/*.csv', auto_detect=true);

-- Mixed CSV schemas - this will adapt as the schema changes
SELECT *
FROM read_csv('food_collection/*.csv', 
union_by_name=true, 
auto_detect=true,
filename=true);

-- Files don't need to be local
SELECT *
FROM read_csv(
'https://www2.census.gov/programs-surveys/popest/datasets/2020-2022/cities/totals/sub-est2022.csv'
);
```

## Loading JSON files
We'll be using the flexible [read_json](https://duckdb.org/docs/data/json/overview#json-loading) function

```sql
-- Here's my FitBit data
SELECT *
FROM read_json('exercise-*.json');

-- Let's hint the types, adjust the timezone
SELECT 
  startTime + INTERVAL 11 hours as activityTime
, activityName
, activityLevel
, averageHeartRate
, calories
, duration / 60000 as duration_minutes
, steps
, distance
, distanceUnit
, tcxLink
, source
FROM read_json('exercise-*.json'
, columns={startTime: 'TIMESTAMP', activityName: 'VARCHAR',  activityLevel: 'JSON', averageHeartRate: 'INTEGER', calories: 'INTEGER', duration: 'INTEGER', steps: 'INTEGER', tcxLink: 'VARCHAR', distance: 'DOUBLE', distanceUnit: 'VARCHAR', source: 'JSON'}
, format='array'
, timestampformat='%m/%d/%y %H:%M:%S');
```


 
## Loading Parquet files
We'll be using the flexible [read_parquet](https://duckdb.org/docs/data/parquet/overview#parquet-files) function

```sql
CREATE OR REPLACE TABLE taxi_data
AS
SELECT *
FROM read_parquet('data/yellow_tripdata*.parquet');

SUMMARIZE taxi_data;

WITH stats AS (
  SELECT  percentile_disc([0.25, 0.50, 0.75]) WITHIN GROUP 
      (ORDER BY "trip_distance") AS percentiles
  FROM taxi_data
)
SELECT
  percentiles[1] AS q1,
  percentiles[2] AS median,
  percentiles[3] AS q3
FROM stats;
```




## DuckDB Extension: SQLite

```sql
SELECT * 
FROM duckdb_extensions();

INSTALL sqlite_scanner;
LOAD sqlite_scanner;

--DETACH imessage_chat_sqlite;

ATTACH './chat.db' as imessage_chat_sqlite (TYPE sqlite,  READ_ONLY TRUE);

SHOW ALL TABLES;

SELECT text
FROM imessage_chat_sqlite.message;
```

## DuckDB Extension: httpfs

```sql
-- Reading remote files with the httpfs extension
INSTALL httpfs;
LOAD httpfs;

CREATE OR REPLACE SECRET mysecret (
    TYPE S3,
    REGION 'us-east-1',
    ENDPOINT 's3.amazonaws.com'
);

SELECT *
FROM read_parquet('s3://duckdb-s3-bucket-public/countries.parquet')
WHERE name SIMILAR TO '.*Republic.*';
```

## DuckDB Extension: full-text search indexes

```sql

-- Allow unused blocks to be offloaded to disk if required
PRAGMA temp_directory='./tmp.tmp';

CREATE OR REPLACE SEQUENCE book_details_seq;

CREATE OR REPLACE TABLE book_details AS
SELECT nextval('book_details_seq') AS book_details_id,
    'Title' as book_title, 
    description as book_description
FROM read_csv('./books_data.csv',  auto_detect=TRUE);

SUMMARIZE book_details;

-- full-text search indexes
INSTALL fts; 
LOAD fts;

PRAGMA create_fts_index(
'book_details',
'book_details_id',
'book_title',
'book_description',
overwrite=1
);

WITH book_cte AS (
  SELECT *, 
  fts_main_book_details.match_bm25(
    book_details_id, 
    'travel italy wine'
    ) AS match_score
  FROM book_details
)
SELECT book_title, book_description, match_score
FROM book_cte
WHERE match_score IS NOT NULL
ORDER BY match_score DESC
LIMIT 10;
```

## DuckDB Extension: Geo-spatial 

```sql
INSTALL spatial; 
LOAD spatial; 

-- The latitude of Sydney Opera House, Australia is -33.856159, and the longitude is 151.215256.
-- The latitude of Sydney Harbour Bridge, Australia is -33.852222, and the longitude is 151.210556

SELECT st_point(-33.856159, 151.215256) AS Sydney_Opera_House;

-- Using Geocentric Datum of Australia (GDA94) / EPSG:3112
SELECT
  st_point(-33.856159, 151.215256) AS Sydney_Opera_House, 
  st_point(-33.852222, 151.210556) AS Sydney_Harbour_Bridge,
  st_distance(
      st_transform(Sydney_Opera_House, 'EPSG:4326', 'EPSG:3112'), 
      st_transform(Sydney_Harbour_Bridge, 'EPSG:4326', 'EPSG:3112')
  ) AS Aerial_Distance_M;

-- Read an Excel binary file
SELECT *
FROM st_read('houses.xlsx', layer='houses');

-- reading Excel XLSX files
CREATE OR REPLACE TABLE houses AS
SELECT *
FROM st_read('houses.xlsx', layer='houses');

SELECT geom 
FROM st_read('./school_boundary_region.geojson');

SELECT house
FROM houses
WHERE st_within(
    st_point(longitude, latitude), 
    (
        SELECT geom
        FROM st_read('./school_boundary_region.geojson')
    )
);
```




## Query JSON from an API

```sql

INSTALL httpfs;
LOAD httpfs;

select *
from read_json('https://api.tvmaze.com/singlesearch/shows?q=The%20Simpsons', 
auto_detect=true, format='newline_delimited');

-- Now we know The Simpsons is coded as show number 83

select season, 
number, 
name, 
json_extract_string(rating, '$.average') as avg_rating, 
summary
from read_json('https://api.tvmaze.com/shows/83/episodebynumber?season=34&number=2', auto_detect=true, format='newline_delimited');

```




