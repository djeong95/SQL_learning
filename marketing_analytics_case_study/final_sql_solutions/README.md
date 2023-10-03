# 1. Marketing Analytics
To recap what we’ve covered so far - let’s view our entity relationship diagram (ERD) and remind ourselves about the requirements via the email template - then we will systematically step through the entire case study solution from start to finish.

For this tutorial - I will write up the entire solution as though I was preparing a SQL portfolio project myself. This will be a good example to revise when you need to start creating your own portfolio projects!

Firstly let’s introduce a rough structure for this case study solution - this is a new thing I’ve created myself, so hopefully this fun acronym can help you with your future projects!

# 2. PEAR Template
This is something that I came up with recently to try and structure these data case study solutions - I also had the realisation that it might be useful to share the structure before we dive straight into the case study solution itself.

In the same way that we have the structure of STAR technique for behavioural interview questions (Situation, Task, Action, Result) - PEAR is my solution for framing up any data case study which you’d like to showcase as part of your personal portfolio!

P - Problem
What is the business problem and what sort of value can we generate by solving it?

E - Exploration
What available resources do we have and what initial data exploration can we perform to better understand our data?

A - Analysis
This is where we start showing off our arsenal of data analysis techniques in a very structured and systematic manner.

R - Report
Once we’ve finished our analysis we want to package up our various code snippets into a single process and generate the final outputs we need to solve our business problem.

Hopefully you can internalise this project structure for your future portfolio projects - it will help to structure and formalise your work this way as it forces you to really understand all the components of your project!

# 3. Problem

We have been tasked by the DVD Rental Co marketing team to help them generate the analytical inputs required to drive their very first customer email campaign.

The marketing team expects their personalised emails to drive increased sales and engagement from the DVD Rental Co customer base.

The main initiative is to share insights about each customer’s viewing behaviour to demonstrate DVD Rental Co’s relentless focus on customer experience.

The insights requested by the marketing team include key statistics about each customer’s top 2 categories and favourite actor. There are also 3 personalised recommendations based off each customer’s previous viewing history as well as titles which are popular with other customers.

### 3.1.1. Category Insights
#### 3.1.1.1. Top Category
1. What was the top category watched by total rental count?
2. How many total films have they watched in their top category and how does it compare to the DVD Rental Co customer base?
 - How many more films has the customer watched compared to the average DVD Rental Co customer?
- How does the customer rank in terms of the top X% compared to all other customers in this film category?
3. What are the top 3 film recommendations in the top category ranked by total customer rental count which the customer has not seen before?

#### 3.1.1.2. Second Category
4. What is the second ranking category by total rental count?
5. What proportion of each customer’s total films watched does this count make?
6. What are top 3 recommendations for the second category which the customer has not yet seen before?

### 3.1.2. Actor Insights
7. Which actor has featured in the customer’s rental history the most?
8. How many films featuring this actor has been watched by the customer?
9. What are the top 3 recommendations featuring this same actor which have not been watched by the customer?

## Entity Relationship Diagram
We’ve also been provided with the entity relationship diagram (ERD) as shown below with all foreign table keys and column data types:

<img width="532" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/0de0180e-d5dd-45a8-a9fb-3846502e0db5">

# 4. Exploration

We have a total of 7 tables in our ERD for this case study highlighting the important columns which we should use to join our tables for our data analysis task.

## Table Investigation

We analysed the relationships between the most important columns from each table to determine if there were one-many, many-one or many-to-many relationships to further guide our table joining investigation.

We were required to compare the following tables detailed in the following animated GIF for our category level insights - and a similar sequence was followed but with the actor tables instead of the category tables:

![gif](https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/66679c7c-88ae-4953-a4c5-67a3ef9ea009)

## Join Column Analysis Example 1

We first explored the different join column values to check the coverage of our join columns and also identify any duplicates in joining column values.

<img width="475" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/b1d14158-4ce6-4a1b-876d-297774b6d1c0">

Our analysis informed our choice of SQL table join method. In our examples - most of the join column values have a 100% overlap with no duplicate column values so we have decided to proceed with exclusively INNER JOIN for our SQL scripts. We also confirmed that there was no difference in the row counts by comparing LEFT JOIN and INNER JOIN results with our previous queries.

Here is a sample of the analysis we completed for our exploration of the dvd_rentals.rental and dvd_rentals.inventory tables:


1. Perform an anti join to check which column values exist in dvd_rentals.rental but not in dvd_rentals.inventory

```sql
-- how many foreign keys only exist in the left table and not in the right?

SELECT
  COUNT(DISTINCT rental.inventory_id) as count
FROM `younglapalma.dvd_rentals.rental` as rental
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.inventory` as inventory
  WHERE rental.inventory_id = inventory.inventory_id
);
```
count: 0


2. Now check the right side table using the same process: dvd_rentals.inventory

```sql
-- 2. Now check the right side table using the same process: dvd_rentals.inventory
SELECT
  COUNT(DISTINCT inventory.inventory_id) as count
FROM `younglapalma.dvd_rentals.inventory` as inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.rental` as rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```

count: 1

3. There seems to be a single value which is not showing up - let’s investigate which film it is:


```sql
SELECT
  *
FROM `younglapalma.dvd_rentals.inventory` as inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.rental` as rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```

| inventory_id | film_id | store_id | last_update           |
|--------------|---------|----------|-----------------------|
| 5            | 1       | 2        | 2006-02-15 05:09:17   |

Conclusion: although there is a single inventory_id record which is missing from the dvd_rentals.rental table - there might be no issues with this discrepancy as it seems that some inventory might just never be rented out to customers at the retail rental stores.

4. Finally - let’s confirm that both left and inner joins do not differ at all when we look at the resulting row counts from the joint tables:

```sql
-- 4. Finally - let’s confirm that both left and inner joins do not differ at all when we look at the resulting row counts from the joint tables:
CREATE TEMP TABLE left_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM `younglapalma.dvd_rentals.rental` as rental
LEFT JOIN `younglapalma.dvd_rentals.inventory` as inventory
  ON rental.inventory_id = inventory.inventory_id;

CREATE TEMP TABLE inner_rental_join AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM `younglapalma.dvd_rentals.rental` as rental
INNER JOIN `younglapalma.dvd_rentals.inventory` as inventory
  ON rental.inventory_id = inventory.inventory_id;

-- Output SQL
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)
UNION ALL
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM inner_rental_join
);
```

| join_type   | record_count | unique_key_values |
|-------------|--------------|-------------------|
| inner join  | 16044        | 4580              |
| left join   | 16044        | 4580              |

We perform this same analysis for all of our tables within our core tables and concluded that the distribution for each of the join keys are as expected and are similar to what we see for these first 2 tables.

## 4.3. Join Column Analysis Example 2
We also need to investigate the relationships between the actor_id and film_id columns within the dvd_rentals.film_actor table.

Intuitively - we can hypothesise that one single actor might show up in multiple films and one film can have multiple actors. This is known as a many-to-many relationship.

Let’s perform some analysis on the data tables to see if our hunch is on point:

```sql
WITH actor_film_counts AS (
  SELECT
    actor_id,
    COUNT(DISTINCT film_id) AS film_count
  FROM `younglapalma.dvd_rentals.film_actor`
  GROUP BY actor_id
)
SELECT
  film_count,
  COUNT(*) AS total_actors
FROM actor_film_counts
GROUP BY film_count
ORDER BY film_count DESC;
```

Below we can conclude that 14 is the minimum number of films that one actor stars in - these actors are absolute stars!

| film_count | total_actors |
|------------|--------------|
| 42         | 1            |
| 41         | 1            |
| 40         | 1            |
| 39         | 1            |
| 37         | 1            |
| 36         | 1            |
| 35         | 6            |
| 34         | 5            |
| 33         | 13           |
| 32         | 10           |
| 31         | 16           |
| 30         | 16           |
| 29         | 11           |
| 28         | 11           |
| 27         | 17           |
| 26         | 14           |
| 25         | 19           |
| 24         | 14           |
| 23         | 8            |
| 22         | 10           |
| 21         | 5            |
| 20         | 9            |
| 19         | 4            |
| 18         | 2            |
| 16         | 1            |
| 15         | 2            |
| 14         | 1            |

Let’s also confirm that there are multiple actors per film also (although this should be pretty obvious!):

```sql
WITH film_actor_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT actor_id) AS actor_count
  FROM `younglapalma.dvd_rentals.film_actor`
  GROUP BY film_id
)
SELECT
  actor_count,
  COUNT(*) AS total_films
FROM film_actor_counts
GROUP BY actor_count
ORDER BY actor_count DESC;
```
Well - what do you know? Some films only consist of a single actor! Good thing we checked after all!

| film_count | total_actors |
|------------|--------------|
| 42         | 1            |
| 41         | 1            |
| 40         | 1            |
| 39         | 1            |
| 37         | 1            |
| 36         | 1            |
| 35         | 6            |
| 34         | 5            |
| 33         | 13           |
| 32         | 10           |
| 31         | 16           |
| 30         | 16           |
| 29         | 11           |
| 28         | 11           |
| 27         | 17           |
| 26         | 14           |
| 25         | 19           |
| 24         | 14           |
| 23         | 8            |
| 22         | 10           |
| 21         | 5            |
| 20         | 9            |
| 19         | 4            |
| 18         | 2            |
| 16         | 1            |
| 15         | 2            |
| 14         | 1            |

In conclusion - we can see that there is indeed a many to many relationship of the film_id and the actor_id columns within the dvd_rentals.film_actor table so we must take extreme care when we are joining these 2 tables as part of our analysis in the next section of this project!

# 5. Analysis
Now that we’ve identified the key columns and highlighted some things we need to keep in mind when performing some table joins for our data analysis - we need to formalise our plan of attack.

Before we dive into the actual SQL implementation of the final solution, let’s list out all of the steps we will take without any code so we can keep a track of our thought process as we go through the following SQL solution.

At a high level - we will tackle the category insights first before turning our attention to the actor level insights and recommendations.

## 5.1. Solution Plan

### 5.1.1. Category Insights

1. Create a base dataset and join all relevant tables
- complete_joint_dataset
2. Calculate customer rental counts for each category
- category_counts
3. Aggregate all customer total films watched
- total_counts
4. Identify the top 2 categories for each customer
- top_categories
5. Calculate each category’s aggregated average rental count
- average_category_count
6. Calculate the percentile metric for each customer’s top category film count
- top_category_percentile
7. Generate our first top category insights table using all previously generated tables
- top_category_insights
8. Generate the 2nd category insights
- second_category_insights

### 5.1.2. Category Recommendations
1. Generate a summarised film count table with the category included, we will use this table to rank the films by popularity
- film_counts
2. Create a previously watched films for the top 2 categories to exclude for each customer
- category_film_exclusions
3. Finally perform an anti join from the relevant category films on the exclusions and use window functions to keep the top 3 from each category by popularity - be sure to split out the recommendations by category ranking
- category_recommendations

### 5.1.3. Actor Insights
1. Create a new base dataset which has a focus on the actor instead of category
- actor_joint_table
2. Identify the top actor and their respective rental count for each customer based off the ranked rental counts
- top_actor_counts

### 5.1.4. Actor Recommendations
1. Generate total actor rental counts to use for film popularity ranking in later steps
- actor_film_counts
2. Create an updated film exclusions table which includes the previously watched films like we had for the category recommendations - but this time we need to also add in the films which were previously recommended
- actor_film_exclusions
3. Apply the same ANTI JOIN technique and use a window function to identify the 3 valid film recommendations for our customers
- actor_recommendations

## 5.2. Category Insights
> Note: This following section will be light on explanations and will require you to read further into the code to better understand what is going on. Be sure to refer to previous tutorials in case you forget about join techniques and specific window functions!

### 5.2.1. Create Base Dataset
We first created a complete_joint_dataset which joins multiple tables together after analysing the relationships between each table to confirm if there was a one-to-many, many-to-one or a many-to-many relationship for each of the join columns.

We also included the rental_date column to help us split ties for rankings which had the same count of rentals at a customer level - this helps us prioritise film categories which were more recently viewed.

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name,
  -- also included rental_date for sorting purposes
  rental.rental_date
FROM `younglapalma.dvd_rentals.rental` as rental
INNER JOIN `younglapalma.dvd_rentals.inventory` as inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` as film
  ON inventory.film_id = film.film_id
INNER JOIN `younglapalma.dvd_rentals.film_category` as film_category
  ON film.film_id = film_category.film_id
INNER JOIN `younglapalma.dvd_rentals.category` as category
  ON film_category.category_id = category.category_id;

SELECT * FROM complete_joint_dataset limit 10;
```

| customer_id | film_id | title           | category_id | category_name |
|-------------|---------|-----------------|-------------|---------------|
| 130         | 80      | BLANKET BEVERLY | 8           | Family        |
| 459         | 333     | FREAKY POCUS    | 12          | Music         |
| 408         | 373     | GRADUATE LORD   | 3           | Children      |
| 333         | 535     | LOVE SUICIDES   | 11          | Horror        |
| 222         | 450     | IDOLS SNATCHERS | 3           | Children      |
| 549         | 613     | MYSTIC TRUMAN   | 5           | Comedy        |
| 269         | 870     | SWARM GOLD      | 11          | Horror        |
| 239         | 510     | LAWLESS VISION  | 2           | Animation     |
| 126         | 565     | MATRIX SNOWMAN  | 9           | Foreign       |
| 399         | 396     | HANGING DEEP    | 7           | Drama         |

### 5.2.2. Category Counts

We then created a follow-up table which uses the complete_joint_dataset to aggregate our data and generate a rental_count and the latest rental_date for our ranking purposes downstream.

```sql
CREATE TEMP TABLE category_counts AS
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_joint_dataset
GROUP BY
  customer_id,
  category_name;

SELECT *
FROM category_counts
WHERE customer_id = 1
ORDER BY
    rental_count DESC,
    latest_rental_date DESC;
```
| customer_id | category_name | rental_count | latest_rental_date    |
|-------------|---------------|--------------|-----------------------|
| 1           | Classics      | 6            | 2005-08-19 09:55:16   |
| 1           | Comedy        | 5            | 2005-08-22 19:41:37   |
| 1           | Drama         | 4            | 2005-08-18 03:57:29   |
| 1           | Animation     | 2            | 2005-08-22 20:03:46   |
| 1           | Sci-Fi        | 2            | 2005-08-21 23:33:57   |
| 1           | New           | 2            | 2005-08-19 13:56:54   |
| 1           | Action        | 2            | 2005-08-17 12:37:54   |
| 1           | Music         | 2            | 2005-07-09 16:38:01   |
| 1           | Sports        | 2            | 2005-07-08 07:33:56   |
| 1           | Family        | 1            | 2005-08-02 18:01:38   |
| 1           | Documentary   | 1            | 2005-08-01 08:51:04   |
| 1           | Foreign       | 1            | 2005-07-28 16:18:23   |
| 1           | Travel        | 1            | 2005-07-11 10:13:46   |
| 1           | Games         | 1            | 2005-07-08 03:17:05   |

### 5.2.3. Total Counts

We will then use this category_counts table to generate our total_counts table.

```sql
CREATE TEMP TABLE total_counts AS
SELECT
  customer_id,
  SUM(rental_count) AS total_count
FROM category_counts
GROUP BY
  customer_id;

SELECT *
FROM total_counts
LIMIT 5;
```

| customer_id | total_count |
|-------------|-------------|
| 184         | 23          |
| 87          | 30          |
| 273         | 35          |
| 477         | 22          |
| 550         | 32          |


### 5.2.4. Top Categories

We can also use a simple DENSE_RANK window function to generate a ranking of categories for each customer.

We will also split arbitrary ties by preferencing the category which had the most recent latest_rental_date value we generated in the category_counts for this exact purpose. To further prevent any ties - we will also sort the category_name in alphabetical (ascending) order just in case!


```sql
CREATE TEMP TABLE top_categories AS
WITH ranked_cte AS (
  SELECT
    customer_id,
    category_name,
    rental_count,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY
        rental_count DESC,
        latest_rental_date DESC,
        category_name
    ) AS category_rank
  FROM category_counts
)
SELECT * FROM ranked_cte
WHERE category_rank <= 2;

SELECT *
FROM top_categories
LIMIT 5;
```
| customer_id | category_name | rental_count | category_rank |
|-------------|---------------|--------------|---------------|
| 1           | Classics      | 6            | 1             |
| 1           | Comedy        | 5            | 2             |
| 2           | Sports        | 5            | 1             |
| 2           | Classics      | 4            | 2             |
| 3           | Action        | 4            | 1             |


### 5.2.5. Average Category Count

Next we will need to use the category_counts table to generate the average aggregated rental count for each category rounded down to the nearest integer using the FLOOR function

```sql
CREATE TEMP TABLE average_category_count AS
SELECT
  category_name,
  FLOOR(AVG(rental_count)) AS category_average
FROM category_counts
GROUP BY category_name;

SELECT *
FROM average_category_count
ORDER BY
  category_average DESC,
  category_name;
```
| category_name | category_average |
|---------------|------------------|
| Action        | 2                |
| Animation     | 2                |
| Classics      | 2                |
| Documentary   | 2                |
| Drama         | 2                |
| Family        | 2                |
| Foreign       | 2                |
| Games         | 2                |
| New           | 2                |
| Sci-Fi        | 2                |
| Sports        | 2                |
| Children      | 1                |
| Comedy        | 1                |
| Horror        | 1                |
| Music         | 1                |
| Travel        | 1                |


### 5.2.6. Top Category Percentile
Now we need to compare each customer’s top category rental_count to all other DVD Rental Co customers - we do this using a combination of a LEFT JOIN and a PERCENT_RANK window function ordered by descending rental count to show the required top N% customer insight.

We will also use a CASE WHEN to replace a 0 ranking value to 1 as it doesn’t make sense for the customer to be in the Top 0%!

```sql
CREATE TEMP TABLE top_category_percentile AS
WITH calculated_cte AS (
SELECT
  top_categories.customer_id,
  top_categories.category_name AS top_category_name,
  top_categories.rental_count,
  category_counts.category_name,
  top_categories.category_rank,
  PERCENT_RANK() OVER (
    PARTITION BY category_counts.category_name
    ORDER BY category_counts.rental_count DESC
  ) AS raw_percentile_value
FROM category_counts
LEFT JOIN top_categories
  ON category_counts.customer_id = top_categories.customer_id
)
SELECT
  customer_id,
  category_name,
  rental_count,
  category_rank,
  CASE
    WHEN ROUND(100 * raw_percentile_value) = 0 THEN 1
    ELSE ROUND(100 * raw_percentile_value)
  END AS percentile
FROM calculated_cte
WHERE
  category_rank = 1
  AND top_category_name = category_name;

SELECT *
FROM top_category_percentile
LIMIT 10;
```

| customer_id | category_name | rental_count | percentile |
|-------------|---------------|--------------|------------|
| 323         | Action        | 7            | 1          |
| 506         | Action        | 7            | 1          |
| 151         | Action        | 6            | 1          |
| 410         | Action        | 6            | 1          |
| 126         | Action        | 6            | 1          |
| 51          | Action        | 6            | 1          |
| 487         | Action        | 6            | 1          |
| 363         | Action        | 6            | 1          |
| 209         | Action        | 6            | 1          |
| 560         | Action        | 6            | 1          |

### 5.2.7. 1st Category Insights

Let’s now compile all of our previous temporary tables into a single category_insights table with what we have so far - we will use our most recently generated top_category_percentile table as the base and LEFT JOIN our average table to generate an average_comparison column.


```sql
CREATE TEMP TABLE first_category_insights AS
SELECT
  base.customer_id,
  base.category_name,
  base.rental_count,
  base.rental_count - average.category_average AS average_comparison,
  base.percentile
FROM top_category_percentile AS base
LEFT JOIN average_category_count AS average
  ON base.category_name = average.category_name;

SELECT *
FROM first_category_insights
LIMIT 10;
```

| customer_id | category_name | rental_count | average_comparison | percentile |
|-------------|---------------|--------------|--------------------|------------|
| 323         | Action        | 7            | 5                  | 1          |
| 506         | Action        | 7            | 5                  | 1          |
| 151         | Action        | 6            | 4                  | 1          |
| 410         | Action        | 6            | 4                  | 1          |
| 126         | Action        | 6            | 4                  | 1          |
| 51          | Action        | 6            | 4                  | 1          |
| 487         | Action        | 6            | 4                  | 1          |
| 363         | Action        | 6            | 4                  | 1          |
| 209         | Action        | 6            | 4                  | 1          |
| 560         | Action        | 6            | 4                  | 1          |

### 5.2.8. 2nd Category Insights

Our second ranked category insight is pretty simple as we only need our top_categories table and the total_counts table to process our insights.

The only thing to note here is that we’ll need to cast one of our fraction components of our total_percentage column to avoid the dreaded integer floor division!

```sql
CREATE TEMP TABLE second_category_insights AS
SELECT
  top_categories.customer_id,
  top_categories.category_name,
  top_categories.rental_count,
  -- need to cast as NUMERIC to avoid INTEGER floor division!
  ROUND(
    100 * CAST(top_categories.rental_count AS INT64) / total_counts.total_count
  ) AS total_percentage
FROM top_categories
LEFT JOIN total_counts
  ON top_categories.customer_id = total_counts.customer_id
WHERE category_rank = 2;

SELECT *
FROM second_category_insights
LIMIT 10;
```

| customer_id | category_name | rental_count | total_percentage |
|-------------|---------------|--------------|------------------|
| 184         | Drama         | 3            | 13               |
| 87          | Sci-Fi        | 3            | 10               |
| 477         | Travel        | 3            | 14               |
| 273         | New           | 4            | 11               |
| 550         | Drama         | 4            | 13               |
| 51          | Drama         | 4            | 12               |
| 394         | Documentary   | 3            | 14               |
| 272         | Documentary   | 3            | 15               |
| 70          | Music         | 2            | 11               |
| 190         | Classics      | 3            | 11               |

## 5.3. Category Recommendations

### 5.3.1. Film Counts
We wil first generate another total rental count aggregation from our base table complete_joint_dataset - however this time we will use the film_id and title instead of the category - we still need to keep the category_name in our aggregation - so we will need to use a window function instead of a group by to perform this step.

The DISTINCT is really important for this query - if we were to omit it we would end up with duplicates in our table, which is definitely not what we want!

```sql
CREATE TEMP TABLE film_counts AS
SELECT DISTINCT
  film_id,
  title,
  category_name,
  COUNT(*) OVER (
    PARTITION BY film_id
  ) AS rental_count
FROM complete_joint_dataset;

SELECT *
FROM film_counts
ORDER BY rental_count DESC
LIMIT 10;
```

| film_id | title               | category_name | rental_count |
|---------|---------------------|---------------|--------------|
| 103     | BUCKET BROTHERHOOD  | Travel        | 34           |
| 738     | ROCKETEER MOTHER    | Foreign       | 33           |
| 331     | FORWARD TEMPLE      | Games         | 32           |
| 489     | JUGGLER HARDLY      | Animation     | 32           |
| 767     | SCALAWAG DUCK       | Music         | 32           |
| 382     | GRIT CLOCKWORK      | Games         | 32           |
| 730     | RIDGEMONT SUBMARINE | New           | 32           |
| 973     | WIFE TURN           | Documentary   | 31           |
| 621     | NETWORK PEAK        | Family        | 31           |
| 1000    | ZORRO ARK           | Comedy        | 31           |

### 5.3.2. Category Film Exclusions

For the next step in our recommendation analysis - we will need to generate a table with all of our customer’s previously watched films so we don’t recommend them something which they’ve already seen before.

We will use the complete_joint_dataset base table to get this information - it is pretty straightforward, the only thing to keep in mind is how we will perform an ANTI JOIN with our previous film_counts table so we need to also keep the same film_id column to use as our join column.

We also want to make sure that the DISTINCT is also applied for this table too - it is not as important as in our last step, but it would be best practice to apply it here also!

Note: we could also keep the title and category_name columns but they are redundant and we want to minimise the amount of data we need to use!

```sql
CREATE TEMP TABLE category_film_exclusions AS
SELECT DISTINCT
  customer_id,
  film_id
FROM complete_joint_dataset;

SELECT *
FROM category_film_exclusions
LIMIT 10;
```

| customer_id | film_id |
|-------------|---------|
| 596         | 103     |
| 176         | 121     |
| 459         | 724     |
| 375         | 641     |
| 153         | 730     |
| 1           | 480     |
| 291         | 285     |
| 144         | 93      |
| 158         | 786     |
| 211         | 962     |

### 5.3.3. Final Category Recommendations

Finally we have arrived at the final point of our category recommendations analysis where we need to perform an ANTI JOIN on our category_film_exclusions table using a WHERE NOT EXISTS SQL implementation for our top 2 categories found in the top_categories table we generated a few steps prior.

After this exclusion - we will then perform a window function to select the top 3 films for each of the top 2 categories per customer. To avoid random ties - we will sort by the title alphabetically in case the rental_count values are equal in the ORDER BY clause for our window function.

We also need to keep our category_rank column in our final output so we can easily identify our recommendations for each customer’s preferred categories.

This ANTI JOIN is likely to be the most complex and challenging piece to understand in this entire analysis - please do not go past this point until you understand what is going on!

| customer_id | category_name | category_rank | film_id | title             | rental_count | reco_rank |
|-------------|---------------|---------------|---------|-------------------|--------------|-----------|
| 1           | Classics      | 1             | 891     | TIMBERLAND SKY    | 31           | 1         |
| 1           | Classics      | 1             | 358     | GILMORE BOILED    | 28           | 2         |
| 1           | Classics      | 1             | 951     | VOYAGE LEGALLY    | 28           | 3         |
| 1           | Comedy        | 2             | 1000    | ZORRO ARK         | 31           | 1         |
| 1           | Comedy        | 2             | 127     | CAT CONEHEADS     | 30           | 2         |
| 1           | Comedy        | 2             | 638     | OPERATION OPERATION| 27          | 3         |

## 5.4. Actor Insights

### 5.4.1. Actor Joint Table

For this entire analysis on actors - we will need to create a new base table as we will need to introduce the dvd_rentals.film_actor and dvd_rentals.actor tables to extract all the required data points we need for the final output.

We should also check that the combination of rows in our final table is expected because we should see many more rows than previously used in our categories insights as there is a many-to-many relationship between film_id and actor_id as we alluded to earlier in our data exploration section of this case study.

We also included the rental_date column in this table so we can use it in case there are any ties - just like our previous analysis for the top categories piece.

```sql
CREATE TEMP TABLE actor_joint_dataset AS
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM `younglapalma.dvd_rentals.rental` as rental
INNER JOIN `younglapalma.dvd_rentals.inventory` as inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` as film
  ON inventory.film_id = film.film_id
-- different to our previous base table as we know use actor tables
INNER JOIN `younglapalma.dvd_rentals.film_actor` as film_actor
  ON film.film_id = film_actor.film_id
INNER JOIN `younglapalma.dvd_rentals.actor` as actor
  ON film_actor.actor_id = actor.actor_id;

-- show the counts and count distinct of a few important columns
SELECT
  COUNT(*) AS total_row_count,
  COUNT(DISTINCT rental_id) AS unique_rental_id,
  COUNT(DISTINCT film_id) AS unique_film_id,
  COUNT(DISTINCT actor_id) AS unique_actor_id,
  COUNT(DISTINCT customer_id) AS unique_customer_id
FROM actor_joint_dataset;
```
| total_row_count | unique_rental_id | unique_film_id | unique_actor_id | unique_customer_id |
|-----------------|------------------|----------------|-----------------|--------------------|
| 87980           | 16004            | 955            | 200             | 599                |

```sql
-- show the first 10 rows
SELECT *
FROM actor_joint_dataset
LIMIT 10;
```

| customer_id | rental_id | film_id | rental_date           | title           | actor_id | first_name | last_name |
|-------------|-----------|---------|-----------------------|-----------------|----------|------------|-----------|
| 130         | 1         | 80      | 2005-05-24 22:53:30   | BLANKET BEVERLY | 200      | THORA      | TEMPLE    |
| 130         | 1         | 80      | 2005-05-24 22:53:30   | BLANKET BEVERLY | 193      | BURT       | TEMPLE    |
| 130         | 1         | 80      | 2005-05-24 22:53:30   | BLANKET BEVERLY | 173      | ALAN       | DREYFUSS  |
| 130         | 1         | 80      | 2005-05-24 22:53:30   | BLANKET BEVERLY | 16       | FRED       | COSTNER   |
| 459         | 2         | 333     | 2005-05-24 22:54:33   | FREAKY POCUS    | 147      | FAY        | WINSLET   |
| 459         | 2         | 333     | 2005-05-24 22:54:33   | FREAKY POCUS    | 127      | KEVIN      | GARLAND   |
| 459         | 2         | 333     | 2005-05-24 22:54:33   | FREAKY POCUS    | 105      | SIDNEY     | CROWE     |
| 459         | 2         | 333     | 2005-05-24 22:54:33   | FREAKY POCUS    | 103      | MATTHEW    | LEIGH     |
| 459         | 2         | 333     | 2005-05-24 22:54:33   | FREAKY POCUS    | 42       | TOM        | MIRANDA   |
| 408         | 3         | 373     | 2005-05-24 23:03:39   | GRADUATE LORD   | 140      | WHOOPI     | HURT      |

### 5.4.2. Top Actor Counts

We can now generate our rental counts per actor and since we are only interested in the top actor for each of our customers - we can also perform a filter step to just keep the top actor records and counts for our downstream insights.

We will break up our analysis into separate CTEs so we can see the entire process without introducing more complex window functions within the initial GROUP BY queries.

```sql
CREATE TEMP TABLE actor_joint_dataset AS
SELECT
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM `younglapalma.dvd_rentals.rental` as rental
INNER JOIN `younglapalma.dvd_rentals.inventory` as inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` as film
  ON inventory.film_id = film.film_id
-- different to our previous base table as we know use actor tables
INNER JOIN `younglapalma.dvd_rentals.film_actor` as film_actor
  ON film.film_id = film_actor.film_id
INNER JOIN `younglapalma.dvd_rentals.actor` as actor
  ON film_actor.actor_id = actor.actor_id;

CREATE TEMP TABLE top_actor_counts AS
WITH actor_counts AS (
  SELECT
    customer_id,
    actor_id,
    first_name,
    last_name,
    COUNT(*) AS rental_count,
    -- we also generate the latest_rental_date just like our category insight
    MAX(rental_date) AS latest_rental_date
  FROM actor_joint_dataset
  GROUP BY
    customer_id,
    actor_id,
    first_name,
    last_name
),
ranked_actor_counts AS (
  SELECT
    actor_counts.*,
    DENSE_RANK() OVER (
      PARTITION BY customer_id
      ORDER BY
        rental_count DESC,
        latest_rental_date DESC,
        -- just in case we have any further ties, we'll throw in the names too!
        first_name,
        last_name
    ) AS actor_rank
  FROM actor_counts
)
SELECT
  customer_id,
  actor_id,
  first_name,
  last_name,
  rental_count
FROM ranked_actor_counts
WHERE actor_rank = 1;

SELECT *
FROM top_actor_counts
LIMIT 10;
```

| customer_id | actor_id | first_name | last_name  | rental_count |
|-------------|----------|------------|------------|--------------|
| 1           | 37       | VAL        | BOLGER     | 6            |
| 2           | 107      | GINA       | DEGENERES  | 5            |
| 3           | 150      | JAYNE      | NOLTE      | 4            |
| 4           | 102      | WALTER     | TORN       | 4            |
| 5           | 12       | KARL       | BERRY      | 4            |
| 6           | 191      | GREGORY    | GOODING    | 4            |
| 7           | 65       | ANGELA     | HUDSON     | 5            |
| 8           | 167      | LAURENCE   | BULLOCK    | 5            |
| 9           | 23       | SANDRA     | KILMER     | 3            |
| 10          | 12       | KARL       | BERRY      | 4            |

## 5.5. Actor Recommendations

Finally we are up to the last hurdle of our analysis stage!


### 5.5.1. Actor Film Counts

We need to generate aggregated total rental counts across all customers by actor_id and film_id so we can join onto our top_actor_counts table - this time it’s a little more complicated than just using our simple window function like before!

Since we have now introduced many many more rows than actual rentals - we will need to perform a split aggregation on our table and perform an additional left join back to our base table in order to obtain the right rental_count values.

The DISTINCT is really important in the final part of the CTE as it will remove duplicates which will have a huge impact on our downstream joins later!

```sql

CREATE TEMP TABLE actor_film_counts AS
WITH film_counts AS (
  SELECT
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  -- why do we keep the title here? can you figure out why?
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;

SELECT *
FROM actor_film_counts
LIMIT 10;
```
| film_id | actor_id | rental_count |
|---------|----------|--------------|
| 80      | 200      | 12           |
| 80      | 193      | 12           |
| 80      | 173      | 12           |
| 80      | 16       | 12           |
| 333     | 147      | 17           |
| 333     | 127      | 17           |
| 333     | 105      | 17           |
| 333     | 103      | 17           |
| 333     | 42       | 17           |
| 373     | 140      | 16           |

### 5.5.2. Actor Film Exclusions
We can perform the same steps we used to create the category_film_exclusions table - however we also need to UNION our exclusions with the relevant category recommendations that we have already given our customers.

The rationale behind this - customers would not want to receive a recommendation for the same film twice in the same email!

```sql
CREATE TEMP TABLE actor_film_exclusions AS
-- repeat the first steps as per the category exclusions
-- we'll use our original complete_joint_dataset as the base here
-- can you figure out why???
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_joint_dataset
)
-- we use a UNION to combine the previously watched and the recommended films!
UNION ALL
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM category_recommendations
);

SELECT DISTINCT *
FROM actor_film_exclusions
LIMIT 10;
```

| customer_id | film_id |
|-------------|---------|
| 493         | 567     |
| 114         | 789     |
| 596         | 103     |
| 176         | 121     |
| 459         | 724     |
| 375         | 641     |
| 153         | 730     |
| 291         | 285     |
| 1           | 480     |
| 144         | 93      |

### 5.5.3. Final Actor Recommendations

We can now conclude our analysis phase with this final ANTI JOIN and DENSE_RANK query to perform the same operation as category insights previously - only this time we will use top_actor_counts, actor_film_counts and actor_film_exclusions tables for our analysis.



# 8. Final Quiz Questions
The following questions are part of the final case study quiz - these are example questions the Marketing team might be interested in!

1. Which film title was the most recommended for all customers?

```sql
WITH all_recommendations AS (
  (SELECT 
    customer_id,
    title
  FROM `younglapalma.dvd_rentals.temp_actor_recommendations`)
UNION ALL
  (SELECT
    customer_id,
    title
  FROM `younglapalma.dvd_rentals.temp_category_recommendations`)
)
SELECT
  title,
  COUNT(*) as count
FROM all_recommendations
GROUP BY title
ORDER BY COUNT(*) DESC;
```

DOGMA FAMILY

2. How many customers were included in the email campaign?

```sql
-- How many customers were included in the email campaign?
WITH total_customers AS (
  (SELECT
    customer_id
  FROM `younglapalma.dvd_rentals.temp_complete_joint_dataset`)
UNION ALL
  (SELECT
    customer_id
  FROM `younglapalma.dvd_rentals.temp_actor_joint_dataset`)
)
SELECT  
  COUNT(DISTINCT customer_id)
FROM total_customers
```
599

3. Out of all the possible films - what percentage coverage do we have in our recommendations? (total unique films recommended divided by total available films)

```sql
WITH all_recommendations AS (
  (SELECT 
    customer_id,
    title
  FROM `younglapalma.dvd_rentals.temp_actor_recommendations`)
UNION ALL
  (SELECT
    customer_id,
    title
  FROM `younglapalma.dvd_rentals.temp_category_recommendations`)
),
recommendations AS (
  SELECT COUNT(DISTINCT title) AS _count FROM all_recommendations
),
all_films AS (
  SELECT COUNT(DISTINCT title) AS _count FROM `younglapalma.dvd_rentals.film`
)
SELECT
  all_films._count AS `all`,
  recommendations._count AS recommended,
  ROUND(100*CAST(recommendations._count AS FLOAT64)/CAST(all_films._count AS FLOAT64)) as coverage_percentage
FROM recommendations
CROSS JOIN all_films;
```
25%

4. What is the most popular top category?

```sql
SELECT
  category_name,
  COUNT(*) as category_count
FROM `younglapalma.dvd_rentals.temp_top_categories`
WHERE category_rank = 1
GROUP BY category_name
ORDER BY 2 DESC;
```

Sports


This answer is correct.
```sql

WITH ranked_cte AS (
  SELECT
    category_name,
    COUNT(*) AS category_count,
    ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC) AS category_rank
  FROM first_category_insights
  GROUP BY 1
)
SELECT * FROM ranked_cte WHERE category_rank = 4;
```

6. What is the average percentile ranking for each customer in their top category rounded to the nearest 2 decimal places? Hint: be careful of your data types!
```sql
SELECT AVG(percentile)
FROM `younglapalma.dvd_rentals.temp_top_category_percentile`
GROUP BY category_rank
```

5.10

7. What is the cumulative distribution of the top 5 percentile values for the top category from the first_category_insights table rounded to the nearest round percentage?

```sql
WITH GroupedData AS (
  SELECT
    ROUND(percentile) AS rounded_percentile,
    COUNT(*) AS count_per_percentile
  FROM `younglapalma.dvd_rentals.temp_first_category_insights`
  GROUP BY rounded_percentile
)

SELECT
  rounded_percentile,
  count_per_percentile,
  ROUND(100 * CUME_DIST() OVER (ORDER BY rounded_percentile)) AS cumulative_distribution
FROM GroupedData
ORDER BY rounded_percentile;
```

22.0

8. What is the median of the second category percentage of entire viewing history?

```sql
WITH stats AS (
  SELECT
    APPROX_QUANTILES(total_percentage, 100) AS quantiles
  FROM `younglapalma.dvd_rentals.temp_second_category_insights`
)

SELECT
  stats.quantiles[OFFSET(50)] AS median_value,
FROM stats
;
```

13.0

9. What is the 80th percentile of films watched featuring each customer’s favourite actor?

```sql
WITH stats AS (
  SELECT
    APPROX_QUANTILES(rental_count, 100) AS quantiles
  FROM `younglapalma.dvd_rentals.temp_top_actor_counts`
)

SELECT
  stats.quantiles[OFFSET(80)] AS median_value,
FROM stats
;
```

10. What was the average number of films watched by each customer rounded to the nearest whole number?

```sql
WITH average_count AS (
SELECT
  customer_id,
  COUNT(title) as films_watched
FROM `younglapalma.dvd_rentals.temp_complete_joint_dataset`
GROUP BY customer_id
ORDER BY films_watched DESC)
SELECT ROUND(AVG(films_watched), 0)
FROM average_count;
```

27

11. What is the top combination of top 2 categories and how many customers if the order is relevant (e.g. Horror and Drama is a different combination to Drama and Horror)?

```sql
WITH rank1_table AS (SELECT
  customer_id,
  category_name
FROM `younglapalma.dvd_rentals.temp_top_categories`
WHERE category_rank = 1),

rank1_2_combined AS (SELECT
  rank1.customer_id,
  CONCAT(rank1.category_name, ' ',rank2.category_name) AS rank_category_name,
FROM `younglapalma.dvd_rentals.temp_top_categories` AS rank2
INNER JOIN rank1_table AS rank1 ON
  rank1.customer_id = rank2.customer_id
WHERE rank2.category_rank = 2)

SELECT
  rank_category_name,
  COUNT(customer_id) AS rank_count
FROM rank1_2_combined
GROUP BY rank_category_name
ORDER BY rank_count DESC;
```
Sports and Animation - 11

12. Which actor was the most popular for all customers?


```sql
SELECT
  actor,
  COUNT(*) as actor_count
FROM `younglapalma.dvd_rentals.temp_final_data_asset`
GROUP BY actor
ORDER BY actor_count DESC, actor DESC;
```
Walter Torn

13. How many films on average had customers already seen that feature their favourite actor rounded to closest integer?

```sql
SELECT AVG(rental_count)
FROM `younglapalma.dvd_rentals.temp_top_actor_counts`
```

4

14. What is the most common top categories combination if order was irrelevant and how many customers have this combination? (e.g. Horror and Drama is the same as Drama and Horror)

```sql
WITH rank1_table AS (
  SELECT
    customer_id,
    category_name
  FROM `younglapalma.dvd_rentals.temp_top_categories`
  WHERE category_rank = 1
),

rank1_2_combined AS (
  SELECT
    combined.customer_id,
    -- Create an ordered array of category names
    ARRAY_TO_STRING(ARRAY_AGG(category_name ORDER BY category_name DESC), " and ") AS rank_category_name
  FROM (
    SELECT customer_id, category_name FROM rank1_table
    UNION ALL
    SELECT customer_id, category_name FROM `younglapalma.dvd_rentals.temp_top_categories` WHERE category_rank = 2
  ) AS combined
  GROUP BY customer_id
)

SELECT
  rank_category_name,
  COUNT(customer_id) AS rank_count
FROM rank1_2_combined
GROUP BY rank_category_name
ORDER BY rank_count DESC;
```
Sports and Animation - 14