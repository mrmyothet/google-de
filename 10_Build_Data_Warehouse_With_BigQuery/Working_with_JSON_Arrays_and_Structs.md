# Working with JSON, Arrays, and Structs in BigQuery

### Practice working with arrays in SQL

```sql

SELECT
['raspberry', 'blackberry', 'strawberry', 'cherry'] AS fruit_array

```

```sql

SELECT
['raspberry', 'blackberry', 'strawberry', 'cherry', 1234567] AS fruit_array

```

```sql

SELECT person, fruit_array, total_cost
FROM `data-to-insights.advanced.fruit_store`;

```

### Loading semi-structured JSON into BigQuery

`cloud-training/data-insights-course/labs/optimizing-for-performance/shopping_cart.json`

### Create your own arrays with ARRAY_AGG()

```sql

SELECT
  fullVisitorId,
  date,
  v2ProductName,
  pageTitle
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
ORDER BY date

```

```sql

SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(v2ProductName) AS products_viewed,
  ARRAY_AGG(pageTitle) AS pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date

```

Use the ARRAY_LENGTH() function to count the number of pages and products that were viewed:

```sql

SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(v2ProductName)) AS num_products_viewed,
  ARRAY_AGG(pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(pageTitle)) AS num_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date

```

Deduplicate the pages and products so you can see how many unique products were viewed by adding DISTINCT to ARRAY_AGG():

```sql

SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(DISTINCT v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT v2ProductName)) AS distinct_products_viewed,
  ARRAY_AGG(DISTINCT pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT pageTitle)) AS distinct_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date

```

### Query tables containing arrays

```sql

SELECT
  *
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398

```

```sql

SELECT
  visitId,
  hits.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398

```

**Error:** Cannot access field page on a value with type ARRAY<STRUCT<hitNumber INT64, time INT64, hour INT64, ...>> at [3:8]

**Answer:** Use the UNNEST() function on your array field:

```sql

SELECT DISTINCT
  visitId,
  h.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`,
UNNEST(hits) AS h
WHERE visitId = 1501570398
LIMIT 10

```

### Introduction to STRUCTs

`bigquery-public-data.google.analytics_sample.ga_sessions_(366)` table

The main advantage of having 32 STRUCTs in a single table is it allows you to run queries like this one without having to do any JOINs:

```sql

SELECT
  visitId,
  totals.*,
  device.*
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398
LIMIT 10

```

### Practice with STRUCTs and arrays

```sql

SELECT STRUCT("Rudisha" as name, 23.4 as split) as runner

```

```sql

SELECT STRUCT("Rudisha" as name, [23.4, 26.3, 26.4, 26.1] as splits) AS runner

```

`cloud-training/data-insights-course/labs/optimizing-for-performance/race_results.json`

### Practice querying nested and repeated fields

```sql
-- see all of our racers for the 800 Meter race:

SELECT * FROM racing.race_results

```

```sql
-- list the name of each runner and the type of race

SELECT race, participants.name
FROM racing.race_results

```

```sql

SELECT race, participants.name
FROM racing.race_results
CROSS JOIN
race_results.participants -- full STRUCT name

```

```sql

SELECT race, participants.name
FROM racing.race_results AS r, r.participants

```

### Lab question: STRUCT()

1. COUNT how many racers were there in total.

```sql

SELECT COUNT(p.name) AS racer_count
FROM racing.race_results AS r, UNNEST(r.participants) AS p

```

### Unpacking arrays with UNNEST()

List the total race time for racers whose names begin with R.  
Order the results with the fastest total time first.

```sql

SELECT
  p.name,
  SUM(split_times) as total_race_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_times
WHERE p.name LIKE 'R%'
GROUP BY p.name
ORDER BY total_race_time ASC;

```

### Filter within array values

You happened to see that the fastest lap time recorded for the 800 M race was 23.2 seconds, but you did not see which runner ran that particular lap.  
Create a query that returns that result.

```sql

SELECT
  p.name,
  split_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_time
WHERE split_time = 23.2;

```
