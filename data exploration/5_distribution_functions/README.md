# Beyond Summary Statistics

Let’s recap those final summary statistics in our last tutorial, because there were clearly some issues with that weight column!

| Column Name         | Data Value         |
|---------------------|-------------------|
| minimum_value       |               0.00 |
| maximum_value       |        39642120.00 |
| mean_value          |           28786.85 |
| median_value        |              75.98 |
| mode_value          |              68.49 |
| standard_deviation  |         1062759.55 |
| variance_value      | 1129457862383.41   |

A few questions come to mind straightaway (I hope!):

- Does it make sense to have such low minimum values and such a high value?
- Why is the average value 28,786kg but the median is 75.98kg?
- The standard deviation of values is WAY too large at 1,062,759kg

This leads us to the next question - what the heck do you do when the summary statistics is clearly telling you that something is dodgy with your data?

In this following section I am going to introduce you to your new best friend:
**cumulative distribution functions**.

# Cumulative What?

Remember that example about school test taking and what it takes to beat 99% of other kids taking the test?

<img width="469" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/cc1a68c1-7e2e-4265-98ac-ca3518d11de8">

In that same way where we can investigate what percentage of people I beat with my test results - we can inspect something similar for our data points.

In mathematical terms, a cumulative distribution function takes a value and returns us the percentile or in other words: the probability of any value between the minimum value of our dataset `X` and the value `V` as shown below:

$$F(V) = \int^{V}_{min(X)} f(x)dx=Pr[min(X)≤x≤V]$$

The beautiful thing about probabilities is that they always add up to 1!

So what does this mean? Let’s explain a bit further with the help of some algorithms and reverse-engineering!

## Reverse Engineering

Let’s break this down into a few components - but we’ll try a different approach this time.

Just like mechanics sometimes learn how to repair cars by first taking them apart, I am going to cut to the chase without much further explanation - we want something like this:

| percentile | floor_value | ceiling_value | percentile_counts |
|----------- |------------ |-------------- |------------------ |
| 1          | 0           | 29.029888     | 28                |
| 2          | 29.48348    | 32.0689544    | 28                |
| 3          | 32.205032   | 35.380177     | 28                |
| 4          | 35.380177   | 36.74095      | 28                |
| 5          | 36.74095    | 37.194546     | 28                |
| 6          | 37.194546   | 38.101727     | 28                |
| ...        | ...         | ...           | ...               |
| 96         | 130.54207   | 131.570999146 | 27                |
| 97         | 131.670013428 | 132.776     | 27                |
| 98         | 132.776000977 | 133.832000732 | 27                |
| 99         | 133.89095   | 136.531192    | 27                |
| 100        | 136.531192  | 39642120      | 27                |

Let’s relate this to the test results again - the percentile is that probability or percentage of how many other test-takers I beat with my score value.

In our example however - instead of using JUST my test score - I’ve requested both a `floor` and a `ceiling` value for each percentile.

Also - I want the percentile counts to show how many records exist in that percentile, i.e. how many values lie between the floor and the ceiling values for that bucket of records.

## Algorithmic Thinking

Now that we know the endpoint of our analysis - let’s work from the bottom up to design an algorithm to make this output happen!

1. Order all of the values from smallest to largest
In your mind imagine all of those weight data points and line them up from smallest to largest
----

2. Split them into 100 equal buckets - and assign a number from 1 through to 100 for each bucket

We often refer to this as bucketing or to use the actual function name `NTILE`-ing a data input - these bucket values are also your percentiles!

When we split out sorted dataset into 100 buckets - we have effectively generated our new “groups” or buckets of data to continue with our algorithm.

If you think about this carefully - you’ll notice that each bucket should have 1% of the total number of records from the entire dataset!

How many records did we have again for `measure = 'weight'`? Does our 1% of values make sense in this case?

```sql
SELECT
  COUNT(*)
FROM health.user_logs
WHERE measure = 'weight';
```
**count: 2782**

3. For each bucket:
    - calculate the minimum value and the maximum value for the ceiling and floor values
    - count how many records there are

For example say we look at the smallest 1% worth of data sorted by measure value with that percentile value attached:

| **percentile** | **floor_value** | **ceiling_value** | **percentile_counts** |
|----------- |------------ |-------------- |------------------ |
| 1          | 0           | 29.029888     | 28                |

4. Combine all the aggregated bucket results into a final summary table

| **percentile** | **floor_value** | **ceiling_value** | **percentile_counts** |
|--------------|---------------|------------------|--------------------|
| 1            |           0    |       29.029888   |               28    |
| 2            |       29.48348 |       32.0689544  |               28    |
| 3            |       32.205032|       35.380177   |               28    |
| 4            |       35.380177|       36.74095    |               28    |
| 5            |       36.74095 |       37.194546   |               28    |
| 6            |       37.194546|       38.101727   |               28    |
| ...          |       ...      |       ...         |               ...   |
| 96           |      130.54207 |      131.570999146|               27    |
| 97           |      131.670013|      132.776      |               27    |
| 98           |      132.776000|      133.832000732|               27    |
| 99           |      133.89095 |      136.531192   |               27    |
| 100          |      136.531192|      39642120     |               27    |

## SQL Implementation
So how do we do this in SQL?

We can complete all those algorithm steps in a single query because SQL is really awesome like that.

Everything below we’ve seen before except for the new `NTILE` window function and the `OVER` and `ORDER BY` components of analytical functions - this is something which we will cover in much more depth in the next case study section of Serious SQL so for now, don’t worry too much about understanding the details of these things!

So let’s break down our algorithm steps through the SQL components:

### Order and Assign

1. Order all of the weight measurement values from smallest to largest
2. Split them into 100 equal buckets - and assign a number from 1 through to 100 for each bucket

We can actually achieve both of these algorithmic steps in a single bit of SQL functionality with the all-powerful **analytical function**

Firstly the `OVER` and `ORDER BY` clauses in the following query help us re-order the dataset by the measure_value column - it sorts by ascending order by default

Then the `NTILE` window function is used to perform the assignment of numbers 1 through 100 for each row in the records for each `measure_value`.

```sql
SELECT
  measure_value,
  NTILE(100) OVER (
    ORDER BY
      measure_value
  ) AS percentile
FROM `health.user_logs`
WHERE measure = 'weight'
ORDER BY measure_value
```

*Only the first 5 and last 5 rows of the output are shown below*

| **measure_value** | **percentile** |
|------------------|--------------|
|           0       |             1 |
|           0       |             1 |
|           1.814368|             1 |
|           2.26796 |             1 |
|           2.26796 |             1 |
| ...              |           ... |
|         190.4     |           100 |
|         200.487664|           100 |
|      576484       |           100 |
|   39642120       |           100 |
|   39642120       |           100 |

### Bucket Calculations
3. For each bucket:
    - calculate the minimum value and the maximum value for the ceiling and floor values
    - count how many records there are

Since we now have our percentile values and our dataset is split into 100 buckets - we can simply use a `GROUP BY` on the `percentile` column from the previous table to calculate our `MIN` and `MAX` `measure_value` ranges for each bucket and also the `COUNT` of records for the `percentile_counts` field.

We can also use the previous query in a CTE so we can pull all the calculations in a single SQL query:

| **percentile** | **floor_value** | **ceiling_value** | **percentile_counts** |
|--------------|---------------|------------------|---------------------|
| 1             | 0              | 29.029888         | 28                   |
| 2             | 29.48348       | 32.0689544        | 28                   |
| 3             | 32.205032      | 35.380177         | 28                   |
| 4             | 35.380177      | 36.74095          | 28                   |
| 5             | 36.74095       | 37.194546         | 28                   |
| 6             | 37.194546      | 38.101727         | 28                   |
| ...           | ...            | ...               | ...                  |
| 96            | 130.54207      | 131.570999146     | 27                   |
| 97            | 131.670013428  | 132.776           | 27                   |
| 98            | 132.776000977  | 133.832000732     | 27                   |
| 99            | 133.89095      | 136.531192        | 27                   |
| 100           | 136.531192     | 39642120          | 27                   |

## What Do I Do With This?

So you’re probably asking yourself - ok cool…so what do I do with this cumulative distribution table then?

The first place to take a careful look at is the tails of the values - namely `percentile = 1` and `percentile = 100`

What do you notice from these two rows alone?

| **percentile** | **floor_value** | **ceiling_value** | **percentile_counts** |
|--------------|---------------|------------------|---------------------|
| 1             | 0              | 29.029888         | 28                   |
| 100           | 136.531192     | 39642120          | 27                   |

So from first glance you get the following insights right away:

1. 28 values lie between 0 and ~29KG
2. 27 values lie between 136.53KG and 39,642,120KG

Please tell me that you don’t think insight number 2 is normal - unless you’re a GIANT!

Let’s dive a little bit deeper and start thinking critically about what these insights mean.

## Critical Thinking

Ok - firstly we need to consider what the data point is that we’re actually using here - in this case, it is a weight value in KG units.

When we think of those small values in the 1st percentile under 29kg, a few things should come to mind:

1. Maybe there were some incorrectly measured values - leading to some 0kg weight measurements
2. Perhaps some of the low weights under 29kg were actually valid measurements from young children who were part of the customer base

For the 100th percentile we could consider:

1. Does that 136KG floor value make sense?
2. How many error values were there actually in the dataset?

Let’s first inspect that 1st and 100th percentile values carefully and inspect each value to see what happened!

## Deep Dive Into 100th Percentile

Let’s re-use our same query to quickly show a sorted table from largest to smallest for the `measure_value` field.

Take special note of the values of the first few rows - they might surprise you!

We haven’t covered this in-depth yet in this Serious SQL course but this is a perfect simple example to show you the differences between some of these window functions!

I’ll let you try to figure out the logical differences `ROW_NUMBER`, `RANK` and `DENSE_RANK` in the following query!

### Window Functions for Sorting Values

```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  measure_value,
  -- these are examples of window functions below
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) as row_number_order,
  RANK() OVER (ORDER BY measure_value DESC) as rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value DESC) as dense_rank_order
FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;
```

| **measure_value** | **row_number_order** | **rank_order** | **dense_rank_order** |
|------------------|--------------------|---------------|---------------------|
|        39642120   |                   1 |              1 |                    1 |
|        39642120   |                   2 |              1 |                    1 |
|          576484   |                   3 |              3 |                    2 |
|      200.487664   |                   4 |              4 |                    3 |
|          190.4     |                   5 |              5 |                    4 |
|      188.69427    |                   6 |              6 |                    5 |
|      186.8799     |                   7 |              7 |                    6 |
|      185.51913    |                   8 |              8 |                    7 |
|      175.086512   |                   9 |              9 |                    8 |
|      173.725736   |                  10 |             10 |                    9 |
|      170.5506     |                  11 |             11 |                   10 |
|      170.5506     |                  12 |             11 |                   10 |
|      170.5506     |                  13 |             11 |                   10 |
|        164        |                  14 |             14 |                   11 |
|      157.5778608  |                  15 |             15 |                   12 |
|      149.68536    |                  16 |             16 |                   13 |
|      145.14944    |                  17 |             17 |                   14 |
|      144.242256   |                  18 |             18 |                   15 |
|      141.1578304  |                  19 |             19 |                   16 |
|      138.799152   |                  20 |             20 |                   17 |
|      137.438376   |                  21 |             21 |                   18 |
|      136.984784   |                  22 |             22 |                   19 |
|      136.984784   |                  23 |             22 |                   19 |
|      136.762008667|                  24 |             24 |                   20 |
|      136.531192   |                  25 |             25 |                   21 |
|      136.531192   |                  26 |             25 |                   21 |
|      136.531192   |                  27 |             25 |                   21 |

### Optional Exercise

This is actually a really popular SQL interview question:

> What is the difference between `ROW_NUMBER`, `RANK` and `DENSE_RANK` window functions?

The `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` functions are all window functions in SQL that assign a unique rank to each row within a partition of a result set. However, they handle ties (rows with equal values) differently.

`ROW_NUMBER()`:

Assigns a unique number to each row within the result set, starting at 1 for the first row.
If there are ties (identical values in the columns you are ordering by), `ROW_NUMBER()` will assign different numbers to each tied row arbitrarily.
No two rows will have the same row number, even if they have the same values in the ordered columns.

`RANK()`:

Assigns a unique rank to each distinct row within the result set.
If there are ties, `RANK()` will give the same rank to all tied rows. The next rank after a tie will be incremented by the number of tied rows. For example, if there are two rows with rank 3, the next rank will be 5.
This means that ranks might be skipped in the presence of ties.

`DENSE_RANK()`:

Similar to `RANK()`, it assigns a unique rank to each distinct row within the result set and gives the same rank to all tied rows.
Unlike `RANK()`, `DENSE_RANK()` does not skip any rank numbers. For example, if there are two rows with rank 3, the next rank will be 4.
This ensures that there are no gaps between rank numbers, regardless of ties.

## Large Outliers

So it looks like there are at least 3 HUGE values which are outliers in our weight values in that top percentile.

| **measure_value** | **row_number_order** | **rank_order** | **dense_rank_order** |
|------------------|--------------------|---------------|---------------------|
|        39642120   |                   1 |              1 |                    1 |
|        39642120   |                   2 |              1 |                    1 |
|          576484   |                   3 |              3 |                    2 |
|      200.487664   |                   4 |              4 |                    3 |

The last 200kg value MIGHT be a real measurement - so we would have to apply our judgement to determine whether this is actually an outlier or not.

Now that we’ve identified these large outliers - the next question is: what do we do with them?

The answer is luckily simple: you remove them!

We will demonstrate this removal soon but before that - let’s inspect our first percentile with the smallest values.

## Small Outliers

Let’s again show all of those ordering window functions so you can take a look again - this time we will sort from smallest to largest so we will remove all of the `DESC` parts in our query to end up with the following:

```sql 
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  measure_value,
  ROW_NUMBER() OVER (ORDER BY measure_value) as row_number_order,
  RANK() OVER (ORDER BY measure_value) as rank_order,
  DENSE_RANK() OVER (ORDER BY measure_value) as dense_rank_order
FROM percentile_values
WHERE percentile = 1
ORDER BY measure_value;
```

| **measure_value** | **row_number_order** | **rank_order** | **dense_rank_order** |
|------------------|--------------------|---------------|---------------------|
|           0       |                   1 |              1 |                    1 |
|           0       |                   2 |              1 |                    1 |
|      1.814368     |                   3 |              3 |                    2 |
|      2.26796      |                   4 |              4 |                    3 |
|      2.26796      |                   5 |              4 |                    3 |
|           8       |                   6 |              6 |                    4 |
|      10.432616    |                   7 |              7 |                    5 |
|      11.3398      |                   8 |              8 |                    6 |
|      12.700576    |                   9 |              9 |                    7 |
|      15.422128    |                  10 |             10 |                    8 |
|      18.14368     |                  11 |             11 |                    9 |
|      20.865232    |                  12 |             12 |                   10 |
|      23.586784    |                  13 |             13 |                   11 |
|      24.94756     |                  14 |             14 |                   12 |
|      25.854744    |                  15 |             15 |                   13 |
|      25.854744    |                  16 |             15 |                   13 |
|      27.21552     |                  17 |             17 |                   14 |
|      27.21552     |                  18 |             17 |                   14 |
|      27.21552     |                  19 |             17 |                   14 |
|      27.21552     |                  20 |             17 |                   14 |
|      27.669112    |                  21 |             21 |                   15 |
|      28.122704    |                  22 |             22 |                   16 |
|      28.122704    |                  23 |             22 |                   16 |
|      28.576296    |                  24 |             24 |                   17 |
|      28.576296    |                  25 |             24 |                   17 |
|      29.029888    |                  26 |             26 |                   18 |
|      29.029888    |                  27 |             26 |                   18 |
|      29.029888    |                  28 |             26 |                   18 |

To interpret these results - it seems like there are a few 0 weight values as well as a few VERY small values.

Again, this will be a judgement call as to whether we should keep or remove some of these values.

For me - I would just remove the 0 values and keep everything else - it’s not perfect, but sometimes good enough is simply that - good enough!

## Removing Outliers

Now we get to the fun part - removing these outliers! To do this - we can simply use a `WHERE` filter with some SQL conditions.

Let’s create a temporary `clean_weight_logs` table with our outliers removed by applying strict inequality conditions.

To follow this up, let’s create that same overall summary statistics table we created with the original `health.user_logs` prior to our outlier removal to compare the results!

1. Create a temporary table using a `CREATE TEMP TABLE <> AS` statement

```sql
DROP TABLE IF EXISTS clean_weight_logs;
CREATE TEMP TABLE clean_weight_logs AS (
  SELECT *
  FROM health.user_logs
  WHERE measure = 'weight'
    AND measure_value > 0
    AND measure_value < 201
);
```

2. Calculate summary statistics on this new temp table

Let’s re-use that same code we used for the original summary statistics and compare this to the original output.

```sql
WITH stats AS (
  SELECT
    ROUND(MIN(measure_value), 2) AS minimum_value,
    ROUND(MAX(measure_value), 2) AS maximum_value,
    ROUND(AVG(measure_value), 2) AS mean_value,
    ROUND(STDDEV(measure_value), 2) AS standard_dev_value,
    ROUND(VARIANCE(measure_value), 2) AS variance_value,
    APPROX_QUANTILES(measure_value, 100)[OFFSET(50)] AS median_value
  FROM `health.clean_weight_logs`
),

mode AS (
  SELECT value AS mode_value
  FROM (
    SELECT 
      measure_value AS value, 
      COUNT(*) AS count
    FROM `health.clean_weight_logs`
    GROUP BY measure_value
    ORDER BY count DESC
    LIMIT 1
  )
)

SELECT
  minimum_value,
  maximum_value,
  mean_value,
  ROUND(median_value, 2) AS median_value,
  ROUND((SELECT mode_value FROM mode), 2) AS mode_value,
  standard_dev_value,
  variance_value
FROM stats;
```

| **minimum_value** | **maximum_value** | **mean_value** | **median_value** | **mode_value** | **standard_deviation** | **variance_value** |
|------------------|------------------|--------------|-----------------|--------------|----------------------|------------------|
|            1.81   |           200.49  |         80.76  |            75.98  |         68.49  |                 26.91  |            724.29  |



How does this compare to our original summary statistics on the weight values prior to our treatment of the outliers?

| **minimum_value** | **maximum_value** | **mean_value** | **median_value** | **mode_value** | **standard_deviation** | **variance_value** |
|------------------|------------------|--------------|-----------------|--------------|----------------------|------------------|
|               0.00|       39642120.00|       28786.85|            75.98|         68.49|             1062759.55|    1129457862383.41|

3. Show the new cumulative distribution function with treated data

```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM `health.clean_weight_logs`
)
SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;
```
*All 100 rows are shown below!*


| **percentile** | **floor_value** | **ceiling_value** | **percentile_counts** |
|--------------:|---------------:|-----------------:|----------------------:|
|             1 |        1.814368 |          29.48348 |                    28 |
|             2 |       29.48348  |          32.4771872 |                    28 |
|             3 |       32.658623 |          35.380177  |                    28 |
|             4 |       35.380177 |          36.74095   |                    28 |
|             5 |       36.74095  |          37.194546  |                    28 |
|             6 |       37.648136 |          38.101727  |                    28 |
|             7 |       38.101728 |          39.00891   |                    28 |
|             8 |       39.00891  |          40.36969   |                    28 |
|             9 |       40.36969  |          41.730464  |                    28 |
|            10 |       41.730465 |          43.998425  |                    28 |
|            11 |       44.452015 |          52.16308   |                    28 |
|            12 |       52.163082 |          56.5       |                    28 |
|            13 |       56.517609 |          57.6       |                    28 |
|            14 |       57.6      |          61.688512  |                    28 |
|            15 |       61.688512 |          62.142104  |                    28 |
|            16 |       62.142104 |          62.595696  |                    28 |
|            17 |       62.595696 |          62.595696  |                    28 |
|            18 |       62.595696 |          63.049288  |                    28 |
|            19 |       63.049288 |          63.50288   |                    28 |
|            20 |       63.50288  |          63.7750352 |                    28 |
|            21 |       63.7750352|          64.1379088 |                    28 |
|            22 |       64.1379088|          64.410064  |                    28 |
|            23 |       64.410064 |          64.5915008 |                    28 |
|            24 |       64.5915008|          64.6822192 |                    28 |
|            25 |       64.6822192|          64.863656  |                    28 |
|            26 |       64.863656 |          64.9543744 |                    28 |
|            27 |       64.9543744|          65.1358112 |                    28 |
|            28 |       65.1358112|          65.317248  |                    28 |
|            29 |       65.317248 |          65.77084   |                    28 |
|            30 |       65.77084  |          66.224432  |                    28 |
|            31 |       66.224432 |          67.58526313|                    28 |
|            32 |       67.58526313|          67.58526313|                    28 |
|            33 |       67.58526313|          67.58526313|                    28 |
|            34 |       67.58526313|          67.58526313|                    28 |
|            35 |       67.58526313|          67.812059315|                    28 |
|            36 |       67.812059315|         68.49244787|                    28 |
|            37 |       68.49244787|          68.49244787|                    28 |
|            38 |       68.49244787|          68.49244787|                    28 |
|            39 |       68.49244787|          68.49244787|                    28 |
|            40 |       68.49244787|          68.719244055|                    28 |
|            41 |       68.719244055|         69.39963261|                    28 |
|            42 |       69.39963261|          70         |                    28 |
|            43 |       70        |          71.213944  |                    28 |
|            44 |       71.213944 |          73.481904  |                    28 |
|            45 |       73.48196394|          74.162292  |                    28 |
|            46 |       74.162352495|         74.615884  |                    28 |
|            47 |       74.615884 |          75         |                    28 |
|            48 |       75        |          75.296272  |                    28 |
|            49 |       75.296272 |          75.5       |                    28 |
|            50 |       75.5      |          76.203456  |                    28 |
|            51 |       76.203456 |          76.883906715|                    28 |
|            52 |       76.883906715|         77.337499085|                    28 |
|            53 |       77.337499085|         77.791091455|                    28 |
|            54 |       77.791091455|         78.01788764 |                    28 |
|            55 |       78.01788764 |         78.698212   |                    28 |
|            56 |       78.698276195|         78.698276195|                    28 |
|            57 |       78.698276195|         79.151804   |                    28 |
|            58 |       79.151804 |          79.151868565|                    28 |
|            59 |       79.151868565|         79.605460935|                    28 |
|            60 |       79.605460935|         79.83225712 |                    28 |
|            61 |       79.83225712 |         80.013691299|                    28 |
|            62 |       80.058988 |          80.739376   |                    28 |
|            63 |       80.739376 |          83.91452    |                    28 |
|            64 |       84        |          86.18248    |                    28 |
|            65 |       86.18248  |          87.54332741 |                    28 |
|            66 |       87.54332741 |         88.45044    |                    28 |
|            67 |       88.5      |          88.994822994|                    28 |
|            68 |       88.999811846|         89.75       |                    28 |
|            69 |       89.759002686|         90.7184     |                    28 |
|            70 |       90.7184   |          91.55       |                    28 |
|            71 |       91.625584 |          92.98636    |                    28 |
|            72 |       92.98636  |          94          |                    28 |
|            73 |       94.347136 |          96.05       |                    28 |
|            74 |       96.05     |          96.433740631|                    28 |
|            75 |       96.524456336|         96.887333001|                    28 |
|            76 |       96.9      |          97.703796498|                    28 |
|            77 |       97.73400116 |         98.88313666 |                    28 |
|            78 |       99        |         100.697426   |                    27 |
|            79 |       100.697426 |         102.2        |                    27 |
|            80 |       102.398950789|        103.872568   |                    27 |
|            81 |       103.963378906|        107.047712   |                    27 |
|            82 |       107.047714 |         108.86208    |                    27 |
|            83 |       108.86208  |         112.037224   |                    27 |
|            84 |       112.037224 |         114.9402128  |                    27 |
|            85 |       114.985572 |         117.5        |                    27 |
|            86 |       117.5710464|         119.1        |                    27 |
|            87 |       119.1      |         120.4        |                    27 |
|            88 |       120.4      |         121.4719376  |                    27 |
|            89 |       121.4719376|         122          |                    27 |
|            90 |       122.050003052|        122.9        |                    27 |
|            91 |       122.9      |         123.450004578|                    27 |
|            92 |       123.5      |         124.5        |                    27 |
|            93 |       124.5      |         127.4        |                    27 |
|            94 |       127.43800354 |        129.762008667|                    27 |
|            95 |       129.77278  |         130.52802    |                    27 |
|            96 |       130.5389   |         131.54168    |                    27 |
|            97 |       131.54169  |         132.6599     |                    27 |
|            98 |       132.736    |         133.765      |                    27 |
|            99 |       133.80965  |         136.0776     |                    27 |
|           100 |       136.0776   |         200.487664   |                    27 |

# Closing Remarks
So this has been a long tutorial and you might be wondering why we took such a long roundabout way to cover the cumulative distributions first before going straight to a histogram. To answer these queries - I pose a few questions of my own for you to think about as we close this session:

1. How would the histogram chart look if we didn’t remove any outliers?
2. How would you have found the outliers without inspecting the percentiles?
3. How can you determine the percentiles if you only had the frequency values from the histogram?

These two tools of cumulative distribution functions and the frequency histogram are really important for different reasons and each should be part of your data exploration toolbox!


# Conclusion
Wow! Congratulations for sticking with me for this entire tutorial - I bet you weren’t expecting to cover some data visualization too! Here is what you covered successfully in this session:

- Mathematical representation of cumulative distribution functions and their SQL implementation
- Reverse engineer a table output using the `NTILE` function and generate a cumulative distribution output
- Apply critical thinking to accurately detect outliers in the upper and lower ranges of a column by inspecting the ordered top and bottom percentile values
- Inspected the difference between `ROW_NUMBER`, `RANK` and `DENSE_RANK` window functions
- Treat outliers in a dataset by applying a `WHERE` filter and creating a temporary table for further data analysis
- Generate new summary statistics and a final cumulative distribution function output with a treated dataset cleaned of outliers
