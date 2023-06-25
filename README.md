<h1>Case Study #8: Fresh Segments </h1>

### ![8](https://github.com/hazalsezgin/case/assets/77546910/9d565fa8-d9e2-4f22-a6d6-bbbe6d2840f6)
Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

If you want to learn more about this case, click [here](https://8weeksqlchallenge.com/case-study-8)

<h1>🧹 Data Exploration and Cleansing<h1>
   
### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```ALTER TABLE fresh_segments.interest_metrics
   ALTER COLUMN month_year TYPE DATE USING to_date('01-' || month_year, 'DD-MM-YYYY');
```

2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```![2](https://github.com/hazalsezgin/case/assets/77546910/44fde222-5973-40de-94a2-eed34d9677c1)

SELECT 
  month_year, COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```
![2](https://github.com/hazalsezgin/case/assets/77546910/44fde222-5973-40de-94a2-eed34d9677c1)

3.What do you think we should do with these null values in the fresh_segments.interest_metrics
```

```
![3](https://github.com/hazalsezgin/case/assets/77546910/54467b6f-efb8-439e-9316-55db4cfb1f25)

4.How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
```
SELECT 
  COUNT(DISTINCT map.id) AS map_id_count,
  COUNT(DISTINCT metrics.interest_id::varchar) AS metrics_id_count,
  SUM(CASE WHEN map.id is NULL THEN 1 END) AS not_in_metric,
  SUM(CASE WHEN metrics.interest_id is NULL THEN 1 END) AS not_in_map
FROM fresh_segments.interest_map map
FULL OUTER JOIN fresh_segments.interest_metrics metrics
  ON metrics.interest_id::integer = map.id;
```
![image](https://github.com/hazalsezgin/case/assets/77546910/c7070aac-3404-43c4-9360-67199020b3d3)

![4](https://github.com/hazalsezgin/case/assets/77546910/b93c167d-2c6b-4936-98d1-e65bf3f712df)

5.Summarise the id values in the fresh_segments.interest_map by its total record count in this table
```
```

6.What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
```
SELECT *
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id::integer
WHERE metrics.interest_id = '21246'
  AND metrics._month IS NOT NULL;
```
![6](https://github.com/hazalsezgin/case/assets/77546910/d2c7ee4d-3a6b-4f07-940d-f405771742b7)


7.Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
```
SELECT COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = CAST(metrics.interest_id AS integer)
WHERE TO_DATE(metrics.month_year || '-01', 'MM-YYYY') < map.created_at::DATE;
```
![7](https://github.com/hazalsezgin/case/assets/77546910/a26a7c8f-699c-496c-8e9a-484450a06b84)


<h1>:fire:Interest Analysis<h1>

### 1.Which interests have been present in all month_year dates in our dataset?
```
SELECT
    month_year,
    COUNT(*) AS interest_count
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year;
```
![int 1](https://github.com/hazalsezgin/case/assets/77546910/2d41d78e-916f-4639-a426-709d6c04580e)

2.Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
```
WITH month_yearPer_interest AS(
SELECT 
    interest_id,
    COUNT(month_year) AS month_year_counts
FROM fresh_segments.interest_metrics
GROUP BY interest_id
)
SELECT 
    month_year_counts,
    COUNT(interest_id) AS interest_count,
    ROUND(100*SUM(COUNT(interest_id)) OVER(ORDER BY month_year_counts DESC)/ SUM(COUNT(interest_id)) OVER(), 2) AS cumulative_percent
FROM month_yearPer_interest
GROUP BY month_year_counts
ORDER BY month_year_counts DESC;
```
![int2](https://github.com/hazalsezgin/case/assets/77546910/1594ac25-aa0f-4d50-94b4-4f611ec59815)

3.If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
```
WITH unremoved_records AS(
SELECT 
    interest_id
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
HAVING COUNT(DISTINCT month_year) >=6
)
SELECT
    COUNT(*) AS removed_records
FROM fresh_segments.interest_metrics
WHERE NOT EXISTS (
        SELECT 1
        FROM unremoved_records
        WHERE interest_metrics.interest_id = unremoved_records.interest_id);
```

![int4](https://github.com/hazalsezgin/case/assets/77546910/406e8990-8b26-47ac-89bc-b18c8de45c43)

4.Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
```


```

5.How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

```

```
<h1>🎯Segment Analysis<h1>
   
### 1.Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
```

```
2.Which 5 interests had the lowest average ranking value?
```
SELECT
    interest_map.interest_name,
    ROUND(AVG(ranking), 1) AS average_ranking,
    COUNT(*) AS record_count
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
    ON CAST(interest_metrics.interest_id AS INTEGER) = interest_map.id
GROUP BY interest_map.interest_name
ORDER BY average_ranking
LIMIT 5;

```
![seg 2](https://github.com/hazalsezgin/case/assets/77546910/d23fdca1-4c17-41a4-a25f-00ce2c2186d1)

3.Which 5 interests had the largest standard deviation in their percentile_ranking value?
```
SELECT
    interest_metrics.interest_id,
    interest_map.interest_name,
    ROUND(CAST(STDDEV(percentile_ranking) AS NUMERIC), 1) AS stddev_pc,
    MAX(percentile_ranking) AS max_pc,
    MIN(percentile_ranking) AS min_pc,
    COUNT(*) AS record_counts
FROM fresh_segments.interest_metrics
INNER JOIN fresh_segments.interest_map
    ON CAST(interest_metrics.interest_id AS INTEGER) = interest_map.id
WHERE month_year IS NOT NULL
GROUP BY 
    interest_metrics.interest_id,
    interest_map.interest_name
HAVING STDDEV(percentile_ranking) IS NOT NULL
ORDER BY stddev_pc DESC
LIMIT 5;
```
![seg3](https://github.com/hazalsezgin/case/assets/77546910/be8a9249-a174-4d90-bb9c-3c3dbc5e68db)

4.For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
```
WITH max_stddev_interests AS (
  SELECT
    CAST(interest_metrics.interest_id AS INTEGER) AS interest_id,
    interest_map.interest_name,
    ROUND(CAST(STDDEV(percentile_ranking) AS NUMERIC), 1) AS stddev_pc,
    MAX(percentile_ranking) AS max_pc
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map ON interest_metrics.interest_id = interest_map.id
  WHERE month_year IS NOT NULL
  GROUP BY interest_metrics.interest_id, interest_map.interest_name
  HAVING STDDEV(percentile_ranking) IS NOT NULL
  ORDER BY stddev_pc DESC
  LIMIT 5
)
SELECT
  t2.interest_name,
  t1.month_year,
  t1.ranking,
  t1.percentile_ranking,
  t1.composition,
  t2.stddev_pc,
  RANK() OVER (ORDER BY stddev_pc DESC) AS max_stddev_ranking
FROM fresh_segments
```
<h1>🎯Index Analysis<h1>
   
### 1.What is the top 10 interests by the average composition for each month?
```
WITH avg_compositions AS (
  SELECT 
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND(CAST(interest_metrics.composition / interest_metrics.index_value AS NUMERIC), 2) AS index_composition,
    RANK() OVER (PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map ON interest_metrics.interest_id::integer = interest_map.id
)
SELECT *
FROM avg_compositions
WHERE index_rank <= 10;

```
![index1](https://github.com/hazalsezgin/case/assets/77546910/bd8314d0-a93f-4779-814e-9467d79be6d4)

2.For all of these top 10 interests - which interest appears the most often?
```
WITH avg_compositions AS (
  SELECT 
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND(CAST(interest_metrics.composition / interest_metrics.index_value AS NUMERIC), 2) AS index_composition,
    RANK() OVER (PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map ON interest_metrics.interest_id::integer = interest_map.id
)
SELECT 
    interest_name,
    COUNT(*) AS interest_frequency
FROM avg_compositions
WHERE index_rank <= 10
GROUP BY interest_name
ORDER BY interest_frequency DESC;
```
![index 2](https://github.com/hazalsezgin/case/assets/77546910/fa4db9bd-e039-47bc-ade5-a90582d05e2e)

3.What is the average of the average composition for the top 10 interests for each month?
```
WITH avg_compositions AS (
  SELECT 
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND(CAST(interest_metrics.composition / interest_metrics.index_value AS NUMERIC), 2) AS index_composition,
    RANK() OVER (PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map ON interest_metrics.interest_id::integer = interest_map.id::integer
)
SELECT 
    month_year,
    ROUND(AVG(index_composition), 2) AS avg_index_composition
FROM avg_compositions
WHERE index_rank <= 10
GROUP BY month_year
ORDER BY month_year;
```
![index 3](https://github.com/hazalsezgin/case/assets/77546910/31e8e780-419f-43c9-987a-d5718398329b)

4.What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
```
WITH compositions AS (
  SELECT 
    interest_metrics.month_year,
    interest_map.interest_name,
    ROUND(CAST(interest_metrics.composition / interest_metrics.index_value AS NUMERIC), 2) AS index_composition,
    MAX(ROUND(CAST(interest_metrics.composition / interest_metrics.index_value AS NUMERIC), 2)) OVER (PARTITION BY interest_metrics.month_year) AS max_index_composition,
    RANK() OVER (PARTITION BY interest_metrics.month_year ORDER BY interest_metrics.composition / interest_metrics.index_value DESC) AS index_rank
  FROM fresh_segments.interest_metrics
  INNER JOIN fresh_segments.interest_map ON interest_metrics.interest_id::integer = interest_map.id::integer
),
max_composition_data AS (
  SELECT
    month_year,
    interest_name,
    max_index_composition,
    ROUND(AVG(max_index_composition) OVER (ORDER BY month_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS mvg_avg_3month,
    LAG(interest_name || ': ' || max_index_composition, 1) OVER (ORDER BY month_year) AS "1month_ago",
    LAG(interest_name || ': ' || max_index_composition, 2) OVER (ORDER BY month_year) AS "2months_ago"
  FROM compositions
  WHERE index_rank::integer = 1
)
SELECT *
FROM max_composition_data
WHERE "2months_ago" IS NOT NULL;

```
![index 4](https://github.com/hazalsezgin/case/assets/77546910/a601678e-9803-4421-89d2-05439ca511f7)


