<h1>Case Study #5: Data Mart</h1>
💼 Table of Contents
Business Task
Entity Relationship Diagram
Question and Solution
Please note that all the information regarding the case study has been sourced from the following link: .

🧹<h1>A. Data Exploration and Cleansing<h1>
######  1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

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
3.What do you think we should do with these null values in the fresh_segments.interest_metrics
```

![3](https://github.com/hazalsezgin/case/assets/77546910/54467b6f-efb8-439e-9316-55db4cfb1f25)







