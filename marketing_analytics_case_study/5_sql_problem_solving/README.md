# SQL Data Problem Solving

Now that we’ve combined all of our different datasets together into a single base table which we can use for our insights - let’s revise our email template and that base table that we now use for many of our downstream components.

## SQL Base Table

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

SELECT * FROM complete_joint_dataset limit 10;
```

| customer_id | film_id | title           | rental_date         | category_name |
|-------------|---------|-----------------|---------------------|---------------|
| 130         | 80      | BLANKET BEVERLY | 2005-05-24 22:53:30 | Family        |
| 459         | 333     | FREAKY POCUS    | 2005-05-24 22:54:33 | Music         |
| 408         | 373     | GRADUATE LORD   | 2005-05-24 23:03:39 | Children      |
| 333         | 535     | LOVE SUICIDES   | 2005-05-24 23:04:41 | Horror        |
| 222         | 450     | IDOLS SNATCHERS | 2005-05-24 23:05:21 | Children      |
| 549         | 613     | MYSTIC TRUMAN   | 2005-05-24 23:08:07 | Comedy        |
| 269         | 870     | SWARM GOLD      | 2005-05-24 23:11:53 | Horror        |
| 239         | 510     | LAWLESS VISION  | 2005-05-24 23:31:46 | Animation     |
| 126         | 565     | MATRIX SNOWMAN  | 2005-05-25 00:00:40 | Foreign       |
| 399         | 396     | HANGING DEEP    | 2005-05-25 00:02:21 | Drama         |

# Data Next Steps
We will use this base table as our starting point as we work towards the customer level insights and the film recommendations.

In this tutorial we will aim to cover those core calculated fields which we broke down in our first reverse engineering section of this case study.

Let’s also revisit some of these calculations we need to perform again to jog our memory:

## Core Calculated Fields

- `category_name`: The name of the top 2 ranking categories
- `rental_count`: How many total films have they watched in this category
- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer
- `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?
- `category_percentage`: What proportion of total films watched does this category make up?

We will need these calculated fields to help us arrive at the various interim table outputs to reach our final required outputs for this case study.

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                  | 2          | 16                  |
| 2           | 1                | Sports        | 5            | 3                  | 7          | 19                  |
| 2           | 2                | Classics      | 4            | 2                  | 11         | 15                  |
| 3           | 1                | Action        | 4            | 2                  | 14         | 15                  |
| 3           | 2                | Animation     | 3            | 1                  | 39         | 12                  |
| 4           | 1                | Horror        | 3            | 2                  | 22         | 14                  |
| 4           | 2                | Travel        | 2            | 1                  | 57         | 9                   |

We mentioned earlier in the Multiple Table Joining tutorial that we would need to keep all of the rental category counts for each customer - just like it’s shown in the sample output above - however we might run into issues if we only keep those top 2 ranking categories as we perform some of these calculations.

Let’s dig into this a little bit more in the next section.

## Potential Errors with Our Approach

When we look at these core calculated metrics - the final 3 metrics average_comparison, percentile and category_percentage are actually dependent on all of the category counts and not just the top 2 ranked categories.

We will definitely need those top 2 category_name and rental_count values for every customer - but how can we compute those 3 calculations if we only want to keep those top 2 values?

Let’s inspect this potential error a little bit deeper with a simple and clear illustrated example.

## Illustrated Problem Example

Let’s paint an imaginary scenario where we only have 3 customers in our entire database - we can do this using our existing data example and taking only customer_id values of 1, 2 and 3.

We can generate each customers’ aggregated rental_count values for every category_name value from our complete_joint_dataset temporary table also.

Let’s also sort the output by customer_id and show the rental_count from largest to smallest for each customer:

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
WHERE customer_id in (1, 2, 3)
GROUP BY
  customer_id,
  category_name
ORDER BY
  customer_id,
  rental_count DESC;
```

**Customer 1**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 1           | Classics     | 6            |
| 1           | Comedy       | 5            |
| 1           | Drama        | 4            |
| 1           | Sports       | 2            |
| 1           | Action       | 2            |
| 1           | Music        | 2            |
| 1           | Animation    | 2            |
| 1           | New          | 2            |
| 1           | Sci-Fi       | 2            |
| 1           | Travel       | 1            |
| 1           | Documentary  | 1            |
| 1           | Family       | 1            |
| 1           | Foreign      | 1            |
| 1           | Games        | 1            |

**Customer 2**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 2           | Sports       | 5            |
| 2           | Classics     | 4            |
| 2           | Action       | 3            |
| 2           | Animation    | 3            |
| 2           | Travel       | 2            |
| 2           | New          | 2            |
| 2           | Games        | 2            |
| 2           | Sci-Fi       | 1            |
| 2           | Family       | 1            |
| 2           | Music        | 1            |
| 2           | Documentary  | 1            |
| 2           | Children     | 1            |
| 2           | Foreign      | 1            |

**Customer 3**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 3           | Action       | 4            |
| 3           | Sci-Fi       | 3            |
| 3           | Animation    | 3            |
| 3           | Horror       | 2            |
| 3           | Sports       | 2            |
| 3           | Comedy       | 2            |
| 3           | New          | 2            |
| 3           | Music        | 2            |
| 3           | Games        | 2            |
| 3           | Classics     | 1            |
| 3           | Family       | 1            |
| 3           | Drama        | 1            |
| 3           | Documentary  | 1            |

### Top 2 Category per Customer
Let’s now imagine that we just go full speed ahead and trim our dataset keeping only the top 2 categories for each customer - we would get the following results below.

Let’s just say we visually inspected our customer records because we will need to cover some window function magic before we can implement this ranking and filtering:

**Customer 1**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 1           | Classics     | 6            |
| 1           | Comedy       | 5            |

**Customer 2**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 2           | Sports       | 5            |
| 2           | Classics     | 4            |

**Customer 3**
| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 3           | Action       | 4            |
| 3           | Sci-Fi       | 3            |
| 3           | Animation    | 3            |

Customer 3 is an edge case where it just so happens that both Sci-Fi and Animation categories have a rental_count value of 3 - let’s dive into this a bit more.

### Dealing With Ties
We refer to this matching occurence as “ties” in regular conversation, usually you’ll hear things said in meetings and such like - “How are we going to deal with ties?”

There are multiple ways you can deal with these “equal” ranking row ties:

1. Sort the values further by an additional condition or criteria
2. Randomly select a single row
#### Do Not Select Randomly
In most cases - you do not want to randomly select single rows as this is **not reproducible** - meaning that often the behaviour will change each time you run the SQL query.

This is a really important concept when it comes to data science in practice as we will often need to prove that we can repeatedly generate the same data points, even if some script or process is ran at different times.

So that leaves us with only option 1 - we will need to figure out what we should be sorting by as an additional criterion for our example.

#### Additional Sorting Criteria
Often times when we think about these additional sorting or ranking conditions - we should aim to choose something which makes the most sense in terms of a business or customer perspective or something which is low cost and simple to execute.

For example - a super simple method to execute might be to just sort the category_name fields alphabetically - it might not be the “best” solution for our customer experience, but it might just work when we need to do something really quickly without needing to acquire additional data!

**Customer 3 categories sorted by rental count descending and alphabetical order**

| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 3           | Action       | 4            |
| 3           | Animation    | 3            |
| 3           | Sci-Fi       | 3            |

However, for our email example (and in general) we should consider how a customer might respond as a result of certain decisions like this.

One really common sorting method is to look at the most recent purchase or rental and sort by some recency metric based on when the last purchase was made.

We could propose that we investigate when the latest rental was completed for each category - if we had this rental_date value for each individual rental record, we could easily find the MAX(rental_date) value for each customer_id and category_name combination.

If we wanted to do this rental_date tracking - we must acquire this data point in our base temporary table for use in our queries.

However, luckily for us - it is not difficult to incorporate this additional rental_date field for our new calculations. You can check the query in the section below if you want to see how to do it:

The query below adds an additional filter for only `customer_id = 3` instead of `customer_id IN (1,2,3)` as our 3rd customer is the one with the ranking issue!

```sql
--firstly bring in rental_date as a field from the table joins
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

-- Finally perform group by aggregations on category_name and customer_id
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
-- note the different filter here!
WHERE customer_id = 3
GROUP BY
  customer_id,
  category_name
ORDER BY
  customer_id,
  rental_count DESC,
  latest_rental_date DESC;
```
**customer 3**
| customer_id | category_name | rental_count | latest_rental_date      |
|-------------|--------------|--------------|-------------------------|
| 3           | Action       | 4            | 2005-07-29 11:07:04     |
| 3           | Sci-Fi       | 3            | 2005-08-22 09:37:27     |
| 3           | Animation    | 3            | 2005-08-18 14:49:55     |
| 3           | Music        | 2            | 2005-08-23 07:10:14     |
| 3           | Comedy       | 2            | 2005-08-20 06:14:12     |
| 3           | Horror       | 2            | 2005-07-31 11:32:58     |
| 3           | Sports       | 2            | 2005-07-30 13:31:20     |
| 3           | New          | 2            | 2005-07-28 04:46:30     |
| 3           | Games        | 2            | 2005-07-27 04:54:42     |
| 3           | Classics     | 1            | 2005-08-01 14:19:48     |
| 3           | Family       | 1            | 2005-07-31 03:27:58     |
| 3           | Drama        | 1            | 2005-07-30 21:45:46     |
| 3           | Documentary  | 1            | 2005-06-19 08:34:53     |

Great - now we can see that Customer 3 most recent rental was from the Sci-Fi category - so we can use this additional criteria to sort the output and select the 2nd ranking category!

#### Additional Thoughts on Sorting and Testing

Keep in mind that this specific criteria we’ve used to sort is all theoretical - we can’t quite understand real customer preferences with the data we currently have!

Usually in these scenarios - we would perform some sort of split testing or other customer experiments to see what works. We might also send out some customer surveys and ask customers about what type of recommendations they would like to receive.

In general - these types of tests are often referred to as A/B tests or “champion vs challenger” tests and are super common in digital marketing, marketing analytics and even through to more complicated experimentation design with machine learning models!

Experimentation is a very broad topic and this simple example of us simply sorting our customer categories is nowhere near enough to cover even a fraction of the challenges and thinking behind this!

### Calculate Averages on Top 2 Categories

So now that we have all of our customers top 2 categories - let’s see what happens when we try to calculate the average on only just the top 2 categories dataset:

**Top 2 categories for all 3 customers**

| customer_id | category_name | rental_count |
|-------------|--------------|--------------|
| 1           | Classics     | 6            |
| 1           | Comedy       | 5            |
| 2           | Sports       | 5            |
| 2           | Classics     | 4            |
| 3           | Action       | 4            |
| 3           | Sci-Fi       | 3            |

To demonstrate what happens - let’s manually generate this dataset using our trusted CTE and VALUES method we’ve seen multiple time before - you can run the following SQL snippet in your SQLPad GUI to make sure you get the same output as above:

```SQL
CREATE TEMP TABLE top_2_category_rental_count AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
    STRUCT(1 AS customer_id, 'Classics' AS category_name, 6 AS rental_count),
    (1, 'Comedy', 5),
    (2, 'Sports', 5),
    (2, 'Classics', 4),
    (3, 'Action', 4),
    (3, 'Sci-Fi', 3)
 ])
)

SELECT * FROM input_data;

-- check output
SELECT * FROM top_2_category_rental_count;
```

It should seem pretty clear that we have already experienced some sort of “data loss” - all of the `Classics` category films that customer 3 has watched are no longer in this existing dataset, and the same can be said about all of the other non top 2 category rental_count values for all of the other categories in the `top_2_category_rental_count` dataset.

If we were to calculate the average of all customer’s `Classics` films - there is actually no record for customer 3 and now the average is heavily skewed to only customers who have `Classics` as one of their top 2 categories - this is a bit of a no no!

So let’s back up a bit here and compare our averages with the original aggregated `rental_count` values for all of our categories - we’ll use that initial `GROUP BY` query as a `CTE` so we can keep everything in a single SQL statement:

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

WITH aggregated_rental_count AS (
  SELECT
    customer_id,
    category_name,
    COUNT(*) AS rental_count
  FROM complete_joint_dataset
  WHERE customer_id in (1, 2, 3)
  GROUP BY
    customer_id,
    category_name
  /* -- we remove this order by because we don't need it here!
     ORDER BY
     customer_id,
     rental_count DESC
  */
)
SELECT
  category_name,
  -- round out large decimals to just 1 decimal point
  ROUND(AVG(rental_count), 1) AS avg_rental_count
FROM aggregated_rental_count
GROUP BY
  category_name
-- this will sort our output in alphabetical order
ORDER BY
  category_name;
```
| category_name | avg_rental_count |
|---------------|------------------|
| Action        | 3.0              |
| Animation     | 2.7              |
| Children      | 1.0              |
| Classics      | 3.7              |
| Comedy        | 3.5              |
| Documentary   | 1.0              |
| Drama         | 2.5              |
| Family        | 1.0              |
| Foreign       | 1.0              |
| Games         | 1.7              |
| Horror        | 2.0              |
| Music         | 1.7              |
| New           | 2.0              |
| Sci-Fi        | 2.0              |
| Sports        | 3.0              |
| Travel        | 1.5              |

Now let’s try calculating the same average rental count values for the top_2_category_rental_count dataset so we can compare them with the same values for the entire dataset.

```sql
CREATE TEMP TABLE top_2_category_rental_count AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
    STRUCT(1 AS customer_id, 'Classics' AS category_name, 6 AS rental_count),
    (1, 'Comedy', 5),
    (2, 'Sports', 5),
    (2, 'Classics', 4),
    (3, 'Action', 4),
    (3, 'Sci-Fi', 3)
 ])
)

SELECT * FROM input_data;

SELECT
  category_name,
  -- round out large decimals to just 1 decimal point
  ROUND(AVG(rental_count), 1) AS avg_rental_count
FROM top_2_category_rental_count
GROUP BY
  category_name
-- this will sort our output in alphabetical order
ORDER BY
  category_name;
```
| category_name | avg_rental_count |
|---------------|------------------|
| Action        | 4.0              |
| Classics      | 5.0              |
| Comedy        | 5.0              |
| Sci-Fi        | 3.0              |
| Sports        | 5.0              |

And when we compare just the categories which show up in the top_2_category values - we can see quite a large discrepancy in values!

| category_name | top_2_average | all_category_average |
|---------------|--------------|----------------------|
| Action        | 4.0          | 3.0                  |
| Classics      | 5.0          | 3.7                  |
| Comedy        | 5.0          | 3.5                  |
| Sci-Fi        | 3.0          | 2.0                  |
| Sports        | 5.0          | 3.0                  |

We can also use a few separate CTEs and a LEFT JOIN on category_name to generate this table above with only SQL code instead of manually comparing them.

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  rental.rental_date,
  category.name AS category_name
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE top_2_category_rental_count AS
WITH input_data AS (
 SELECT *
 FROM UNNEST([
    STRUCT(1 AS customer_id, 'Classics' AS category_name, 6 AS rental_count),
    (1, 'Comedy', 5),
    (2, 'Sports', 5),
    (2, 'Classics', 4),
    (3, 'Action', 4),
    (3, 'Sci-Fi', 3)
 ])
)

SELECT * FROM input_data;

WITH aggregated_rental_count AS (
  SELECT
    customer_id,
    category_name,
    COUNT(*) AS rental_count
  FROM complete_joint_dataset
  WHERE customer_id in (1, 2, 3)
  GROUP BY
    customer_id,
    category_name
  /* -- we remove this order by because we don't need it here!
     ORDER BY
     customer_id,
     rental_count DESC
  */
),
all_categories AS (SELECT
  category_name,
  -- round out large decimals to just 1 decimal point
  ROUND(AVG(rental_count), 1) AS all_category_average
FROM aggregated_rental_count
GROUP BY
  category_name
-- this will sort our output in alphabetical order
ORDER BY
  category_name),

top_2_categories AS (-- check output
SELECT 
  category_name,
  -- round out large decimals to just 1 decimal point
  ROUND(AVG(rental_count), 1) AS top_2_average
FROM top_2_category_rental_count
GROUP BY
  category_name
-- this will sort our output in alphabetical order
ORDER BY
  category_name)

-- final select statement for output
SELECT
  top_2_categories.category_name,
  top_2_categories.top_2_average,
  all_categories.all_category_average
FROM top_2_categories
LEFT JOIN all_categories
  ON top_2_categories.category_name = all_categories.category_name
ORDER BY
  top_2_categories.category_name;
```

## An Alternative?

So now that we know that there are going to be some serious differences when we look at those average values across each of the categories - we need to figure out an alternative solution instead of just picking the top 2 catgories and naively applying those aggregate functions.

This same issue will definitely impact the percentile value - how can we compare a specific customer’s ranking percentage compared to other customers if we don’t have the rental count for other customers for a specific category?

And finally the category_percentage calculation is actually only relative to a single customer’s rental behaviour - but how can we count the total rentals if we only have the top 2 categories?

Luckily there is a simple solution - we can just use the entire dataset for some of these calculations BEFORE we isolate the first 2 categories for our final output.

Let’s now try this whole process using the entire dataset instead of just customer_id of 1, 2 and 3!

# Data Aggregation on Whole Dataset

Just like we did before with our simple illustrated example - we will need to aggregate the rental_count values for each of our customers and category values, however this time - we will do our aggregations on the whole complete_joint_dataset temporary table we created earlier.

For the following few sections - we will split up each part of our various aggregations and calculations into separate temporary tables. See if you can figure out exactly why I’m doing this by the time we reach the end of this tutorial!

I’m not going to let you in on the exact reason now - but I will say that this breaking up into multiple tables will help us a lot when we need to finally compile an entire SQL script to generate our case study solution!

## Customer Rental Count

Let’s first aggregate that rental_count value first - however let’s also use the version of the joint dataset that also had the rental_date for each record too so we can generate an additional latest_rental_date field for use with our sorting and ordering step, just like in our simple illustrated example.

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

-- profile just customer_id = 1 values sorted by desc rental_count
SELECT *
FROM category_rental_counts
WHERE customer_id = 1
ORDER BY
  rental_count DESC;
```

| customer_id | category_name | rental_count | latest_rental_date   |
|-------------|--------------|--------------|----------------------|
| 1           | Classics     | 6            | 2005-08-19 09:55:16  |
| 1           | Comedy       | 5            | 2005-08-22 19:41:37  |
| 1           | Drama        | 4            | 2005-08-18 03:57:29  |
| 1           | Sci-Fi       | 2            | 2005-08-21 23:33:57  |
| 1           | Animation    | 2            | 2005-08-22 20:03:46  |
| 1           | Sports       | 2            | 2005-07-08 07:33:56  |
| 1           | Music        | 2            | 2005-07-09 16:38:01  |
| 1           | Action       | 2            | 2005-08-17 12:37:54  |
| 1           | New          | 2            | 2005-08-19 13:56:54  |
| 1           | Travel       | 1            | 2005-07-11 10:13:46  |
| 1           | Family       | 1            | 2005-08-02 18:01:38  |
| 1           | Documentary  | 1            | 2005-08-01 08:51:04  |
| 1           | Games        | 1            | 2005-07-08 03:17:05  |
| 1           | Foreign      | 1            | 2005-07-28 16:18:23  |

## Total Customer Rentals

In order to generate the category_percentage calculation we will need to get the total rentals per customer. This is a piece of cake using a simple GROUP BY and SUM

> `category_percentage`: What proportion of each customer’s total films watched does this count make?

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

-- show output for first 5 customer_id values
SELECT *
FROM customer_total_rentals
WHERE customer_id <= 5
ORDER BY customer_id;
```

| customer_id | total_rental_count |
|-------------|--------------------|
| 1           | 32                 |
| 2           | 27                 |
| 3           | 26                 |
| 4           | 22                 |
| 5           | 38                 |

## Average Category Rental Counts

Finally we can also use the AVG function with all of our category records for all customers to calculate the true average rental count for each category.

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;

-- output the entire table by desc avg_rental_count
SELECT *
FROM average_category_rental_counts
ORDER BY
  avg_rental_count DESC;
```

| category_name | avg_rental_count         |
|---------------|--------------------------|
| Animation     | 2.3320000000000000       |
| Sports        | 2.2716763005780347       |
| Family        | 2.1876247504990020       |
| Action        | 2.1803921568627451       |
| Documentary   | 2.1739130434782609       |
| Sci-Fi        | 2.1715976331360947       |
| Drama         | 2.1157684630738523       |
| Foreign       | 2.0953346855983773       |
| Games         | 2.0443037974683544       |
| New           | 2.0085470085470085       |
| Classics      | 2.0064102564102564       |
| Children      | 1.9605809128630705       |
| Comedy        | 1.9010101010101010       |
| Travel        | 1.8936651583710407       |
| Horror        | 1.8758314855875831       |
| Music         | 1.8568232662192394       |


## Update Table Values

Since it might seem a bit weird to be telling customers that they watched 1.346 more films than the average customer - let’s make an executive decision and just take the FLOOR value of the decimal to give our customers a bit of a feel-good boost that they watch more films (as opposed to if we rounded the number to the nearest integer!)

We can do this directly with that temp table we created by running an UPDATE command.

If you need to update multiple columns at one time, you can use a comma to separate each set of column and new value pairs.

We can also set a WHERE clause to only update specific rows that meet some condition we want.

Additionally - we can return the rows which were adjusted by specifying a RETURNING * at the end of the query.

Just to demonstrate all of this functionality - let’s create an exact copy of our average_category_rental_counts temporary table and call it testing_average_category_rental_counts and we will try to update a few things for films that start with the letter 'C'

Let’s try adding 100 to our average rental count value and also adding an extra string ‘Category’ to the end of our category_name field:

```sql
-- this is just copy and paste from the lesson (not adjusted to work in BigQuery)
-- first create a copy of average_category_rental_counts
DROP TABLE IF EXISTS testing_average_category_rental_counts;
CREATE TEMP TABLE testing_average_category_rental_counts AS
  TABLE average_category_rental_counts;

-- now update all the things!
UPDATE testing_average_category_rental_counts
SET
  avg_rental_count = avg_rental_count + 10,
  category_name = category_name || ' Category'
WHERE
  -- first character of category_name is 'C'
  LEFT(category_name, 1) = 'C'
-- show all updated rows as the query output
RETURNING *;
```

The above query returns the following output:

| category_name      | avg_rental_count  |
|--------------------|-------------------|
| Classics Category  | 12.0064102564102564|
| Comedy Category    | 11.9010101010101010|
| Children Category  | 11.9605809128630705|

And we can finally check that the underlying values in the testing_average_category_rental_counts table has indeed changed:

```sql
SELECT *
FROM testing_average_category_rental_counts
ORDER BY category_name;
```

| category_name      | avg_rental_count      |
|--------------------|-----------------------|
| Action             | 2.1803921568627451    |
| Animation          | 2.3320000000000000    |
| Children category  | 11.9605809128630705   |
| Classics category  | 12.0064102564102564   |
| Comedy category    | 11.9010101010101010   |
| Documentary        | 2.1739130434782609    |
| Drama              | 2.1157684630738523    |
| Family             | 2.1876247504990020    |
| Foreign            | 2.0953346855983773    |
| Games              | 2.0443037974683544    |
| Horror             | 1.8758314855875831    |
| Music              | 1.8568232662192394    |
| New                | 2.0085470085470085    |
| Sci-Fi             | 2.1715976331360947    |
| Sports             | 2.2716763005780347    |
| Travel             | 1.8936651583710407    |

Ok - now that we’ve tested out all the different bits for the UPDATE statement - let’s now update our table for real. We will still use the RETURNING * to show what has changed in the process:



```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;

-- output the entire table by desc avg_rental_count
SELECT *
FROM average_category_rental_counts
ORDER BY
  avg_rental_count DESC;

UPDATE average_category_rental_counts
SET avg_rental_count = FLOOR(avg_rental_count)
WHERE category_name LIKE "%%";

-- output the entire table by desc avg_rental_count
SELECT 
  category_name,
  CAST(avg_rental_count AS INT64) AS avg_rental_count
FROM average_category_rental_counts
ORDER BY
  category_name ASC;
```

| category_name | avg_rental_count |
|---------------|------------------|
| Action        | 2                |
| Animation     | 2                |
| Children      | 1                |
| Classics      | 2                |
| Comedy        | 1                |
| Documentary   | 2                |
| Drama         | 2                |
| Family        | 2                |
| Foreign       | 2                |
| Games         | 2                |
| Horror        | 1                |
| Music         | 1                |
| New           | 2                |
| Sci-Fi        | 2                |
| Sports        | 2                |
| Travel        | 1                |

## Percentile Values

After that quick whirlwind tour of using UPDATE let’s continue with our final calculated field we’ll need to tackle - the percentile field:

> `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?

So let’s take one step back here and revisit one of the previous examples about distribution functions from the Data Exploration section of this course.

Specifically let’s look at the example of me taking those exams as a kid where I scored 95% in an exam and I beat 99% of all the other test takers!

This distribution chart below is a good representation of the same problem we’re facing with this percentile calculation. In the below example - the average test score was 60/100 but my score was 95/100 - landing me in the top 1% of all test takers - or if we interpret the numbers in the chart below, I beat 99% of all the other test takers.

<img width="559" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/3e52b044-a963-47fc-bb76-acaaea9177f3">

So in the same way as me beating the 99% - we need to now generate this same percentile value for each customer when comparing their result - the number of films they’ve watched in a certain category - with all other customer rental counts in that category!

In the end though - we do need to reverse that percentage so we can express the percentile as a “top 2%” insight as we see in the email template sample.

This required percentile calculation actually brings us quite neatly to the next topic that we will need to cover in lots of depth for this Serious SQL course - window functions!

For the rest of this tutorial however - let’s just simply apply some window functions to help us complete these calculations before we dive into the theory and rationale behind window functions.

## Percent Rank Window Function

We can use the PERCENT_RANK window function to easily generate our percentile calculated field - however it only generates decimal percentages from 0 to 1!

All window functions must have an `OVER` clause - and in the case of `PERCENT_RANK` which is actually an ordered analytical function - it must also have an `ORDER BY` clause at the minimum - but a `PARTITION BY` clause can also be used with this window function - which we will demonstrate how to use for our current problem!

For now - you can think of the `PARTITION BY` as a similar version of `GROUP BY` which helps split our dataset into specific “groups” or “window frames” to perform further calculations. For this `percentile` field - we actually need to partition on the `category_name` values as we will be trying to get all of the `percentile` metrics within each unique category

The `ORDER BY` clause is very similar to how it’s used when we want to sort SQL outputs in a specific order - however the ordering is not only required for our `PERCENT_RANK` window function - it is critical!

For our example - we will want to order by the rental_count in descending order in order to return us the expected top N% result that we need for our case study.

Don’t worry if all these terms seem a bit confusing - we will cover them in much more depth in the very next tutorial.

We can use our aggregated rental_count values at a customer_id and category_level in the category_rental_counts temp table we created earlier to generate the required output like so - let’s first inspect the results for customer_id = 1 for all of their records:

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

-- profile just customer_id = 1 values sorted by desc rental_count
SELECT
  customer_id,
  category_name,
  rental_count,
  PERCENT_RANK() OVER (
    PARTITION BY category_name
    ORDER BY rental_count DESC
  ) AS percentile
FROM category_rental_counts
ORDER BY customer_id, rental_count DESC
LIMIT 14;
```
| customer_id | category_name | rental_count | percentile               |
|-------------|---------------|--------------|--------------------------|
| 1           | Classics      | 6            | 0.0021413276231263384    |
| 1           | Comedy        | 5            | 0.006072874493927126     |
| 1           | Drama         | 4            | 0.03                     |
| 1           | Animation     | 2            | 0.38877755511022044      |
| 1           | New           | 2            | 0.2676659528907923       |
| 1           | Action        | 2            | 0.33398821218074654      |
| 1           | Music         | 2            | 0.2040358744394619       |
| 1           | Sports        | 2            | 0.34555984555984554      |
| 1           | Sci-Fi        | 2            | 0.30039525691699603      |
| 1           | Documentary   | 1            | 0.6431535269709544       |
| 1           | Games         | 1            | 0.6004228329809725       |
| 1           | Foreign       | 1            | 0.6178861788617886       |
| 1           | Family        | 1            | 0.654                    |
| 1           | Travel        | 1            | 0.5759637188208617       |

Notice how the percentile values are very low for the `Classics` category - the top ranking record for our customer.

Firstly we will need to multiply the `PERCENT_RANK` output by 100 to make it go from a decimal between 0 and 1 to an actual percentage number between 0 and 100.

However - even if we rounded that new percentile metric to the nearest integer for the `Classics` record - it would still show the value `0` - which might look a bit weird when we generate our customer insight for the email template!

To compile our actual top category insight for `customer_id = 1` for the Classics category - let’s also inspect what the average `rental_count` value was from our `average_category_rental_counts` table.

```sql
SELECT *
FROM average_category_rental_counts
WHERE category_name = 'Classics';
```
| category_name | avg_rental_count |
|---------------|------------------|
| Classics      | 2                |

So based off our top category for our first customer - we might get the following insight if we simply rounded our percentile field:

> You’ve watched 6 Classics films, that’s 4 more than the DVD Rental Co average and puts you in the top 0% of Classics gurus!

Weird right…so instead of rounding - let’s just use the CEILING function to take the upper integer for each of those percentile metrics after we multiply by 100 to bring our decimal value between 0 and 1 to a new value between 0 and 100!

| customer_id | category_name | rental_count | percentile |
|-------------|---------------|--------------|------------|
| 1           | Classics      | 6            | 1          |
| 1           | Comedy        | 5            | 1          |

Great - this will make much more sense to our customers now!

> You’ve watched 6 Classics films, that’s 4 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!

Let’s pop our transformation into a separate temporary table just like all the other components - we can also remove that rental_count column from the table as we will already have the information from our category_rental_counts table for use in our next step.

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

-- inspect top 2 records for customer_id = 1 sorted by ascending percentile
SELECT *
FROM customer_category_percentiles
WHERE customer_id = 1
ORDER BY customer_id, percentile
LIMIT 2;
```

| customer_id | category_name | rental_count | percentile |
|-------------|---------------|--------------|------------|
| 1           | Classics      | 6            | 1          |
| 1           | Comedy        | 5            | 1          |

# Joining Temporary Tables

This approach to splitting out datasets - performing calculations and aggregations at different granularities by using different GROUP BY column inputs or using window functions is a very common technique to split up our various logical components into separate blocks.

This is especially the case for our case study problem as we needed to perform our calculations on the entire dataset before we remove all rows except the top 2 categories for each customer!

We can now easily combine all of the temporary tables together using that same approach we defined in the Joining Multiple Tables tutorial.

Since we would like to keep all of the rental_count records - we can safely use table 1 category_rental_counts as our starting base table for our table joins.

For this example - let’s use the appearance order style of table aliasing just to mix things up a little bit!

Since we know for sure - all of the keys for the following tables also exist in the `category_rental_counts` base - we can safely use inner joins without the fear of accidentally wiping out values which are not present in left and right tables for each join!

The only thing we need to keep an eye on is the columns which we use as the join keys for each of our inner joins - also notice how tables 2, 3 and 4 are joined onto table 1 `category_rental_counts`

Additionally - we will see that table 4 `customer_category_percentiles` will join onto `category_rental_counts` on 2 columns.

Remember to always reference where specific columns are coming from by referencing the alias or table name in the `SELECT` statement!

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;

CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

-- inspect customer_id = 1 rows sorted by percentile
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile;
```

| customer_id | category_name | rental_count | total_rental_count | avg_rental_count | percentile |
|-------------|---------------|--------------|--------------------|------------------|------------|
| 1           | Classics      | 6            | 32                 | 2                | 1          |
| 1           | Comedy        | 5            | 32                 | 1                | 1          |
| 1           | Drama         | 4            | 32                 | 2                | 3          |
| 1           | Music         | 2            | 32                 | 1                | 21         |
| 1           | New           | 2            | 32                 | 2                | 27         |
| 1           | Sci-Fi        | 2            | 32                 | 2                | 31         |
| 1           | Action        | 2            | 32                 | 2                | 34         |
| 1           | Sports        | 2            | 32                 | 2                | 35         |
| 1           | Animation     | 2            | 32                 | 2                | 39         |
| 1           | Travel        | 1            | 32                 | 1                | 58         |
| 1           | Games         | 1            | 32                 | 2                | 61         |
| 1           | Foreign       | 1            | 32                 | 2                | 62         |
| 1           | Documentary   | 1            | 32                 | 2                | 65         |
| 1           | Family        | 1            | 32                 | 2                | 66         |

Ok we are getting closer to our target output table - now let’s add in those final calculations to this table.

## Adding in Calculated Fields

- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer?
- `category_percentage`: What proportion of each customer’s total films watched does this count make?

To spice this example up a little further - let’s drop the temporary table we just created `customer_category_joint_table` and recreate it with these 2 calculations included:

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;

CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  -- round to nearest integer for percentage after multiplying by 100
  ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

-- inspect customer_id = 1 rows sorted by percentile
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile;
```

| customer_id | category_name | rental_count | latest_rental_date      | total_rental_count | avg_rental_count | percentile | average_comparison | category_percentage |
|-------------|---------------|--------------|-------------------------|--------------------|------------------|------------|--------------------|---------------------|
| 1           | Comedy        | 5            | 2005-08-22 19:41:37     | 32                 | 1                | 1          | 4                  | 16                  |
| 1           | Classics      | 6            | 2005-08-19 09:55:16     | 32                 | 2                | 1          | 4                  | 19                  |
| 1           | Drama         | 4            | 2005-08-18 03:57:29     | 32                 | 2                | 3          | 2                  | 13                  |
| 1           | Music         | 2            | 2005-07-09 16:38:01     | 32                 | 1                | 21         | 1                  | 6                   |
| 1           | New           | 2            | 2005-08-19 13:56:54     | 32                 | 2                | 27         | 0                  | 6                   |

Note how we have a division field here - normally we would be applying some sort of casting of either the numerator or denominator to a NUMERIC data type to avoid the dreaded integer floor division which we talked about in the Data Exploration section of this course.

However - we are lucky in this example as that total_rental_count value just so happens to be a NUMERIC type already as it is the output from a SUM function.

So I can already imagine you asking - how do we know the data types?

## Checking Data Types Using the information_schema.columns Table

To check the data types of columns in specific tables you can use the following query and hit the information_schema.columns reference table within PostgreSQL - just note that different SQL flavours will have different ways to inspect the data types of columns and inspect tables further.

This snippet will be really useful anytime you need to inspect data types!

```sql
-- this is a snippet from postgreSQL; not BigQuery
SELECT
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE table_name in ('customer_total_rentals', 'category_rental_counts');
```

There are additional columns which are really useful in the information_schema.columns table including:

- `schema_name`: which database schema is this table from?
- `is_nullable`: can this column contain null values?
- `ordinal_position`: what position is this column inside the table?
- `column_default`: is there a default value for this column?
- `character_maximum_length`: what is the maximum character length of a CHAR or VARCHAR data type column

In the past I actually used these schema tables to perform a ton of data validation and exploration - for example, I would use this query a ton to find all tables in all schemas that consisted of a specific column that I was looking for:

```sql
-- this is a snippet from postgreSQL; not BigQuery
SELECT
  schema_name,
  table_name,
  column_name
FROM information_schema.columns
WHERE column_name ILIKE '%<column_name>%'
```

Then I would start inspecting each table one by one to see if it had other columns which I might want:


```sql
-- this is a snippet from postgreSQL; not BigQuery
SELECT
  schema_name,
  table_name,
  column_name
FROM information_schema.columns
WHERE table_name = 'some-table-from-the-query-above';
```
There are plenty more things you can do with this such as mapping a few shortcuts to your favourite languages or SQL editors and development tools so this is just scratching the surface of what is possible in the world of SQL exploration!

Ok let’s return to the final piece of the puzzle - how can we extract the top 2 rows for each customer?

# Ordering and Filtering Rows with ROW_NUMBER

Ok great - we now have all of our various calculations but we also have all of our categories for every customer, however we only need the top 2 categories.

We can use another ordered analytical function ROW_NUMBER to do this in another window function!

We’ve seen this briefly in a past tutorial but the ROW_NUMBER is used with an OVER clause and adds row numbers for records according to some ORDER BY condition within each “window frame” or “group” of rows assigned by the PARTITION BY clause.

Once the records within each window frame, partition or group are all numbered - the row number simply restarts at 1 for the next group and continues until all of the groups and rows are sorted and numbered.

For our example - we will aim to sort our records within each set of rows with the same customer_id, ordering the rows by rental_count from largest to smallest and also using that latest_rental_date field for each customer’s category as an additional sorting criteria to “deal with ties” (we covered this earlier above!)

We will preference the category with the most recent latest_rental_date so we will also need to use a DESC with this second input to the ORDER BY clause.

We can perform this window function within a CTE and then apply a simple filter to only keep row number records which are less than or equal to 2 only.

Let’s create a final temporary table called top_categories_information with the output from this operation

```sql
CREATE TEMP TABLE complete_joint_dataset_with_rental_date AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

CREATE TEMP TABLE category_rental_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset_with_rental_date
GROUP BY
  customer_id,
  category_name;

CREATE TEMP TABLE average_category_rental_counts AS
SELECT
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY
  category_name;

CREATE TEMP TABLE customer_total_rentals AS
SELECT
  customer_id,
  SUM(rental_count) AS total_rental_count
FROM category_rental_counts
GROUP BY customer_id;

CREATE TEMP TABLE customer_category_percentiles AS
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC
    )
  ) AS percentile
FROM category_rental_counts;

CREATE TEMP TABLE customer_category_joint_table AS
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_count,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  -- round to nearest integer for percentage after multiplying by 100
  ROUND(100 * t1.rental_count / t2.total_rental_count) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4
  ON t1.customer_id = t4.customer_id
  AND t1.category_name = t4.category_name;

CREATE TEMP TABLE top_categories_information AS (
-- use a CTE with the ROW_NUMBER() window function implemented
WITH ordered_customer_category_joint_table AS (
  SELECT
    customer_id,
    ROW_NUMBER() OVER (
      PARTITION BY customer_id
      ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS category_ranking,
    category_name,
    rental_count,
    average_comparison,
    percentile,
    category_percentage
  FROM customer_category_joint_table
)
-- filter out top 2 rows from the CTE for final output
SELECT *
FROM ordered_customer_category_joint_table
WHERE category_ranking <= 2
);

SELECT *
FROM top_categories_information
WHERE customer_id in (1, 2, 3)
ORDER BY customer_id, category_ranking;
```

Let’s now inspect the result for the first 3 customers - it should match exactly what we had for our illustrated simple example above too!

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics     | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy       | 5            | 4                  | 1          | 16                  |
| 2           | 1                | Sports       | 5            | 3                  | 3          | 19                  |
| 2           | 2                | Classics     | 4            | 2                  | 2          | 15                  |
| 3           | 1                | Action       | 4            | 2                  | 5          | 15                  |
| 3           | 2                | Sci-Fi       | 3            | 1                  | 15         | 12                  |

Awesome - this matches not only our illustrated example but it also matches exactly what we need for one of the first output tables for our case study!

# Conclusion

This now brings us to the conclusion of this tutorial - in the next one we will be diving into a whirlwind tour of window functions to better understand what we did with those percentile and category_ranking fields (and more!)

In this tutorial we covered the following concepts as we continued our SQL problem solving exercise:

1. Apply a split, aggregate and join strategy to combine multiple calculated fields into a single table via multiple table joins
2. How to “deal with ties” for row sorting and ordering, and why randomly selecting rows is not a great idea
3. Thinking through customer experience when we make technical decisions that might negatively impact a message or insight
4. Using multiple CTEs in a single SQL statement
5. How to use UPDATE to adjust column values in an existing table, including how to target only specific rows which meet certain criteria
6. Quick overview on the usage of PERCENT_RANK and ROW_NUMBER window functions
7. How to identify column data types and quickly explore database tables to search for columns or table names using information_schema.columns

Let’s now move onto the next tutorial and cover almost everything you need to know about window functions!


