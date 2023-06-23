<h1>Case Study #5: Data Mart</h1>
💼 Table of Contents
Business Task
Entity Relationship Diagram
Question and Solution
Please note that all the information regarding the case study has been sourced from the following link: .

<h1>🧹 Data Exploration and Cleansing<h1>
1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

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
![Uploading image.png…]()

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








