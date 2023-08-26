# Health Analytics Mini Case Study

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the `health.user_logs` dataset.

The Health Co analytics team have shared with us their SQL script - they unfortunately ran into a few bugs that they couldn’t fix!

We’ve been asked to quickly debug their SQL script and use the resulting query outputs to quickly answer a few questions that the GM has requested for a board meeting about their active users.

# Business Questions

Before we start digging into the SQL script - let’s cover the business questions that we need to help the GM answer!

1. How many unique users exist in the logs dataset?
```sql
-- 1. How many unique users exist in the logs dataset?
SELECT
  COUNT( DISTINCT id ) AS unique_users
FROM `health.user_logs`;
```
**unique_users: 554**

For questions 2-8 we created a temporary table. For BigQuery, please make sure `Save query results in a temporary table` is clicked in `Query Setting`.

```sql
DROP TABLE IF EXISTS `younglapalma.health.user_measure_count`;
CREATE TABLE `younglapalma.health.user_measure_count` AS (
  SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM `health.user_logs`
  GROUP BY 1); 
```

2. How many total measurements do we have per user on average?
```sql
-- 2. How many total measurements do we have per user on average?
SELECT
  ROUND(AVG(measure_count), 2)
FROM `younglapalma.health.user_measure_count`;
```
**answer: 79**

3. What about the median number of measurements per user?
```sql
-- 3. What about the median number of measurements per user?
SELECT
  APPROX_QUANTILES(measure_count, 100)[OFFSET(50)] AS median_value
FROM `younglapalma.health.user_measure_count`;
```
**answer: 2**

4. How many users have 3 or more measurements?
```sql
-- 4. How many users have 3 or more measurements?
SELECT
  COUNT(id)
FROM `younglapalma.health.user_measure_count`
WHERE measure_count >= 3;
```
**answer: 209**

5. How many users have 1,000 or more measurements?
```sql
-- 5. How many users have 1,000 or more measurements?
SELECT
  COUNT(id)
FROM `younglapalma.health.user_measure_count`
WHERE measure_count >= 1000;
```
**answer: 5**

Looking at the logs data - what is the number and percentage of the active user base who:

6. Have logged blood glucose measurements?
```sql
-- 6. what is the number of the active user base who have logged blood glucose measurements?
SELECT
  COUNT (DISTINCT id) as number_active_users
FROM `younglapalma.health.user_logs`
WHERE measure = 'blood_glucose';
```
**answer: 325**

7. Have at least 2 types of measurements?
```sql
-- 7. Have at least 2 types of measurements?
SELECT
  COUNT( DISTINCT id )
FROM `younglapalma.health.user_measure_count`
WHERE unique_measures >= 2;
```
**answer: 204**

8. Have all 3 measures - blood glucose, weight and blood pressure?

```sql
-- 8. Have all 3 measures - blood glucose, weight and blood pressure?
SELECT
  COUNT(*)
FROM `younglapalma.health.user_measure_count`
WHERE unique_measures = 3;
```

For users that have blood pressure measurements:

9. What is the median systolic/diastolic blood pressure values?
```sql
-- 9.  What is the median systolic/diastolic blood pressure values?
SELECT
  APPROX_QUANTILES(systolic, 100)[OFFSET(50)] AS median_systolic,
  APPROX_QUANTILES(diastolic, 100)[OFFSET(50)] AS median_diastolic

FROM `younglapalma.health.user_logs`
WHERE measure = 'blood_pressure';
```
**answer: median_systolic: 126,
median_diastolic: 79**
	





