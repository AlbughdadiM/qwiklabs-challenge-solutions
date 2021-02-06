# Solution of Create ML Models with BigQuery ML: Challenge Lab 

## Task 1

Create a BQ dataset in the project 

## Task 2: create a first model 

```sql 
CREATE OR REPLACE MODEL
  bike.tripduration_model1 OPTIONS (model_type='linear_reg',
    input_label_cols=['duration_minutes']) AS (
  SELECT
    start_station_id,location,day,hour,duration_minutes
  FROM (
    SELECT
      x.start_station_id,
      IFNULL(x.duration_minutes,
        0) AS duration_minutes,
      y.location,
      EXTRACT(YEAR
      FROM
        x.start_time) AS year,
      EXTRACT(DAYOFWEEK
      FROM
        x.start_time) AS day,
      EXTRACT(HOUR
      FROM
        x.start_time) AS hour
    FROM
      `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS x
    JOIN
      `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS y
    ON
      x.start_station_id = y.station_id)
  WHERE
    year=2018)
```

## Task 3: create a second model

```sql 
 CREATE OR REPLACE MODEL
  bike.tripduration_model2 OPTIONS (model_type='linear_reg',
    input_label_cols=['duration_minutes']) AS (
  SELECT
    start_station_id,hour,subscriber_type,duration_minutes 
  FROM (
    SELECT
      start_station_id,
      IFNULL(duration_minutes,
        0) AS duration_minutes,
      subscriber_type,
      EXTRACT(HOUR
      FROM
        start_time) AS hour,
      EXTRACT(YEAR
      FROM
        start_time) AS year
    FROM
      `bigquery-public-data.austin_bikeshare.bikeshare_trips`)
  WHERE
    year=2018)   
```

 ## Task 4: evaluate both models 

```sql 
SELECT
  *
FROM
  ML.EVALUATE(MODEL bike.tripduration_model1)
```
```sql
SELECT
  *
FROM
  ML.EVALUATE(MODEL  bike.tripduration2_model2)
```
 
 
 ## On 2019 data

```sql 
  SELECT
  SQRT(mean_squared_error) AS rmse,
  mean_absolute_error
FROM
  ML.EVALUATE(MODEL bike.tripduration_model1,
    (
    SELECT
      start_station_id,location,day,hour,duration_minutes
    FROM (
      SELECT
        x.start_station_id,
        IFNULL(x.duration_minutes,
          0) AS duration_minutes,
        y.location,
        EXTRACT(YEAR
        FROM
          x.start_time) AS year,
        EXTRACT(DAYOFWEEK
        FROM
          x.start_time) AS day,
        EXTRACT(HOUR
        FROM
          x.start_time) AS hour
      FROM
        `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS x
      JOIN
        `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS y
      ON
        x.start_station_id = y.station_id)
    WHERE
      year=2019))  
```

```sql 
  SELECT
  SQRT(mean_squared_error) AS rmse,
  mean_absolute_error
FROM
  ML.EVALUATE(MODEL bike.tripduration_model2,
    (
    SELECT
      start_station_id,hour,subscriber_type 
    FROM (
      SELECT
        start_station_id,
        subscriber_type,
        EXTRACT(YEAR
        FROM
          start_time) AS year,
        EXTRACT(HOUR
        FROM
          start_time) AS hour
      FROM
        `bigquery-public-data.austin_bikeshare.bikeshare_trips`)
    WHERE
      year=2019))    
   ```

 ## Task 5: Get the name of the busiest station in 2019 where subscriber type is Single Trip

```sql 
SELECT 
    start_station_name, avg(duration_minutes) as avg_duration, count(*) as total_trips
FROM
    `bigquery-public-data.austin_bikeshare.bikeshare_trips`
WHERE
    EXTRACT(year FROM start_time) = 2019
GROUP BY start_station_name
ORDER BY total_trips DESC
 ```  

## Then get the average prediction of trip duration using the second model 

```sql 
SELECT
avg(predicted_duration_minutes)
FROM
Ml.PREDICT(MODEL bike.tripduration_model2, 
(
SELECT
      start_station_id,hour,subscriber_type,duration_minutes 
    FROM (
      SELECT
        start_station_id,
        subscriber_type,
        IFNULL(duration_minutes,
          0) AS duration_minutes,
        EXTRACT(YEAR
        FROM
          start_time) AS year,
        EXTRACT(HOUR
        FROM
          start_time) AS hour
      FROM
        `bigquery-public-data.austin_bikeshare.bikeshare_trips`
        WHERE 
        start_station_name='21st & Speedway @PCL')
    WHERE
      year=2019 
      AND 
      subscriber_type='Single Trip'))
 ```     