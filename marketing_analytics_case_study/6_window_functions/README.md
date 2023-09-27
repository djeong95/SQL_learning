# Window Functions

In this tutorial we will look into window functions which are an absolutely critical component of our marketing analytics case study.

This guide will aim to cover almost everything you’ll need to know about all the window functions and how to apply them in multiple different scenarios as you encounter various problems in your data journey.

Previously we have seen some ways to use window functions multiple times throughout Serious SQL so far. We just used the PERCENT_RANK and ROW_NUMBER window functions in our last SQL problem solving tutorial, however - we are yet to really dig into the details and understand what is actually going on under the hood!

Window functions or analytical functions are used practically everywhere in the data world and they are super useful! However they are not the most simple or intuitive to understand at first - this tutorial is here to help you gain that deep level of understanding.

Apologies in advance for the length of this tutorial, as well as the variety of new and old datasets which will be introduced for only this tutorial!

## Introduction

After failing to truly understand window functions as a junior data analyst - I realised many years later that there was a better way to frame up the learning process.

Some of the main concepts we’ll cover in this section of the tutorial include:

- The different components that make up a window function
- How window functions are different to regular group by functions
- Basic operations we can perform using window functions
- Advanced applications of window functions

Firstly - let’s start with some basic definitions.

# Basic Components

Window functions are operations or calculations performed on “window frames” or put simply, groups of rows in a dataset.

Just a warning that some of these things I’m going to describe in the following section might not instantly make sense, but bear with it - because after the next few following sections, hopefully things will become much clearer!

Window functions consist of the following components:

<img width="596" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/26a3ebd1-7a65-4bbb-91d7-759e5fcf687d">

After first viewing this - likely it makes little to no sense, so let’s break this down into a very simple example and build up our understanding from first principles.

# Understanding Partition By

One of the easiest ways to understand the how the PARTITION BY component of window functions worked was to compare it to a basic GROUP BY query.

Let’s start with this super simple example of an aggregate group by sum and its equivalent basic sum window function.

The key things to note are the similarity of the sum_sales column output for each example - they are exactly the same!

<img width="592" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/c571b0f3-755c-4f4f-8d16-62212eed76d9">

You can find the same dataset below with code used to create a temporary table so you can code-along with the visual examples:

```sql
-- This is not in BigQuery
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sales) AS (
 VALUES
 ('A', 300),
 ('A', 150),
 ('B', 100),
 ('B', 200)
)
SELECT * FROM input_data;

-- Group By Sum
-- Note that the ORDER BY is only for output sorting purposes!
SELECT
  customer_id,
  SUM(sales) AS sum_sales
FROM customer_sales
GROUP BY customer_id
ORDER BY customer_id;

-- Sum Window Function
SELECT
  customer_id,
  sales,
  SUM(sales) OVER (
    PARTITION BY customer_id
  ) AS sum_sales
FROM customer_sales;
```

## Partition By 2 Columns

We’ve already seen how a single column can be used with the PARTITION BY clause - let’s now inspect what happens when we partition by 2 columns.

In the visual example below - you can think of the partitioning as splitting the dataset into smaller groups based on the unique combination of column values from each input to the PARTITION BY

<img width="525" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/1803e060-67fa-4d37-a311-2ec7a546bde3">

Also note that we are not restricted to only using columns as inputs, we can use other derived expressions also, just like you would use in a regular SELECT statement.

```sql
-- we remove that existing customer_sales table first!
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sale_id, sales) AS (
 VALUES
 ('A', 1, 300),
 ('A', 1, 150),
 ('A', 2, 100),
 ('B', 3, 200)
)
SELECT * FROM input_data;

-- Sum Window Function with 2 columns in PARTITION BY
SELECT
  customer_id,
  sales,
  SUM(sales) OVER (
    PARTITION BY
      customer_id,
      sale_id
  ) AS sum_sales
FROM customer_sales;
```

## Multiple Level Partition By

We can also use different levels for multiple window functions in a single query - there is one very specific reason why we usually prefer to use window functions to perform multiple level aggregations in this way compared to other methods.

We also demonstrate the empty OVER clause in the query below to calculate the total_sales column for our dataset.

<img width="545" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/2fa3fecd-d211-44ab-9f0f-4fb2c6f03a83">

```sql
SELECT
  customer_id,
  sale_id,
  sales,
  SUM(sales) OVER (
    PARTITION BY
      customer_id,
      sale_id
  ) AS sum_sales,
  SUM(SALES) OVER (
    PARTITION BY customer_id
  ) AS customer_sales,
  SUM(SALES) OVER () AS total_sales
FROM customer_sales;
```

## Multiple Calculations

We can also apply multiple different window function calculations instead of just using the SUM function like we’ve been using for all the previous examples.

In the following example, we demonstrate how to use AVG and MAX window functions.

<img width="548" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/7d5a053a-960b-400f-ba18-51497b8e303a">

```sql
SELECT
  customer_id,
  sale_id,
  sales,
  SUM(sales) OVER (
    PARTITION BY
      customer_id,
      sale_id
  ) AS sum_sales,
  -- the average customer sales is rounded to 2 decimals!
  ROUND(
    AVG(sales) OVER (
      PARTITION BY customer_id
    ),
    2
  ) AS avg_cust_sales,
  MAX(sales) OVER () AS max_sales
FROM customer_sales;
```

## Empty or Missing Partition By

We mentioned that the default behaviour for PARTITION BY when the window function has an empty OVER clause is to perform calculations across all the rows of the dataset - in fact, we’ve also used this exact strategy before for parts of our data exploration section earlier!

This is directly copied from the Dealing with Duplicates tutorial earlier when we were inspecting the health.user_logs table to calculate the percentage of values of the various measures available in the dataset.

> Let’s also inspect that measure column and take a look at the most frequent values within this column using a GROUP BY and ORDER BY DESC combo from the last tutorial - let’s also throw in that percentage column that we went through also!

```sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY frequency DESC;
```

| measure        | frequency | percentage |
|----------------|-----------|------------|
| blood_glucose  | 38692     | 88.15      |
| weight         | 2782      | 6.34       |
| blood_pressure | 2417      | 5.51       |

The key thing to note here is that this is a more specialized form of window function where we also use a GROUP BY in the same query!

This actually takes us nicely into a slight detour to something which is super important, especially in the context of window functions - the logical order of SQL query execution!

# SQL Logical Execution Order

In a nutshell - all SQL queries are “ran” in the following order:

1. FROM
    - WHERE filters
    - ON table join conditions
2. GROUP BY
3. SELECT aggregate function calculations
4. HAVING
5. Window functions
6. ORDER BY
7. LIMIT

Note: that the actual execution order might differ slightly from the below due to the SQL optimizer making its own decisions for performance reasons!

We can use our previous query to break this down and understand how it works - can you identify which SQL components we have below?

```sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY frequency DESC;
```

In the following parts of this tutorial - we will reconstruct our above query from the ground up to see how each component interacts with eachother to build up our understanding of the logical execution order.

# Ordered Window Functions

In the previous window function examples - we have always been dealing with records which did not need to be ordered, we only applied the PARTITION BY clause to define the window frame.

In the following section we will start looking into that ORDER BY component of window functions and start to understand what it is doing - in a nutshell, the ORDER BY acts in exactly the same way as a regular ORDER BY clause would act in a standard SQL query.

Logically - we can think of the ORDER BY happening after the PARTITION BY clause as the sorting of records will happen within each partition or group that is separated as part of the window function.

<img width="537" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/c222b7b7-fbe7-4c6c-80a5-9982351c606e">

We can also use a ORDER BY DESC to sort our rows differently and it will provide a reversed outcome!

<img width="534" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/c02c6aed-fb69-4466-85ba-562654bf0e42">

```sql
-- we remove any existing customer_sales table first!
DROP TABLE IF EXISTS customer_sales;
CREATE TEMP TABLE customer_sales AS
WITH input_data (customer_id, sales_date, sales) AS (
 VALUES
 ('A', '2021-01-01'::DATE, 300),
 ('A', '2021-01-02'::DATE, 150),
 ('B', '2021-01-03'::DATE, 100),
 ('B', '2021-01-02'::DATE, 200)
)
SELECT * FROM input_data;

-- RANK Window Function with default ORDER BY
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    PARTITION BY customer_id
    ORDER BY sales_date
  ) AS sales_date_rank
FROM customer_sales;

-- RANK Window Function with descending ORDER BY
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    PARTITION BY customer_id
    ORDER BY sales_date DESC
  ) AS sales_date_rank
FROM customer_sales;
```

Note that if there is an empty partition clause - the ORDER BY will still take place - just on all the rows of the dataset as opposed within the separated groups like we would see with a non-empty PARTITION BY clause.

```sql
-- RANK Window Function with descending ORDER BY and empty PARTITION BY
SELECT
  customer_id,
  sales_date,
  sales,
  RANK() OVER (
    ORDER BY sales_date DESC
  ) AS sales_date_rank
FROM customer_sales;
```

| customer_id | sales_date | sales | sales_date_rank |
|-------------|------------|-------|-----------------|
| B           | 2021-01-03 | 100   | 1               |
| A           | 2021-01-02 | 150   | 2               |
| B           | 2021-01-02 | 200   | 2               |
| A           | 2021-01-01 | 300   | 4               |


## Different Ordering Window Functions

The most popular ordered window function calculations are the following - note that all of them do not need any inputs:

These functions return integers:

- ROW_NUMBER()
- RANK()
- DENSE_RANK()

These following functions return numeric outputs between 0 and 1:

- PERCENT_RANK()
- CUME_DIST()

One other function we’ve seen before is the NTILE function which we used in the Data Exploration section.

NTILE is the odd one out from this set of functions as it requires a single integer input to define the number of buckets or end ranges from 1 to the input - for example our example uses NTILE(100) to assign a number between 1 and 100 for our dataset based off the distribution of each record.

Let’s revisit a slightly different version of this query so we can dig into our window functions a little bit further and understand their differences:

## Default Ascending Order By

The start of our base query is simply all of the weight values from the health.user_logs table - the first 10 records look like this:

```sql
SELECT
  measure,
  measure_value
FROM health.user_logs
WHERE measure = 'weight'
LIMIT 10;
```

# Lag & Lead Window Functions

Lag and Lead window functions do not perform calculations on the window frame records - but rather they simply grab the value before or after the current row respectively.

Traditionally - these window functions usually used with a non-empty ORDER BY clause otherwise you will return arbitrarily ordered results - which defeats the purpose of using the lag and lead functions in the first place! (unless you have a very very specific use case…)

These 2 functions are super useful but also slightly tricky to wield because of one simple reason - they can be manipulated to do exactly the same thing!?

## Identify Null Rows

There is actually a really elegant method to identify all rows with any null values in any column, so long as the columns that you need to check are NUMERIC types.

There is a concept called the propogation of null values whereby if you add null value to a number - it will simply return a null!

We can use this to filter out our trading.daily_btc table to hunt for those null rows of data.

```sql
SELECT *
FROM trading.daily_btc
WHERE (
  open_price + high_price + low_price +
  close_price + adjusted_close_price + volume
) IS NULL;
```
We will use these dates for our investigation into LAG and LEAD window functions so keep this following WHERE filter on hand and ready to be used below!

```sql
WHERE market_date IN (
  '2020-04-17',
  '2020-10-09',
  '2020-10-12',
  '2020-10-13'
)
```

Alternatively - we could have also just done a basic where filter with every single column being used as below:

```sql
SELECT *
FROM trading.daily_btc
WHERE
  market_date IS NULL
  OR open_price IS NULL
  OR high_price IS NULL
  OR low_price IS NULL
  OR close_price IS NULL
  OR adjusted_close_price IS NULL
  OR volume IS NULL
;
```

## Data Philosophy

Now before we dive in to the solution - let’s talk about a philosophical aspect of data - in particular - data that is coming from the past, present and future!

The majority of the time - when we perform analysis on datasets, they are usually historical records. This means that it is a representation of what has already happened: the past

You might have heard about real-time or streaming data - this is a view of what is currently happeneing or in other words: the present

And when we are dealing with predictions or forecasts or anything where we need to look into time periods where we do not have data: the future

One of the most common pitfalls of data analysis, data science and machine learning is this mismatch between past, present and future data.

When we look at historical data, usually we will want to analyse what has happened in the past and compare it to what is happening right now or in recent times. The next natural step is to use those insights to make assessments or predictions about the future - note that these do not need to be machine learning based predictions but they could be super simple like the following scenario:

**Past**

> Historic average temperatures in Sydney Australia during the summer months of December to February are relatively high compared to winter temperatures

**Present**

> The current summer season average temperature is higher than the historic average

**Future**

> We predict that the summer average temperature will continue to rise in the coming years

There is one base concept which is almost like a commandment in the data world:

> **You must not leak future data into the past**

This might not sink in straightaway - but it is really really important. Please keep this point in mind as you read the rest of this tutorial about LAG and LEAD window functions!

## Filling Null Values

Remember those missing value rows we identified previously? We’ll now use both our LAG and LEAD window functions to demonstrate how we can fill these values - but we’ll also take a deeper inspection into why one method is better than the other when we are dealing with time based data!

For our Bitcoin data example - we will need to fill in the missing values from our rows of data before we continue with our analysis. Usually null values will be ignored in many functions - however we want to make sure there are no null values in our data because there really should be data for every single day!

Welcome to the world of true messy data :)

### Data Before and After
Let’s first inspect one of our null record values and the day before and after the date ‘2020-04-17’ from the trading.daily_btc dataset using a BETWEEN:

```sql
SELECT * FROM trading.daily_btc
WHERE market_date BETWEEN ('2020-04-17'::DATE - 1) AND ('2020-04-17'::DATE + 1);
```
| market_date | open_price  | high_price  | low_price   | close_price | adjusted_close_price |      volume      |
|-------------|-------------|-------------|-------------|-------------|----------------------|------------------|
| 2020-04-16  | 6640.454102 | 7134.450684 | 6555.504395 | 7116.804199 | 7116.804199          | 46783242377      |
| 2020-04-17  | null        | null        | null        | null        | null                 | null             |
| 2020-04-18  | 7092.291504 | 7269.956543 | 7089.247070 | 7257.665039 | 7257.665039          | 32447188386      |

### Null Value Filling Methods
Now let’s look at our options for how we should actually fill in the null values:

1. Use the column average
2. Fill with 0

Can you identify what’s wrong with the above 2 methods? You’re probably thinking to yourself - “But Danny - these are the most common null filling methods used for machine learning!”

And yes - you are correct, these are the most common null filling methods seen everywhere on Kaggle and blogs around the internet - but we need to ask ourselves - do they make sense for our problem???

We have daily Bitcoin prices - and we want to fill in a missing day’s worth of data so we can use it for our analysis without removing any data (that’s also another option - but we’re not going to do that here!)

Filling it with the average of the entire dataset should be not be done - what did we just mention in the philosophical section above?

Filling it 0 makes zero sense too :) Unless the ENTIRE blockchain went down on a day and there was actually no trading activity that is…

So what else could we try?

3. Use the next day’s data

No. See the philosophical section to remember the past, present and future!

4. Use the previous day’s data

Bingo!!!

Well in reality - we could also do more complex things like predicting or forecasting what the missing day’s worth of data was based off even more past data, but let’s imagine that we didn’t want to do that and we want to take the simplest option!

We can imagine that the missing day’s worth of trading data was exactly the same as the day before.

It’s not a perfect representation of what might have actually happened on that day where the data is missing - but at least it’s something that we can use and won’t impact downstream calculations too much (hopefully!)

So let’s say that we will make this data manipulation happen - how can we actually do this?

Enter the LAG window function!

### Lag To Fill Missing Values

We can simply use the LAG window function OVER the entire dataset - but remember that we will always want to use an ORDER BY clause for ordered window functions and LAG is just one of these ordered window functions!

The LAG calculation function accepts a single column name as its first function input and also accepts a second optional input as the OFFSET or number of rows to use for the calculation (more on this later!) - by default the OFFSET value is set to 1.

There is also a 3rd optional input for the LAG and LEAD functions which defines the DEFAULT value - when it is not specific explicitly, the value is set to null - you will see this in action below but it essentially uses this DEFAULT value when there are no records to perform the LAG calculation (i.e. there are no rows before this current row!)

Also note that great care has to be taken when using this input as the data-type must match the column that is being used as the first input to the LAG or LEAD function.

Let’s demonstrate how to implement this for just the open_price column below. The LAG function without any additional input apart from the column works by simply taking the target column’s value for the first row before it within the window frame (i.e: an offset of 1)

Usually in practice you will rarely see the OFFSET or DEFAULT function inputs used - but in the query below we will use just the OFFSET value. Also take note that these function inputs MUST be used in the specified order. You can find out more at the PostgreSQL window functions documentation page - [Click here to open the docs in a new tab](https://www.postgresql.org/docs/12/functions-window.html)!

```sql
SELECT
  market_date,
  open_price,
  LAG(open_price, 1) OVER (ORDER BY market_date) AS lag_open_price 
FROM `younglapalma.trading.daily_btc`
WHERE market_date BETWEEN (CAST('2020-04-17' AS DATE) - 1) AND (CAST('2020-04-17' AS DATE) + 1)
ORDER BY market_date ASC;
```

| market_date | open_price  | lag_open_price |
|-------------|-------------|----------------|
| 2020-04-16  | 6640.454102 | null           |
| 2020-04-17  | null        | 6640.454102    |
| 2020-04-18  | 7092.291504 | null           |

What do you notice about the output above? A few questions might come to mind:

- Why don’t I get a lag_open_price for April the 16th?

The LAG function looks at the row prior to the current row within the specified window frame for the target column value - since there is no rows before April 16th in our window frame it returns a null.

But wait a moment - don’t we have data for the 15th of April in our original dataset - how come it isn’t showing up in this window frame?

- Can you see why the row for the 15th of April is not used for our window function? (Hint: it has something to do with the logical order of execution!)

This exact point is a perfect place to demonstrate what we meant when talked about the 3rd optional function input for the LAG and LEAD functions:

Let’s demonstrate how we can change the 3rd optional DEFAULT input in the query below to 6,000:

```sql
SELECT
  market_date,
  open_price,
  LAG(open_price, 1, 6000) OVER (ORDER BY market_date) AS lag_open_price 
FROM `younglapalma.trading.daily_btc`
WHERE market_date BETWEEN (CAST('2020-04-17' AS DATE) - 1) AND (CAST('2020-04-17' AS DATE) + 1)
ORDER BY market_date ASC;
```

| market_date | open_price  | lag_open_price |
|-------------|-------------|----------------|
| 2020-04-16  | 6640.454102 | 6000           |
| 2020-04-17  | null        | 6640.454102    |
| 2020-04-18  | 7092.291504 | null           |

Nice so it looks like we can replace that first row null value to 6,000 - but what about that null value in the last row for April 18th?

Because we used an ORDER BY market_date - the window frame is sorted such that April 17th is the row directly before April 18th - and what is the value for the open_price column?

It is also null - so it’s only expected that our LAG window function would return this value too!

Now that we’ve covered the inputs of the LAG function - there is the next confusing thing: we can do this exact same operation but using the LEAD function!

### Lead Alternate Implementation

The LEAD window function is exactly the same as the LAG window function but instead of looking at the previous row in the window frame - it looks at the following row!

We can actually perform the same null filling strategy of taking the previous day’s value for our Bitcoin dataset - by changing the order of our ORDER BY clause to rearrange the window frame row order!

Let’s add a new column to our previous query for the lead_open_price:

```sql
SELECT
  market_date,
  open_price,
  LAG(open_price) OVER (ORDER BY market_date) AS lag_open_price,
  LEAD(open_price) OVER (ORDER BY market_date DESC) AS lead_open_price
FROM `younglapalma.trading.daily_btc`
WHERE market_date BETWEEN (CAST('2020-04-17' AS DATE) - 1) AND (CAST('2020-04-17' AS DATE) + 1)
ORDER BY market_date ASC;
```

| market_date | open_price  | lag_open_price | lead_open_price |
|-------------|-------------|----------------|-----------------|
| 2020-04-16  | 6640.454102 | null           | null            |
| 2020-04-17  | null        | 6640.454102    | 6640.454102     |
| 2020-04-18  | 7092.291504 | null           | null            |

Well what do you know? They are exactly the same…or are they?!?

### Philosophy Round 2

Here is where our philosophical concept really kicks in - can you identify the issue with using a LEAD window function to calculate the previous day’s value in our time series example?

When we are dealing with data - the language that we use to explain what we are doing with our transformations and manipulations is really important!

Some people brush this off as just “syntax” or “verbatim” but in reality - there are multiple reasons why we should avoid this!

When reading and reviewing code - the function that is used is important!

When you see a LEAD window function being used - a skilled SQL developer will instantly think that you are obtaining data points from a following row.

However - as you continue to read the code and see an ORDER BY DESC - you need to process an additional component, adding to the mental complexity of the function - now you need to replace the “following row” concept in your mind with the “previous row” - but still stay aware of the fact that you are using a LEAD function and not a LAG function.

See - this last paragraph is confusing as heck and I’m not even reviewing any specific code!!!

Instead of doing this round-about logical acrobatics - wouldn’t it be so much simpler if you just used the right function for the right situation???

So whenever you need to obtain a previous row with a time based or an incremental based ORDER BY criteria - ALWAYS use LAG and a default ascending ORDER BY clause!

The reverse is true if you ever need a “future” value for the current row - use the LEAD function with a default ascending ORDER BY clause!

Ok - time to end my rant so we can continue with our example!

## Coalesce to Update Null Rows

So now that we’ve learnt about the proper use of a LAG function (and the improper use of the LEAD function) - how can we use this to actually help us update the dataset so we can fill in these null values once and for all?

Enter the COALESCE function - let’s demonstrate how we can use this with our previous example with the 17th of April data only inside a CTE:

```sql
WITH april_17_data AS (SELECT
  market_date,
  open_price,
  LAG(open_price) OVER (ORDER BY market_date) AS lag_open_price,
  LEAD(open_price) OVER (ORDER BY market_date DESC) AS lead_open_price
FROM `younglapalma.trading.daily_btc`
WHERE market_date BETWEEN (CAST('2020-04-17' AS DATE) - 1) AND (CAST('2020-04-17' AS DATE) + 1)
ORDER BY market_date ASC)
SELECT 
  market_date,
  open_price,
  lag_open_price,
  COALESCE(open_price, lag_open_price) AS coalesce_open_price
FROM april_17_data;
```

| market_date | open_price  | lag_open_price | coalesce_open_price |
|-------------|-------------|----------------|---------------------|
| 2020-04-16  | 6640.454102 | null           | 6640.454102         |
| 2020-04-17  | null        | 6640.454102    | 6640.454102         |
| 2020-04-18  | 7092.291504 | null           | 7092.291504         |

This looks great - and neatly demonstrates what the COALESCE function does - just note the function can actually accept any number of columns or even raw values (as long as they are the data type) - the order is important as it will return the first non-null value from the inputs from left to right!

## Update Tables

Now that we’ve learnt how to use the COALESCE and the LAG function - let’s now create a new temporary table which we will use for the rest of this tutorial by updating all of the columns and null rows with what we’ve learnt - but let’s make this more difficult by forcing you to do this 2 ways!

### Create New Temp Table

Let’s first create a new temporary table based off a query:

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price) OVER (ORDER BY market_date)
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price) OVER (ORDER BY market_date)
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price) OVER (ORDER BY market_date)
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price) OVER (ORDER BY market_date)
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price) OVER (ORDER BY market_date)
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume) OVER (ORDER BY market_date)
  ) AS volume
FROM `younglapalma.trading.daily_btc`;

-- test that our previously missing value dates are filled!
SELECT *
FROM updated_daily_btc
WHERE market_date IN (
  '2020-04-17',
  '2020-10-09',
  '2020-10-12',
  '2020-10-13'
)
ORDER BY market_date;
```

| market_date | open_price   | high_price   | low_price    | close_price  | adjusted_close_price | volume      |
|-------------|--------------|--------------|--------------|--------------|----------------------|-------------|
| 2020-04-17  | 6640.454102  | 7134.450684  | 6555.504395  | 7116.804199  | 7116.804199          | 46783242377 |
| 2020-10-09  | 10677.625000 | 10939.799805 | 10569.823242 | 10923.627930 | 10923.627930         | 21962121001 |
| 2020-10-12  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19968627060 |
| 2020-10-13  | null         | null         | null         | null         | null                 | null        |

Oh no!!! What happened to that 13th of October value???

Let’s take a look at a few records around that date from our newly created temporary table first:

```SQL
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price) OVER (ORDER BY market_date)
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price) OVER (ORDER BY market_date)
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price) OVER (ORDER BY market_date)
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price) OVER (ORDER BY market_date)
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price) OVER (ORDER BY market_date)
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume) OVER (ORDER BY market_date)
  ) AS volume
FROM `younglapalma.trading.daily_btc`;

-- test that our previously missing value dates are filled!
SELECT *
FROM updated_daily_btc
WHERE market_date BETWEEN '2020-10-10' AND '2020-10-14'
ORDER BY market_date;
```

| market_date | open_price   | high_price   | low_price    | close_price  | adjusted_close_price | volume      |
|-------------|--------------|--------------|--------------|--------------|----------------------|-------------|
| 2020-10-10  | 11059.142578 | 11442.210938 | 11056.940430 | 11296.361328 | 11296.361328         | 22877978588 |
| 2020-10-11  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19968627060 |
| 2020-10-12  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19968627060 |
| 2020-10-13  | null         | null         | null         | null         | null                 | null        |
| 2020-10-14  | 11429.047852 | 11539.977539 | 11307.831055 | 11429.506836 | 11429.506836         | 24103426719 |

What about our original dataset?

```SQL
SELECT *
FROM trading.daily_btc
WHERE market_date BETWEEN '2020-10-10'::DATE AND '2020-10-14'::DATE;
```

| market_date | open_price   | high_price   | low_price    | close_price  | adjusted_close_price | volume      |
|-------------|--------------|--------------|--------------|--------------|----------------------|-------------|
| 2020-10-10  | 11059.142578 | 11442.210938 | 11056.940430 | 11296.361328 | 11296.361328         | 22877978588 |
| 2020-10-11  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19968627060 |
| 2020-10-12  | null         | null         | null         | null         | null                 | null        |
| 2020-10-13  | null         | null         | null         | null         | null                 | null        |
| 2020-10-14  | 11429.047852 | 11539.977539 | 11307.831055 | 11429.506836 | 11429.506836         | 24103426719 |

Well - what were the values for the 12th of October in our original dataset? Oh yeah - they were also null…D’oh!

Ok - so what should we do now? There is no data for the 12th - but there is data for the 11th of October…if only there was a way to perform the LAG function for more than just 1 row before the current row in the window frame….

### Lag Function With Offset Input

Remember what I mentioned about the second LAG function input - the offset value? By default if we only use a column reference for the LAG function, it will use 1 as the default offset value - meaning that it will take the row directly before the current row during the window function calculation step.

We will only demonstrate the different options on just the open_price column so we don’t need to write so much code - we’ll also keep a filter on just the few days around the 13th of October below.

First - let’s also use a CASE WHEN to specify the equivalent of an IF-ELSE block in SQL - always remember to END your CASE WHEN statements - it’s a very common SQL syntax error!

```sql
SELECT
  market_date,
  open_price,
  COALESCE(
    open_price,
    CASE
      WHEN market_date = '2020-10-13'
        THEN LAG(open_price, 2) OVER (ORDER BY market_date)
      ELSE LAG(open_price, 1) OVER (ORDER BY market_date)
      END
  ) AS adj_open_price
FROM trading.daily_btc
WHERE market_date BETWEEN '2020-10-10'::DATE AND '2020-10-13'::DATE;
```

| market_date | open_price   | adj_open_price |
|-------------|--------------|----------------|
| 2020-10-10  | 11059.142578 | 11059.142578   |
| 2020-10-11  | 11296.082031 | 11296.082031   |
| 2020-10-12  | null         | 11296.082031   |
| 2020-10-13  | null         | 11296.082031   |

Another option is to use an additional COALESCE input since it can take more than just 2 inputs at a time! Remember what we mentioned about the importance of the order of inputs for COALESCE?

```sql
SELECT
  market_date,
  open_price,
  COALESCE(
    open_price,
      LAG(open_price, 1) OVER (ORDER BY market_date),
      LAG(open_price, 2) OVER (ORDER BY market_date)
    )
    AS adj_open_price
FROM trading.daily_btc
WHERE market_date BETWEEN '2020-10-10'::DATE AND '2020-10-13'::DATE
;
```

| market_date | open_price   | adj_open_price |
|-------------|--------------|----------------|
| 2020-10-10  | 11059.142578 | 11059.142578   |
| 2020-10-11  | 11296.082031 | 11296.082031   |
| 2020-10-12  | null         | 11296.082031   |
| 2020-10-13  | null         | 11296.082031   |

So now that we’ve figured out a few ways to deal with this data issue - how can we implement the required changes and update our table so we can get on with our problem?

Well - there are a few ways, and I’ll show you how to do all of them because this course is called Serious SQL after all!

## Window Clause Simplification

If you find that you are repeating a certain window multiple times in the same query - you can actually use the WINDOW clause after the final FROM section of your SQL query and refer to a simple alias in the window function call within the select expressions in the body of the SQL query.

This is awesome because it ticks off a few boxes in terms of having nice looking code:

Harder to make mistakes because we only need to specify the window once
Sticks to the DRY principle (DO NOT REPEAT YOURSELF!)
Keeps the SELECT body of the SQL query very clean looking
Let’s use this new technique for our previous query - take a look at the final line of the query first to see how we can define this window!

Note that you can actually define multiple windows by using a comma separated list as shown below - you don’t even need to use them in the query - just be careful what you name your windows and do not get them mixed up with default SQL functions or any other variable/table name that you’ve created in the database session!


```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);
-- NOTE: checkout the syntax where I've included an unused window below!

-- inspect a few rows of the updated dataset for October
SELECT *
FROM updated_daily_btc
WHERE market_date BETWEEN CAST('2020-10-08' AS DATE) AND CAST('2020-10-15' AS DATE)
ORDER BY market_date;
```

| market_date | open_price   | high_price   | low_price    | close_price  | adjusted_close_price | volume         |
|-------------|--------------|--------------|--------------|--------------|----------------------|----------------|
| 2020-10-08  | 10677.625000 | 10939.799805 | 10569.823242 | 10923.627930 | 10923.627930         | 21,962,121,001 |
| 2020-10-09  | 10677.625000 | 10939.799805 | 10569.823242 | 10923.627930 | 10923.627930         | 21,962,121,001 |
| 2020-10-10  | 11059.142578 | 11442.210938 | 11056.940430 | 11296.361328 | 11296.361328         | 22,877,978,588 |
| 2020-10-11  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19,968,627,060 |
| 2020-10-12  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19,968,627,060 |
| 2020-10-13  | 11296.082031 | 11428.813477 | 11288.627930 | 11384.181641 | 11384.181641         | 19,968,627,060 |
| 2020-10-14  | 11429.047852 | 11539.977539 | 11307.831055 | 11429.506836 | 11429.506836         | 24,103,426,719 |
| 2020-10-15  | 11426.602539 | 11569.914063 | 11303.603516 | 11495.349609 | 11495.349609         | 24,487,233,058 |

# Cumulative Calculations

Now that our dataset is finally cleaned up and we no longer have any null values - we can now focus on the next step of our window functions journey and explore how we can perform cumulative calculations!

## Cumulative Sum

Let’s say for example - we also want to add a new column into our updated_daily_btc dataset which contains the cumulative sum of all previous traded volume up to and including that day’s worth of data.

To break this example down in case you haven’t come across cumulative metrics before - let’s investigate just the first 10 rows of the volume column from our updated_daily_btc dataset sorted by market_date and demonstrate what we need to do at a low level.

Take note that the volume metric is the total number of Bitcoins traded on that specific market_date!

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);
-- NOTE: checkout the syntax where I've included an unused window below!

-- inspect a few rows of the updated dataset for October
SELECT market_date,
  volume
FROM updated_daily_btc
ORDER BY market_date
LIMIT 10;
```

| market_date | volume      |
|-------------|-------------|
| 2014-09-17  | 21,056,800  |
| 2014-09-18  | 34,483,200  |
| 2014-09-19  | 37,919,700  |
| 2014-09-20  | 36,863,600  |
| 2014-09-21  | 26,580,100  |
| 2014-09-22  | 24,127,600  |
| 2014-09-23  | 45,099,500  |
| 2014-09-24  | 30,627,700  |
| 2014-09-25  | 26,814,400  |
| 2014-09-26  | 21,460,800  |

So if we wanted to perform a cumulative sum calculation, adding the previous volume amounts to a running total - the calculations would look a bit like this:

| market_date | volume      | running_total                                          |
|-------------|-------------|--------------------------------------------------------|
| 2014-09-17  | 21,056,800  | 21,056,800                                             |
| 2014-09-18  | 34,483,200  | 21,056,800 + 34,483,200                                |
| 2014-09-19  | 37,919,700  | 21,056,800 + 34,483,200 + 37,919,700                   |
| 2014-09-20  | 36,863,600  | 21,056,800 + 34,483,200 + … + 36,863,600               |
| 2014-09-21  | 26,580,100  | 21,056,800 + 34,483,200 + … + 26,580,100               |
| 2014-09-22  | 24,127,600  | 21,056,800 + 34,483,200 + … + 24,127,600               |
| 2014-09-23  | 45,099,500  | 21,056,800 + 34,483,200 + … + 45,099,500               |
| 2014-09-24  | 30,627,700  | 21,056,800 + 34,483,200 + … + 30,627,700               |
| 2014-09-25  | 26,814,400  | 21,056,800 + 34,483,200 + … + 26,814,400               |
| 2014-09-26  | 21,460,800  | 21,056,800 + 34,483,200 + … + 21,460,800               |

How can we do this using SQL? One way is to use something which is known as a self-join where we join one table back onto itself!

We will demonstrate how to do this using a single CTE volume_data with our first 10 rows of volume data.

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

WITH volume_data AS (
  SELECT
    market_date,
    volume
  FROM updated_daily_btc
  ORDER BY market_date
  LIMIT 10
)
SELECT
  t1.market_date,
  t1.volume,
  SUM(t2.volume) AS running_total
FROM volume_data t1
INNER JOIN volume_data t2
  ON t1.market_date >= t2.market_date
GROUP BY 1,2
ORDER BY 1,2;
```

| market_date | volume      | running_total |
|-------------|-------------|---------------|
| 2014-09-17  | 21,056,800  | 21,056,800    |
| 2014-09-18  | 34,483,200  | 55,540,000    |
| 2014-09-19  | 37,919,700  | 93,459,700    |
| 2014-09-20  | 36,863,600  | 130,323,300   |
| 2014-09-21  | 26,580,100  | 156,903,400   |
| 2014-09-22  | 24,127,600  | 181,031,000   |
| 2014-09-23  | 45,099,500  | 226,130,500   |
| 2014-09-24  | 30,627,700  | 256,758,200   |
| 2014-09-25  | 26,814,400  | 283,572,600   |
| 2014-09-26  | 21,460,800  | 305,033,400   |

Some people really enjoy using this self-join technique and likely you will see this being used often in some SQL training examples online.

However - I’ve found that window functions are more efficient and also easier to read and understand and make it a bit clearer in terms of intent - it’s clear for anyone who reads the code what you are trying to do when you have clearly defined window functions, as opposed to someone second guessing why you have a join in the middle of a cumulative sum calculation!

So how can we do this with a SUM window function? Let’s keep that first CTE with the 10 rows to make our lives easier!

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

WITH volume_data AS (
  SELECT
    market_date,
    volume
  FROM updated_daily_btc
  ORDER BY market_date
  LIMIT 10
)
SELECT
  market_date,
  volume,
  SUM(volume) OVER (ORDER BY market_date) AS cumulative_sum
FROM volume_data;
```

| market_date | volume      | running_total |
|-------------|-------------|---------------|
| 2014-09-17  | 21,056,800  | 21,056,800    |
| 2014-09-18  | 34,483,200  | 55,540,000    |
| 2014-09-19  | 37,919,700  | 93,459,700    |
| 2014-09-20  | 36,863,600  | 130,323,300   |
| 2014-09-21  | 26,580,100  | 156,903,400   |
| 2014-09-22  | 24,127,600  | 181,031,000   |
| 2014-09-23  | 45,099,500  | 226,130,500   |
| 2014-09-24  | 30,627,700  | 256,758,200   |
| 2014-09-25  | 26,814,400  | 283,572,600   |
| 2014-09-26  | 21,460,800  | 305,033,400   |

So much simpler than the self-join method, right?!

Is that all we can do with window functions for cumulative metrics? Of course not - this is Serious SQL!

This is the perfect opportunity for us to explore another one of our window functions - this time we will dive deeper into that FRAME clause we mentioned right at the beginning of this tutorial (I hope you haven’t forgotten about it already!)

# Window Frame Clause

The FRAME clause is something that we’ve been implicitly using without really knowing about it up until now for our cumulative sum calculations.

The first thing to know about the FRAME clause is what is actually used as default when we do not explicitly specify them in our window functions.

As stated in the components - the default is:

`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

Firstly - let’s confirm that the default returns the same result as when we actually input the FRAME clause into our window frame OVER clause:

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

SELECT
  market_date,
  volume,
  SUM(volume) OVER (ORDER BY market_date) AS cumulative_sum,
  SUM(volume) OVER (
    ORDER BY market_date
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS default_cumulative_sum
FROM updated_daily_btc
ORDER BY market_date
LIMIT 10;
```

| market_date | volume      | cumulative_sum | default_cumulative_sum |
|-------------|-------------|----------------|------------------------|
| 2014-09-17  | 21,056,800  | 21,056,800     | 21,056,800             |
| 2014-09-18  | 34,483,200  | 55,540,000     | 55,540,000             |
| 2014-09-19  | 37,919,700  | 93,459,700     | 93,459,700             |
| 2014-09-20  | 36,863,600  | 130,323,300    | 130,323,300            |
| 2014-09-21  | 26,580,100  | 156,903,400    | 156,903,400            |
| 2014-09-22  | 24,127,600  | 181,031,000    | 181,031,000            |
| 2014-09-23  | 45,099,500  | 226,130,500    | 226,130,500            |
| 2014-09-24  | 30,627,700  | 256,758,200    | 256,758,200            |
| 2014-09-25  | 26,814,400  | 283,572,600    | 283,572,600            |
| 2014-09-26  | 21,460,800  | 305,033,400    | 305,033,400            |


## Window Frame Components

So we can confirm that the defaults are being applied for our cumulative sum window function - but what does all this mean?

The following diagram will break down the different components of our window frame clause so we can start understanding how this works and most importantly - how we can use them to generate the different outputs that we need!

There is a lot of information in the visual below but don’t worry - we will cover everything soon!

<img width="543" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/a8d1ce5c-3a3e-431c-9dc3-7b732904a571">

These components are difficult to explain separately as they need to interact with the other components for it to make sense.

The various combinations of window frame options are determined by the intersection of 3 components:

**Window Frame Modes**
- RANGE vs ROWS (vs GROUPS)

**Start and End Frames**
- PRECEDING vs FOLLOWING
- UNBOUNDED vs OFFSET

**Frame Exclusions**
- CURRENT ROW vs TIES vs NO OTHERS (vs GROUP)

note: GROUPS is only implemented for some specific SQL flavours including PostgreSQL and SQLite

To explain and understand these different components more deeply - let’s create a very simple dataset to illustrate the different results.

### Example Dataset

The base dataset contains only a single column called val in the following frame_example table - however we will also be adding 2 columns for the ROW_NUMBER and DENSE_RANK window calculations. You will see why we do this soon!

```sql
CREATE TEMP TABLE frame_example AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
     1,
     1,
     2,
     6,
     9,
     9,
     20,
     20,
     25
 ]) AS val
)
SELECT
  val,
  ROW_NUMBER() OVER w AS _row_number,
  DENSE_RANK() OVER w AS _dense_rank
FROM input_data
WINDOW
  w AS (ORDER BY val);

-- inspect the dataset
SELECT * FROM frame_example
```

| val | _row_number | _dense_rank |
|-----|-------------|-------------|
| 1   | 1           | 1           |
| 1   | 2           | 1           |
| 2   | 3           | 2           |
| 6   | 4           | 3           |
| 9   | 5           | 4           |
| 9   | 6           | 4           |
| 20  | 7           | 5           |
| 20  | 8           | 5           |
| 25  | 9           | 6           |

For the following examples - we will implement the cumulative sum of the val column and compare the different combinations we can generate using the various window frame components to demonstrate the difference in the outputs!

### Default Cumulative Sum

Let’s revisit our basic cumulative sum example but this time using this very basic dataset.

If we were to implement a cumulative sum by simply using the SUM function with an ORDER BY without specifying the frame clause - we would be using the standard default window frame clause: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

Let’s inspect what this yields us for the cumulative sum and see if it checks out with what we expect!

```sql
CREATE TEMP TABLE frame_example AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
     1,
     1,
     2,
     6,
     9,
     9,
     20,
     20,
     25
 ]) AS val
)
SELECT
  val,
  ROW_NUMBER() OVER w AS _row_number,
  DENSE_RANK() OVER w AS _dense_rank
FROM input_data
WINDOW
  w AS (ORDER BY val);

SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    -- you can comment out the following line to make sure it is the default!
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cum_sum,
  _row_number,
  _dense_rank
FROM frame_example;
```

| val | cum_sum | _row_number | _dense_rank |
|-----|---------|-------------|-------------|
| 1   | 2       | 1           | 1           |
| 1   | 2       | 2           | 1           |
| 2   | 4       | 3           | 2           |
| 6   | 10      | 4           | 3           |
| 9   | 28      | 5           | 4           |
| 9   | 28      | 6           | 4           |
| 20  | 68      | 7           | 5           |
| 20  | 68      | 8           | 5           |
| 25  | 93      | 9           | 6           |

What do you notice about this output? Does it match up with what you expected when you thought about cumulative sums?

This is where we need to dig deeper to understand the differences between the window frame modes!

### Window Frame Modes

As we mentioned above - there are 3 window frame modes which will modulate the output differently:

1. RANGE
2. ROWS
3. GROUPS *Note: this is not always implemented for all SQL flavours!*

We will first focus on the difference between the RANGE and ROWS as these are commonly misunderstood by SQL developers and analytics practitioners leading to many issues with their ordered aggregate window function calculations!

The GROUPS mode is available in PostgreSQL and SQLite at this time of writing in February 2021 but not all other SQL flavours have the same functionality so always check the base documentation for whatever type of SQL you are using, always!

Let’s compare what happens when we use the RANGE, ROWS and GROUPS for the frame mode with our previous basic dataset and also keeping our default start and end frames of UNBOUNDED PRECEDING and CURRENT ROW:

```sql
CREATE TEMP TABLE frame_example AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
     1,
     1,
     2,
     6,
     9,
     9,
     20,
     20,
     25
 ]) AS val
)
SELECT
  val,
  ROW_NUMBER() OVER w AS _row_number,
  DENSE_RANK() OVER w AS _dense_rank
FROM input_data
WINDOW
  w AS (ORDER BY val);

SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS _range,
  SUM(val) OVER (
    ORDER BY val
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS _rows,
  -- GROUPS is not supported in BigQuery
  -- SUM(val) OVER (
  --   ORDER BY val
  --   GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  -- ) AS _groups,
  _row_number,
  _dense_rank
FROM frame_example;
```

| val | _range | _rows | _groups | _row_number | _dense_rank |
|-----|--------|-------|---------|-------------|-------------|
| 1   | 2      | 1     | 2       | 1           | 1           |
| 1   | 2      | 2     | 2       | 2           | 1           |
| 2   | 4      | 4     | 4       | 3           | 2           |
| 6   | 10     | 10    | 10      | 4           | 3           |
| 9   | 28     | 19    | 28      | 5           | 4           |
| 9   | 28     | 28    | 28      | 6           | 4           |
| 20  | 68     | 48    | 68      | 7           | 5           |
| 20  | 68     | 68    | 68      | 8           | 5           |
| 25  | 93     | 93    | 93      | 9           | 6           |

### Unbounded Preceding and Current Row

So far we have only been dealing with a single type of start and end frame - UNBOUNDED PRECEDING and CURRENT ROW

Having a start frame of UNBOUNDED PRECEDING means to include records from the start of the window frame.

An end frame of CURRENT ROW means to only include up to the current record for the window function calculation.

This particular set of start and end frame inputs also modulate with the window frame mode as we’ve seen in the example above.

In the following diagram - I’ve illustrated how the cumulative sum is calculated in reference to the window frame specification:

<img width="543" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/a8d1ce5c-3a3e-431c-9dc3-7b732904a571">

The most notable feature is how the RANGE and GROUPS mode will use the same cumulative sum value for records which have equal values - whilst the ROWS will perform the cumulative sum in a way which we intuitively think about it as having the row values added to eachother one by one.

However deciding when we want to actually use ROWS vs RANGE comes down to answer the question - what do you want your output to look like?

Do you want the same sum values for duplicate row values or would you like to have each individual row with its own separate cumulative metric? The choice is yours!

Let’s continue with the comparison between offset and unbounded next.

### Offset and Unbounded

The differences between these 2 terms are simple in that any start or end frame with an offset will only be limited to the specified offset size within the window frame, as opposed to the full window frame unbounded.

However - the practical differences are not that straightforward as the offset values and size of the window frames will be determined by the window frame mode that is chosen!

Let’s compare the 3 separate modes and their offset values below:

<img width="594" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/cf0cfab3-2c93-4566-8050-9c73cc592432">

As you might have noticed - they differ quite greatly across the different window frame modes!

The ROWS mode gives us the most intuitive offset values as we simply increment the offset value by 1 as we move before and after relative to the current row.

The RANGE mode uses the actual ordered values to generate the offset values - this has a deep implication, especially when we are dealing with window functions that use date fields in the ORDER BY within the OVER clause. We will dig into this deeper when we return to our Bitcoin data soon!

The GROUPS mode is sort of like a combination between the ROWS and the RANGE where we would essentially compare not just the actual values, but also the position of their groups relative to the current row.

The following are examples of how we can set the start and end frames using different offset values for our 3 window frame modes:

#### Range Mode
<img width="593" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/5f0f175e-8ec8-4166-8cce-629625aa6886">


#### Rows Mode
<img width="606" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/dcf7e8ad-fc87-423e-a4db-18e204e302db">


#### Groups Mode
<img width="594" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/b01aeec0-8b7f-420b-ab35-e51fa74621be">


### Frame Exclusion

Now that we’ve investigated how to define the window frames using the different window frame modes and also the offset and unbounded preceding and following options - let’s now move onto excluding our values within the window frames.

There are 4 options for excluding records within the window frame:

- CURRENT ROW
- TIES
- GROUP
- NO OTHERS (default)

Note that you must have an explicit window frame clause inside the OVER clause otherwise the frame exclusion will cause an error!

```sql
-- EXCLUDE does not exist in BigQuery. This is for PostgreSQL

SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    -- try uncomenting the line below!
    -- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE CURRENT ROW
  ) AS _excl_current_row
FROM frame_example;
```
Keeping the window frame settings as RANGE UNBOUNDED PRECEDING AND CURRENT ROW - let’s compare how these different exclusion options impact our cumulative sum calculation.

Instead of me explaining to you how they differ - it might be a better idea to see for yourself and try to explain the differences in your own words (this seems like a recurring theme throughout our tutorials!):


```sql
-- EXCLUDE does not exist in BigQuery. This is for PostgreSQL

SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS _default,
  SUM(val) OVER (
    ORDER BY val
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE CURRENT ROW
  ) AS _excl_current_row,
  SUM(val) OVER (
    ORDER BY val
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE TIES
  ) AS _excl_ties,
  SUM(val) OVER (
    ORDER BY val
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE GROUP
  ) AS _excl_groups,
  _row_number,
  _dense_rank
FROM frame_example;

```

| val | _default | _excl_current_row | _excl_ties | _excl_groups | _row_number | _dense_rank |
|-----|----------|-------------------|------------|--------------|-------------|-------------|
| 1   | 2        | 1                 | 1          |              | 1           | 1           |
| 1   | 2        | 1                 | 1          |              | 2           | 1           |
| 2   | 4        | 2                 | 4          | 2            | 3           | 2           |
| 6   | 10       | 4                 | 10         | 4            | 4           | 3           |
| 9   | 28       | 19                | 19         | 10           | 5           | 4           |
| 9   | 28       | 19                | 19         | 10           | 6           | 4           |
| 20  | 68       | 48                | 48         | 28           | 7           | 5           |
| 20  | 68       | 48                | 48         | 28           | 8           | 5           |
| 25  | 93       | 68                | 93         | 68           | 9           | 6           |

Let’s also inspect how these will differ when we use ROWS UNBOUNDED PRECEDING AND CURRENT ROW also:

```sql
-- EXCLUDE does not exist in BigQuery. This is for PostgreSQL

SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS _default,
  SUM(val) OVER (
    ORDER BY val
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE CURRENT ROW
  ) AS _excl_current_row,
  SUM(val) OVER (
    ORDER BY val
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE TIES
  ) AS _excl_ties,
  SUM(val) OVER (
    ORDER BY val
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE GROUP
  ) AS _excl_groups,
  _row_number,
  _dense_rank
FROM frame_example;
```
| val | _default | _excl_current_row | _excl_ties | _excl_groups | _row_number | _dense_rank |
|-----|----------|-------------------|------------|--------------|-------------|-------------|
| 1   | 1        |                   | 1          |              | 1           | 1           |
| 1   | 2        | 1                 | 1          |              | 2           | 1           |
| 2   | 4        | 2                 | 4          | 2            | 3           | 2           |
| 6   | 10       | 4                 | 10         | 4            | 4           | 3           |
| 9   | 19       | 10                | 19         | 10           | 5           | 4           |
| 9   | 28       | 19                | 19         | 10           | 6           | 4           |
| 20  | 48       | 28                | 48         | 28           | 7           | 5           |
| 20  | 68       | 48                | 48         | 28           | 8           | 5           |
| 25  | 93       | 68                | 93         | 68           | 9           | 6           |

Let’s also use the GROUPS frame mode just for completeness, please remember that this is not always implemented in all SQL flavours so always check the documentation for your chosen SQL implementation!

```sql
-- EXCLUDE does not exist in BigQuery. This is for PostgreSQL
SELECT
  val,
  SUM(val) OVER (
    ORDER BY val
    GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS _default,
  SUM(val) OVER (
    ORDER BY val
    GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE CURRENT ROW
  ) AS _excl_current_row,
  SUM(val) OVER (
    ORDER BY val
    GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE TIES
  ) AS _excl_ties,
  SUM(val) OVER (
    ORDER BY val
    GROUPS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    EXCLUDE GROUP
  ) AS _excl_groups,
  _row_number,
  _dense_rank
FROM frame_example;
```
| val | _default | _excl_current_row | _excl_ties | _excl_groups | _row_number | _dense_rank |
|-----|----------|-------------------|------------|--------------|-------------|-------------|
| 1   | 2        | 1                 | 1          |              | 1           | 1           |
| 1   | 2        | 1                 | 1          |              | 2           | 1           |
| 2   | 4        | 2                 | 4          | 2            | 3           | 2           |
| 6   | 10       | 4                 | 10         | 4            | 4           | 3           |
| 9   | 28       | 19                | 19         | 10           | 5           | 4           |
| 9   | 28       | 19                | 19         | 10           | 6           | 4           |
| 20  | 68       | 48                | 48         | 28           | 7           | 5           |
| 20  | 68       | 48                | 48         | 28           | 8           | 5           |
| 25  | 93       | 68                | 93         | 68           | 9           | 6           |

# Window Function Examples
Let’s now return to our Bitcoin data example to demonstrate how we can use our knowledge of these window frame clauses to help us customise our cumulative metrics.

Instead of discussing the theory behind the functions - we’ll go straight into implementation mode. Prepare yourself for some past paced SQL in the following sections!

In case you are returning to this tutorial and need to regenerate our fixed bitcoin dataset - you can run the snippet of code hidden in the block below:
```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

SELECT *
FROM updated_daily_btc
WHERE market_date BETWEEN CAST('2020-10-08' AS DATE) AND CAST('2020-10-15' AS DATE)
ORDER BY market_date;
```

## Weekly Volume Comparison

The following are a set of questions that we’d like to answer using SQL and window functions:

1.  What is the average daily volume of Bitcoin for the last 7 days?
2. Create a 1/0 flag if a specific day is higher than the last 7 days volume average
3. What is the percentage of weeks (starting on a Monday) where there are 4 or more days with increased volume?
4. How many high volume weeks are there broken down by year for the weeks with 5-7 days above the 7 day volume average excluding 2021?

### Average Volumes

1. What is the average daily volume of Bitcoin for the last 7 days?

First we need to calculate the weekly volume for the previous 7 days - we can use our RANGE mode in combination with the actual day range values to accomplish this.

Instead of just using integers like we were using for our previous simple dataset - we are able to use day ranges for our window function calculation also.

Following this we will generate a flag for each row comparing the actual volume to the average calculated volume.

The query below will cover points 1 and 2:

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

WITH window_calculations AS (
  SELECT
    market_date,
    volume,
    AVG(volume) OVER (
      ORDER BY UNIX_DATE(market_date)
      RANGE BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) AS past_weekly_avg_volume
  FROM updated_daily_btc
)

SELECT 
  market_date,
  volume,
  CASE
    WHEN volume > past_weekly_avg_volume THEN 1
    ELSE 0
    END AS volume_flag
FROM window_calculations
ORDER BY market_date DESC
LIMIT 10;
```
| market_date | volume       | volume_flag |
|-------------|--------------|-------------|
| 2021-02-24  | 88,364,793,856 | 1           |
| 2021-02-23  | 106,102,492,824 | 1           |
| 2021-02-22  | 92,052,420,332 | 1           |
| 2021-02-21  | 51,897,585,191 | 0           |
| 2021-02-20  | 68,145,460,026 | 0           |
| 2021-02-19  | 63,495,496,918 | 0           |
| 2021-02-18  | 52,054,723,579 | 0           |
| 2021-02-17  | 80,820,545,404 | 1           |
| 2021-02-16  | 77,049,582,886 | 0           |
| 2021-02-15  | 77,069,903,166 | 0           |

### High Volume Weeks

Next - let’s tackle the week percentage calculations for parts 3 and 4 in a single go.

We are yet to dive deeper into date calculations - but you can simply use DATE_TRUNC('week', <date-column>)::DATE to generate the prior Monday’s date.

If we then aggregate our dataset - we can then generate the distributions using the same SUM(COUNT*) OVER () trick we’ve used before to calculate proportions previously:

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

WITH window_calculations AS (
  SELECT
    market_date,
    volume,
    AVG(volume) OVER (
      ORDER BY UNIX_DATE(market_date)
      RANGE BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) AS past_weekly_avg_volume
  FROM updated_daily_btc
),

-- GENERATE THE DATE
date_calculations AS (
  SELECT
    market_date,
    DATE_TRUNC(CAST(market_date AS DATE), WEEK) AS start_of_week,
    volume,
    CASE
      WHEN volume > past_weekly_avg_volume THEN 1
      ELSE 0
      END AS volume_flag
  FROM window_calculations
),

-- aggregate the metrics and calculate a percentage
aggregated_weeks AS (
  SELECT
    start_of_week,
    SUM(volume_flag) AS weekly_high_volume_days
  FROM date_calculations
  GROUP BY 1
)

-- finally calculate the percentage
SELECT
  weekly_high_volume_days,
  ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS percentage_of_weeks
FROM aggregated_weeks
GROUP BY 1
ORDER BY 1;

```

| weekly_high_volume_days | percentage_of_weeks |
|-------------------------|---------------------|
| 0                       | 6.23                |
| 1                       | 13.65               |
| 2                       | 20.47               |
| 3                       | 20.47               |
| 4                       | 18.99               |
| 5                       | 11.87               |
| 6                       | 6.23                |
| 7                       | 2.08                |


### Breakdown By Year
When do these higher volume weeks of 5-7 days occur by year prior to 2021?

Can you provide some additional insight and/or reasons why this might be the case?

```sql
CREATE TEMP TABLE updated_daily_btc AS
SELECT
  market_date,
  COALESCE(
    open_price,
    LAG(open_price, 1) OVER w,
    LAG(open_price, 2) OVER w
  ) AS open_price,
  COALESCE(
    high_price,
    LAG(high_price, 1) OVER w,
    LAG(high_price, 2) OVER w
  ) AS high_price,
  COALESCE(
    low_price,
    LAG(low_price, 1) OVER w,
    LAG(low_price, 2) OVER w
  ) AS low_price,
  COALESCE(
    close_price,
    LAG(close_price, 1) OVER w,
    LAG(close_price, 2) OVER w
  ) AS close_price,
  COALESCE(
    adjusted_close_price,
    LAG(adjusted_close_price, 1) OVER w,
    LAG(adjusted_close_price, 2) OVER w
  ) AS adjusted_close_price,
  COALESCE(
    volume,
    LAG(volume, 1) OVER w,
    LAG(volume, 2) OVER w
  ) AS volume
FROM `younglapalma.trading.daily_btc`
WINDOW w AS (ORDER BY market_date);

WITH window_calculations AS (
  SELECT
    market_date,
    volume,
    AVG(volume) OVER (
      ORDER BY UNIX_DATE(market_date)
      RANGE BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) AS past_weekly_avg_volume
  FROM updated_daily_btc
),

-- GENERATE THE DATE
date_calculations AS (
  SELECT
    market_date,
    DATE_TRUNC(CAST(market_date AS DATE), WEEK) AS start_of_week,
    volume,
    CASE
      WHEN volume > past_weekly_avg_volume THEN 1
      ELSE 0
      END AS volume_flag
  FROM window_calculations
),

-- aggregate the metrics and calculate a percentage
aggregated_weeks AS (
  SELECT
    start_of_week,
    SUM(volume_flag) AS weekly_high_volume_days
  FROM date_calculations
  GROUP BY 1
)

-- finally calculate the percentage
SELECT
  EXTRACT(YEAR FROM start_of_week) AS market_year,
  COUNT(*) AS high_volume_weeks,
  ROUND(100 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) AS percentage_of_weeks
FROM aggregated_weeks
WHERE (weekly_high_volume_days >=5) AND (start_of_week < CAST('2021-01-01' AS DATE))
GROUP BY 1
ORDER BY 1;
```
| market_year | high_volume_weeks | percentage_of_total |
|-------------|-------------------|---------------------|
| 2014        | 2                 | 2.99                |
| 2015        | 3                 | 4.48                |
| 2016        | 13                | 19.40               |
| 2017        | 17                | 25.37               |
| 2018        | 8                 | 11.94               |
| 2019        | 11                | 16.42               |
| 2020        | 13                | 19.40               |


## Moving Averages

For the next exercise - we will be calculating metrics not just a single window function but multiple in the same query.

For the following time windows: 14, 28, 60, 150 days - calculate the following metrics for the close_price column:

1. Moving average
2. Moving standard deviation
3. The maximum and minimum values

Additionally round all metrics to to the nearest whole number.

```SQL
SELECT
  market_date,
  close_price,
  -- averages
  ROUND(AVG(close_price) OVER w_14) AS avg_14,
  ROUND(AVG(close_price) OVER w_28) AS avg_28,
  ROUND(AVG(close_price) OVER w_60) AS avg_60,
  ROUND(AVG(close_price) OVER w_150) AS avg_150,
  -- standard deviation
  ROUND(STDDEV(close_price) OVER w_14) AS std_14,
  ROUND(STDDEV(close_price) OVER w_28) AS std_28,
  ROUND(STDDEV(close_price) OVER w_60) AS std_60,
  ROUND(STDDEV(close_price) OVER w_150) AS std_150,
  -- max
  ROUND(MAX(close_price) OVER w_14) AS max_14,
  ROUND(MAX(close_price) OVER w_28) AS max_28,
  ROUND(MAX(close_price) OVER w_60) AS max_60,
  ROUND(MAX(close_price) OVER w_150) AS max_150,
  -- min
  ROUND(MIN(close_price) OVER w_14) AS min_14,
  ROUND(MIN(close_price) OVER w_28) AS min_28,
  ROUND(MIN(close_price) OVER w_60) AS min_60,
  ROUND(MIN(close_price) OVER w_150) AS min_150
FROM updated_daily_btc
WINDOW
  w_14 AS (ORDER BY MARKET_DATE RANGE BETWEEN '14 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_28 AS (ORDER BY MARKET_DATE RANGE BETWEEN '28 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_60 AS (ORDER BY MARKET_DATE RANGE BETWEEN '60 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_150 AS (ORDER BY MARKET_DATE RANGE BETWEEN '150 DAYS' PRECEDING AND '1 DAY' PRECEDING)
ORDER BY market_date DESC
LIMIT 10;
```

### Statistical Analysis

For the next exercise - we wish to create an indicator column with value 1 whenever the current day’s close price falls outside the moving average plus or minus 2 moving standard deviations. This is also known as a basic confidence interval and we use this extensively throughout the data world!

This is a basic statistical proxy to simulate a very rudimentary outlier detection - the basic rule of thumb the range between 2 standard deviations from the mean is roughly where 99% of all values should lie based off the properties of the humble normal distribution.

Note that we need to make assumptions that our data is roughly normally distributed, which is not always the case! But that’s a story for another day - for now we can just use these methods naively!

Let’s keep just the avg and std columns from our previous query inside a base CTE before we start performing the calculations flags:

```SQL
WITH base_data AS (
  SELECT
  market_date,
  ROUND(close_price, 2) AS close_price,
  ROUND(AVG(close_price) OVER w_14) AS avg_14,
  ROUND(AVG(close_price) OVER w_28) AS avg_28,
  ROUND(AVG(close_price) OVER w_60) AS avg_60,
  ROUND(AVG(close_price) OVER w_150) AS avg_150,
  ROUND(STDDEV(close_price) OVER w_14) AS std_14,
  ROUND(STDDEV(close_price) OVER w_28) AS std_28,
  ROUND(STDDEV(close_price) OVER w_60) AS std_60,
  ROUND(STDDEV(close_price) OVER w_150) AS std_150
FROM updated_daily_btc
WINDOW
  w_14 AS (ORDER BY MARKET_DATE RANGE BETWEEN '14 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_28 AS (ORDER BY MARKET_DATE RANGE BETWEEN '28 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_60 AS (ORDER BY MARKET_DATE RANGE BETWEEN '60 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_150 AS (ORDER BY MARKET_DATE RANGE BETWEEN '150 DAYS' PRECEDING AND '1 DAY' PRECEDING)
)
SELECT
  market_date,
  close_price,
  CASE
    WHEN close_price BETWEEN
      (avg_14 - 2 * std_14) AND (avg_14 + 2 * std_14)
      THEN 0
    ELSE 1
    END AS outlier_14,
  CASE
    WHEN close_price BETWEEN
      (avg_28 - 2 * std_28) AND (avg_28 + 2 * std_28)
      THEN 0
    ELSE 1
    END AS outlier_28,
  CASE
    WHEN close_price BETWEEN
      (avg_60 - 2 * std_60) AND (avg_60 + 2 * std_60)
      THEN 0
    ELSE 1
    END AS outlier_60,
  CASE
    WHEN close_price BETWEEN
      (avg_150 - 2 * std_150) AND (avg_150 + 2 * std_150)
      THEN 0
    ELSE 1
    END AS outlier_150
FROM base_data
ORDER BY market_date DESC
LIMIT 10;
```

| market_date | close_price | outlier_14 | outlier_28 | outlier_60 | outlier_150 |
|-------------|-------------|------------|------------|------------|-------------|
| 2021-02-24  | 50460.23    | 0          | 0          | 0          | 1           |
| 2021-02-23  | 48824.43    | 0          | 0          | 0          | 0           |
| 2021-02-22  | 54207.32    | 0          | 0          | 1          | 1           |
| 2021-02-21  | 57539.95    | 1          | 0          | 1          | 1           |
| 2021-02-20  | 56099.52    | 0          | 0          | 1          | 1           |
| 2021-02-19  | 55888.13    | 1          | 1          | 1          | 1           |
| 2021-02-18  | 51679.80    | 0          | 0          | 1          | 1           |
| 2021-02-17  | 52149.01    | 0          | 1          | 1          | 1           |
| 2021-02-16  | 49199.87    | 0          | 0          | 1          | 1           |
| 2021-02-15  | 47945.06    | 0          | 0          | 1          | 1           |


What do you notice about the last 10 rows of data? Does it seem odd that there are 1 values for 60 and 150 day outliers for the majority of the rows or is it expected? Check the data visualisation of close_price and market_date to confirm this!

### Weighted Moving Averages

In the previous example we calculated a simple moving average with the last 7, 28, 60 and 150 days. But what were we actually calculating? Let’s take a look at the maths behind it so we can build our intuition for these metrics.

<img width="370" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/ad925907-129f-4154-9e2f-039e88036710">

Can you see any pitfalls in using this standard moving average metric for something as volatile as Bitcoin prices where prices 1 month ago can be drastically different to the current price?

A weighted average is similar to the simple moving average - however we are able to apply a “weight” or a multiplying factor for each specific days worth of data.

By applying a larger weight to recent data we are putting more weight or focus into our recent data - as opposed and increasing it’s contribution towards the final metric - this is in stark contrast to the simple moving average where we would naively use each past data points equally.

The simple moving average metric is similar to saying that the price 150 days ago, has the same impact on the moving average metric as the price from yesterday!

Using a weighted approach, we can apply as customised weights as we like to our data points. In the following examples, we will iterate through a few variations where we might be able to use different applications of window functions and other SQL techniques we’ve learnt so far!

<img width="517" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/dddf3ea4-e557-4ef7-be44-4081ba45cded">

Now let’s make things a bit more interesting for our example!

Imagine that we were tasked by a demanding Cryptocurrency trading client who required custom weights to be applied using the following combination of simple moving averages:

| Time Period | Weight Factor |
|-------------|---------------|
| 1-14 days   | 0.5           |
| 15-28 days  | 0.3           |
| 29-60 days  | 0.15          |
| 61-150 days | 0.05          |


If we split up our time periods into separate window frames using various OFFSET values like we’ve seen in the previous sections above - we can use different offsets to calculate each individual window frame average separately, then we can recombine them by multiplying our given weight values:

```SQL
WITH base_data AS (
  SELECT
    market_date,
    ROUND(close_price, 2) AS close_price,
    ROUND(AVG(close_price) OVER w_1_to_14) AS avg_1_to_14,
    ROUND(AVG(close_price) OVER w_15_28) AS avg_15_28,
    ROUND(AVG(close_price) OVER w_29_60) AS avg_29_60,
    ROUND(AVG(close_price) OVER w_61_150) AS avg_61_150
FROM updated_daily_btc
WINDOW
  w_1_to_14 AS (ORDER BY MARKET_DATE RANGE BETWEEN
    '14 DAYS' PRECEDING AND '1 DAY' PRECEDING),
  w_15_28 AS (ORDER BY MARKET_DATE RANGE BETWEEN
    '28 DAYS' PRECEDING AND '15 DAY' PRECEDING),
  w_29_60 AS (ORDER BY MARKET_DATE RANGE BETWEEN
    '60 DAYS' PRECEDING AND '29 DAY' PRECEDING),
  w_61_150 AS (ORDER BY MARKET_DATE RANGE BETWEEN
  '150 DAYS' PRECEDING AND '61 DAY' PRECEDING)
)
SELECT
  market_date,
  close_price,
  0.5 * avg_1_to_14 + 0.3 * avg_15_28 + 0.15 * avg_29_60 + 0.05 * avg_61_150 AS custom_moving_avg
FROM base_data
ORDER BY market_date DESC;
```

## Exponential Weighted Average

This is actually an extension which lies outside of the window functions family - however since we are on the topic of moving averages, it just seems like a shame if we didn’t cover this!

The EWMA or exponential weighted moving average uses the past values of the weighted average to calculate the current value - it is super popular in data analysis and it is a handy SQL trick to hide up your sleeve!

The most important concept for the EWMA which makes it different from the previous simple moving average and the weighted moving average is simply a “smoothing” term - often referred to as a lambda value or λ
 in mathematical notation.

Let’s say for example - we wanted to calculate a 14 day EWMA. The recommended guideline for a starting value of lambda is usually 2 / (number of days + 1) so for our example - our λ
 lambda value will be 2 / (14 + 1) = 0.13

Now the formula for our EWMA looks a bit like this:

<img width="446" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/8aaf49d0-2b8f-4e0e-9c15-f6622c87ef7e">

Remember that the simple moving average does not incorporate the current date’s value into the calculation so day 15 is the first day where we would actually have enough data to calculate a 14 day simple moving average!

So clearly to say this is a pretty complex calculation - you can imagine how complex this looks once we start incorporating lots of date values!

Luckily for us we have an ultra ace up our sleeve to implement this in SQL - enter the Recursive CTE!

### Recursive CTE

In the past we’ve seen CTEs being used to help us input raw data, split up our multipart queries and improve the readability of our code. In this section - we will demonstrate how we can apply them to perform such complicated recursive calculations as our EWMA example. Buckle up because this section will challenge you!

The first concept we need to understand about recursion or recursive function calculations is the “base” which we will use to initiate our manipulations.

In our 14 day EWMA example - we will initiate our recursive analysis by calculating the average of the first 14 days worth of data from our updated BTC dataset:

Before we dive into the recursive CTE implementation - let’s compute this using our basic window functions to see what value we should expect and create a temp base_table which we will use downstream for our EWMA calculation:

```sql
DROP TABLE IF EXISTS base_table;
CREATE TEMP TABLE base_table AS
SELECT
    market_date,
    ROUND(close_price, 2) AS close_price,
    ROUND(AVG(close_price) OVER w_1_to_14) AS sma_14,
    ROW_NUMBER() OVER w_1_to_14 AS _row_number
FROM updated_daily_btc
WINDOW
  w_1_to_14 AS (ORDER BY MARKET_DATE RANGE BETWEEN
    '14 DAYS' PRECEDING AND '1 DAY' PRECEDING);

-- sample the first 20 rows ordered by date
SELECT *
FROM base_table
ORDER BY market_date;
LIMIT 20
```

| market_date | close_price | sma_14 | _row_number |
|-------------|-------------|--------|-------------|
| 2014-09-17  | 457.33      |        | 1           |
| 2014-09-18  | 424.44      | 457    | 2           |
| 2014-09-19  | 394.80      | 441    | 3           |
| 2014-09-20  | 408.90      | 426    | 4           |
| 2014-09-21  | 398.82      | 421    | 5           |
| 2014-09-22  | 402.15      | 417    | 6           |
| 2014-09-23  | 435.79      | 414    | 7           |
| 2014-09-24  | 423.20      | 417    | 8           |
| 2014-09-25  | 411.57      | 418    | 9           |
| 2014-09-26  | 404.42      | 417    | 10          |
| 2014-09-27  | 399.52      | 416    | 11          |
| 2014-09-28  | 377.18      | 415    | 12          |
| 2014-09-29  | 375.47      | 412    | 13          |
| 2014-09-30  | 386.94      | 409    | 14          |
| 2014-10-01  | 383.61      | 407    | 15          |
| 2014-10-02  | 375.07      | 402    | 16          |
| 2014-10-03  | 359.51      | 398    | 17          |
| 2014-10-04  | 328.87      | 396    | 18          |
| 2014-10-05  | 320.51      | 390    | 19          |
| 2014-10-06  | 330.08      | 385    | 20          |

So it looks we will actually still generate calculations for the first 14 days of our dataset due to the window function default behaviour - however for our EWMA calculation, we will not be generating values for these records as they will not have a commplete SMA value - so we will unforunately be discarding these rows for our EWMA calculations.

For all recursive CTE SQL implementations we need the following elements:

1. An initial query which generates the 1st row of our final output from our base_table
2. A UNION to a second query which will refer to the recursive CTE output_table and the original base_table

The magic part of the CTE occurs in our example when we join the base_table to our output_table on the _row_number field in the second query - see if you can identify exactly where and why this is so important as we “shift” our output to use the ewma_14 field in our second query calculation!

It is super important that all of our columns match up so we take our CTE to the next level by defining the column names we require for output_table and also by creating another temporary table called ewma_output

```sql
DROP TABLE IF EXISTS ewma_output;
CREATE TEMP TABLE ewma_output AS
WITH RECURSIVE output_table
(market_date, close_price, sma_14, ewma_14, _row_number)
AS (

-- The 1st query: we set initial output row using _row_number = 15
  SELECT
    market_date,
    close_price,
    sma_14,
    sma_14 AS ewma_14,
    _row_number
  FROM base_table
  WHERE _row_number = 15

  UNION ALL

  -- The 2nd query: we need to "shift" our output by 1 row
  SELECT
    base_table.market_date,
    base_table.close_price,
    base_table.sma_14,
    -- Let's round out ewma_14 record to 2 decimal places
    ROUND(
      0.13 * base_table.sma_14 + (1 - 0.13) * output_table.ewma_14,
      2
    ) AS ewma_14,
    base_table._row_number
  FROM output_table
  INNER JOIN base_table
    ON output_table._row_number + 1 = base_table._row_number
    AND base_table._row_number > 15
)
SELECT * FROM output_table;

-- Finally show the outputs we generated
SELECT * FROM ewma_output LIMIT 10;
```

| market_date | close_price | sma_14 | ewma_14 | _row_number |
|-------------|-------------|--------|---------|-------------|
| 2014-10-01  | 383.61      | 407    | 407     | 15          |
| 2014-10-02  | 375.07      | 402    | 406.35  | 16          |
| 2014-10-03  | 359.51      | 398    | 405.26  | 17          |
| 2014-10-04  | 328.87      | 396    | 404.06  | 18          |
| 2014-10-05  | 320.51      | 390    | 402.23  | 19          |
| 2014-10-06  | 330.08      | 385    | 399.99  | 20          |
| 2014-10-07  | 336.19      | 379    | 397.26  | 21          |
| 2014-10-08  | 352.94      | 372    | 393.98  | 22          |
| 2014-10-09  | 365.03      | 367    | 390.47  | 23          |
| 2014-10-10  | 361.56      | 364    | 387.03  | 24          |

Let’s take this final analysis even further and we will pivot our data into a long format so we can easily visualise it within the SQLPad GUI.