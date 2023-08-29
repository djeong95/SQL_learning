# Back to the Marketing Case Study

After our long but Gentle Introduction to Table Joins in our last tutorial - we now return to our case study!

In case you’ve forgotten what our original task was - I’ve included the email template as well as the ERD with all of our database tables in the section below.

<img width="561" alt="image" src="https://github.com/djeong95/Opioids-Prescription-Pattern-Analysis/assets/102641321/c6c89a8b-1150-4d2e-b85a-033f585a2e52">


# Design a Plan of Attack

I’m sure you realize that we’re going to have to do some serious table joining in order to generate our data outputs for this case study.

The first thing we need to do is to design a plan of attack by systematically breaking down our required SQL outputs and inspecting exactly which columns we need from the various tables in our ERD.

To keep things simple - let’s first start off with the required SQL outputs for our top categories information.

## Define The Final State

As seen in the previous data overview tutorial, the top categories insight aims to generate the raw data points required to fill in the title and headline insights for the email template.

The key columns that we will need to generate include the following data points at a `customer_id` level:

- `category_name`: The name of the top 2 ranking categories
- `rental_count`: How many total films have they watched in this category
- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer
- `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?
- `category_percentage`: What proportion of total films watched does this category make up?

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics     | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy       | 5            | 4                  | 2          | 16                  |
| 2           | 1                | Sports       | 5            | 3                  | 7          | 19                  |
| 2           | 2                | Classics     | 4            | 2                  | 11         | 15                  |
| 3           | 1                | Action       | 4            | 2                  | 14         | 15                  |
| 3           | 2                | Animation    | 3            | 1                  | 39         | 12                  |
| 4           | 1                | Horror       | 3            | 2                  | 22         | 14                  |
| 4           | 2                | Travel       | 2            | 1                  | 57         | 9                   |

## Reverse Engineering

In the final table output - we can see that the `category_name`, `average_comparison`, `percentile` and `category_percentage` are going to need some intense SQL calculations to generate these outputs.

You might also notice that the `rental_count` value is actually used in all of the calculations:

The top 2 `category_name` values are included for each customer ID because we only sure the category rankings of 1 and 2 for each customer.

Additionally - the `average_comparison` and `percentile` are calculated across all customers at a category level in order to find that DVD Rental Co average.

And finally the `category_percentage` relates to the proportion of each top 2 category to each individual customer’s total number of films watched.

Clearly we can see this `rental_count` is going to be the key for our analysis - so let’s focus on the columns we’ll need to generate this all important value!

If we now remove all the dependent calculated columns: `category_ranking`, `average_comparison`, `percentile` and `category_percentage` - we are left with the following table:

| customer_id | category_name | rental_count |
|-------------|---------------|--------------|
| 1           | Classics     | 6            |
| 1           | Comedy       | 5            |
| 2           | Sports       | 5            |
| 2           | Classics     | 4            |
| 3           | Action       | 4            |
| 3           | Animation    | 3            |
| 4           | Horror       | 3            |
| 4           | Travel       | 2            |

However - there is one more catch!

Since we only have the top 2 categories for each customer in this specific table - how can we calculate those above fields which need to be compared against all customer and categories?

In short - we need to include all of the categories for each customer otherwise we can’t make those calculations!

let’s take a look at all the category rental counts by `customer_id` and `category_name` for just `customer_id = 1`

Now if we take it back even one step further - you should notice that this is going to be some sort of aggregated query to generate this table - something along the lines of:

```sql
SELECT
  customer_id,
  category_name,
  COUNT(*) AS rental_count
FROM the_next_level_down
GROUP BY
  customer_id,
  category_name
```

Now if we were to go one level down and dream up what that granular table might look like - we should have which films a customer has watched and the `category_name` for each rental record:

This is just for `customer_id = 1` but if we generate this same dataset for each customer - we should be ok to generate all of the previously reverse engineered steps further up in this tutorial!

## Identify Key Columns and Start & End Points

So if we need to generate the reverse engineered datasets required to calculate this `rental_count` at a `customer_id` level - or simply put the number of films that a customer has watched in a specific category - we might need the following information for our analysis:

1. `customer_id`
2. `category_name`

The first mental leap we have to make is to realise that we will need to start at the `dvd_rentals.rental` table.

“Why is this the case?” you might be thinking…

The `dvd_rentals.rental` table is the only place where our `customer_id` field exists - it’s the only place where we can identify how many films a customer has watched.

However there is a catch - how can link the records in the `dvd_rentals.rental` table to get the category_name we need for our analysis?

Focusing in on that `category_name` field - the only table which we can get that data point is the `dvd_rentals.category` table.

With this information alone - we now have our start and end points of our data joining journey - now let’s figure out how to combine our data to get these two fields together in the same SQL table output!

## Map the Joining Journey

Starting with our `dvd_rentals.rental` table we can see that we do indeed have the `customer_id` as well as addition columns - but there is no `category_name` in sight, in fact we are very far away!

After inspecting the ERD - we need to get from `dvd_rentals.rental` labeled as number 1 all the way through to table number 5 - `dvd_rentals.category`

Note: you might have realised that the actor tables are not part of this current scenario - don’t worry! The actor tables labeled 6 and 7 will be used later in the case study!

<img width="561" alt="image" src="https://github.com/djeong95/Opioids-Prescription-Pattern-Analysis/assets/102641321/c6c89a8b-1150-4d2e-b85a-033f585a2e52">

When we travel through our ERD from tables 1 through to 5 though we can see that there is indeed a linear journey that we need to take and even further to this - we can use those blue lines to identify the foreign keys (or routes if we want to continue with our metaphor!) that we need to use for our table joining journey!

So here is the final version of our 4 part table joining journey itinerary:

| Join Journey Part | Start         | End           | Foreign Key   |
|-------------------|---------------|---------------|---------------|
| Part 1            | `rental`        | `inventory`     | `inventory_id`  |
| Part 2            | `inventory`     | `film`          | `film_id`       |
| Part 3            | `film`          | `film_category` | `film_id`       |
| Part 4            | `film_category` | `category`      | `category_id`   |

If we were to map out the different tables which we need to join as part of our journey - it would look something like this gif below:

![gif](https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/2691f45e-d065-4315-9ab0-537f799182ad)

Let’s now start analyzing each start and end point in depth so we can proceed on our joining journey with confidence - but first onto the most important part of this tutorial!

# Deciding Which Type of Table Joins to Use

**THIS PART IS IMPORTANT AND INVALUABLE**

If you literally forgot every other thing from this entire tutorial and just remembered this section - I would be ok with it!

To answer this one question “What type of table join should we use?” we actually need to answer 3 additional questions! (crazy right…)

1. What is the purpose of joining these two tables?
2. What is the distribution of foreign keys within each table?
3. How many unique foreign key values exist in each table?

Failing to answer any of these 3 questions before you embark on a SQL table joining journey will harpoon your chances of actually getting the job done!

I found that jumping into joins too early without additional thought and context is THE MOST COMMON ERROR I’ve seen made by SQL developers throughout my career.

To make things even worse - these joining errors are almost always the most **impactful** error because getting your joining strategy wrong can cause all sorts of downstream problems when your data is all messed up!

To avoid these errors - we need to first focus on having a clear purpose.

## What is your Purpose?

Whilst this existential question might be difficult to answer when we think about it at a grander scale - the purpose of table joins is luckily much simpler!

Let’s revisit that summary of our table joining journey so we can identify clearly our purpose for this first table join part.

| Join Journey Part | Start         | End           | Foreign Key   |
|-------------------|---------------|---------------|---------------|
| Part 1            | `rental`        | `inventory`     | `inventory_id`  |
| Part 2            | `inventory`     | `film`          | `film_id`       |
| Part 3            | `film`          | `film_category` | `film_id`       |
| Part 4            | `film_category` | `category`      | `category_id`   |

<img width="571" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/df1e00c2-d4c2-4e90-b0f0-068e3037f01b">

For part 1 of our table joining journey - we can see that we need to start with the `rental` table and end at the `inventory` table using the foreign key `inventory_id` - however does this really tell us anything about the purpose?

We need to fly higher and view our journey from a real bird’s eye view - what are we actually trying to achieve with these series of table joins?

Perhaps it’s been lost in all this talk about purpose and journeys - so let’s go over our first required output shall we?

> We need to generate the `rental_count` calculation - the number of films that a customer has watched in a specific category.

So now if we dive back down to our table joining journey again - we have the `rental` table which consists of the important `customer_id` data point BUT we also know that this rental table also has the actual number of films that each customer has watched.

Show all rentals made by `customer_id = 130`:
```sql
SELECT *
FROM `younglapalma.dvd_rentals.rental`
WHERE customer_id = 130;
```
| rental_id | rental_date          | inventory_id | customer_id | return_date          | staff_id | last_update          |
|-----------|----------------------|--------------|-------------|----------------------|----------|----------------------|
| 1         | 2005-05-24 22:53:30  | 367          | 130         | 2005-05-26 22:04:30  | 1        | 2006-02-15 21:30:53  |
| 746       | 2005-05-29 09:25:10  | 4272         | 130         | 2005-06-02 04:20:10  | 2        | 2006-02-15 21:30:53  |
| ...       | ...                  | ...          | ...         | ...                  | ...      | ...                  |
| 15777     | 2005-08-23 13:29:08  | 4241         | 130         | 2005-08-27 18:50:08  | 2        | 2006-02-15 21:30:53  |


However there is a catch with our `dvd_rentals.rental` table - all the table records are not yet tracked at the `film_id` level - they only have the `inventory_id` which is recorded for each customer’s rental record.

When we look through the other tables in the ERD - we can notice the `inventory` table which can be used to help us get the `film_id` column for each rental.

We can think of the deeper purpose of this join between the two tables in the following way:

> We need to keep all of the customer rental records from `dvd_rentals.rental` and match up each record with its equivalent `film_id` value from the `dvd_rentals.inventory` table.

This is a good example of clarifying what is the “purpose” of a specific table joining operation!

### Left Join or Inner Join?

So once we are clear with the purpose of our join - we need to figure out how we want to implement it using SQL code.

For our first purpose - it looks like we will want to retain all of the data points in the `rental` table and make sure that there are valid `film_id` records for each row that is returned from the resulting table join.

Do you remember the various types of table joins we described in our last tutorial - which join do you think would best suit this scenario or purpose?

Straightaway - your mind should intuitively think about performing either a `LEFT` or an `INNER` join in this scenario.

<img width="567" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/5e001bf5-2246-4fed-a47c-78f46dfc3f2b">

Whenever we need to retain all of the records and match on additional data from a “right” lookup table - these 2 joins should be front of mind.

However - there is one more question. Which join should we use? Should we use the `LEFT` or `INNER` join in our specific scenario?

### 2 Key Analytical Questions

Now here is where things could get tricky, as there are a few unknowns that we need to address as we are matching the `inventory_id` foreign key between the `rental` and `inventory` tables.

1. How many records exist per `inventory_id` value in `rental` or `inventory` tables?
2. How many overlapping and missing unique `foreign key` values are there between the two tables?

**The same questions can be used for all scenarios and is not just limited to this specific table join!**

> 1. How many records exist per `foreign key` value in `left` and `right` tables?
> 2. How many overlapping and missing unique `foreign key` values are there between the two tables?

### The 2 Phase Approach

The best way to answer these follow up questions is usually in 2 separate phases.

Firstly you should think about the actual problem and datasets in terms of what they mean in real life.

Whilst thinking through the problem - we want to generate some hypotheses or assumptions about the data. (Yeah sorta like data science…)

#### Data Hypotheses

Since we know that the `rental` table contains every single rental for each customer - it makes sense logically that each valid rental record in the `rental` table should have a relevant `inventory_id` record as people need to physically hire some item in the store.

Additionally - it also makes sense that a specific item might be rented out by multiple customers at different times as customers return the DVD as shown by the `return_date` column in the dataset.

Now when we think about the `inventory` table - it should follow that every item should have a unique `inventory_id` but there may also be multiple copies of a specific film.

From these 2 key pieces of real life insight - we can generate some hypotheses about our 2 datasets.

1. The number of unique `inventory_id` records will be equal in both `dvd_rentals.rental` and `dvd_rentals.inventory` tables
2. There will be a multiple records per unique `inventory_id` in the `dvd_rentals.rental` table
3. There will be multiple `inventory_id` records per unique `film_id` value in the `dvd_rentals.inventory` table

#### Tackling New Problems
Note that this sort of thinking will be totally different as you consider joining different tables, needless to say that each specific business problem will also be entirely different.

However - this problem solving approach is definitely transferable and I know that this exact thinking is what I have relied on over and over again throughout my career playing with data using SQL!

Next we will want to validate these assumptions with some further analysis such as simple record counts and other summary methods, many of which we covered in the first data exploration section of the Serious SQL course.

### Validating Hypotheses with Data
Let’s start using SQL to validate those 3 hypotheses.

#### Hypothesis 1

> The number of unique `inventory_id` records will be equal in both `dvd_rentals.rental` and `dvd_rentals.inventory` tables

We can quickly use distinct counts on `inventory_id` for both tables to quickly validate this assumption.

**Rental Results**

```sql
SELECT
  COUNT(DISTINCT inventory_id)
FROM `younglapalma.dvd_rentals.rental`;
```

**count: 4580**

**Inventory Results**
```sql
SELECT
  COUNT(DISTINCT inventory_id)
FROM `younglapalma.dvd_rentals.inventory`;
```

**count: 4581**

**Findings**

There seems to be 1 additional inventory_id value in the `dvd_rentals.inventory` table compared to the `dvd_rentals.rental` table

This warrants further investigation but it seems to invalidate our first hypothesis - which is exactly what we are after!

> The number of unique `inventory_id` records will be equal in both `dvd_rentals.rental` and `dvd_rentals.inventory` tables

#### Hypothesis 2

> There will be a multiple records per unique `inventory_id` in the `dvd_rentals.rental` table

A simple group by count with an additional summary group by will do to check this hypothesis.

We follow these 2 simple steps to summarise our dataset:

1. Perform a `GROUP BY` record count on the target column
2. Summarise the record count output to show the distribution of records by unique count of the target column
The target column in this case is our `inventory_id` field.

```sql
-- first generate group by counts on the target_column_values column
WITH counts_base AS (
SELECT
  inventory_id AS target_column_values,
  COUNT(*) AS row_counts
FROM `younglapalma.dvd_rentals.rental`
GROUP BY target_column_values
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT
  row_counts,
  COUNT(target_column_values) as count_of_target_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | count_of_target_values |
|------------|------------------------|
| 1          | 4                      |
| 2          | 1,126                  |
| 3          | 1,151                  |
| 4          | 1,160                  |
| 5          | 1,139                  |

**Findings**

We can indeed confirm that there are multiple rows per `inventory_id` value in our `dvd_rentals.rental` table.

**Additional Notes**

So you might be wondering - why do we use an additional group by with the CTE in the previous SQL query?

Let’s break it down quickly as it is a really fundamental technique when looking at counts for tables.

When we look at that `counts_base` CTE contents - we can see that there are going to be lots and lots of rows - as many rows as there are unique `inventory_id` values - I also throw in an `ORDER BY` in there so we show the `inventory_id` records with the most counts first.

```sql
SELECT
  inventory_id,
  COUNT(*) AS row_counts
FROM `younglapalma.dvd_rentals.rental`
GROUP BY inventory_id
ORDER BY row_counts ASC;
```

There are a total of how many rows again? (spoiler - we answered when we were looking at hypothesis 1!)

| inventory_id | row_counts |
|--------------|------------|
| 1828         | 5          |
| 811          | 5          |
| 1494         | 5          |
| 1131         | 5          |
| 4581         | 5          |
| ...          | ...        |
| 160          | 2          |
| 2662         | 1          |
| 3372         | 1          |
| 2786         | 1          |
| 1580         | 1          |

Whilst this above table is somewhat useful when inspecting the top ranking `inventory_id` values or the max number of rows that an `inventory_id` might appear - we can’t really glean much more insight from this cut of the data.

Another way to view this information would be to further aggregate this output by counting the unique `inventory_id` values whilst performing a a group by on that newly generated `row_counts` column.

This brings us back to our original CTE query with the following summarisation:

```sql
-- first generate group by counts on the target_column_values column
WITH counts_base AS (
SELECT
  inventory_id AS target_column_values,
  COUNT(*) AS row_counts
FROM `younglapalma.dvd_rentals.rental`
GROUP BY target_column_values
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT
  row_counts,
  COUNT(target_column_values) as count_of_target_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | count_of_target_values |
|------------|------------------------|
| 1          | 4                      |
| 2          | 1,126                  |
| 3          | 1,151                  |
| 4          | 1,160                  |
| 5          | 1,139                  |

#### Hypothesis 3

> There will be multiple `inventory_id` records per unique `film_id` value in the `dvd_rentals.inventory` table

We can use this same approach as hypothesis 2 but instead of groupinng on the `inventory_id` - we will instead group on the `film_id` column and perform a count distinct on the `inventory_id` before summarizing the outputs

```sql
-- first generate group by counts on the target_column_values column
WITH counts_base AS (
SELECT
  film_id AS target_column_values,
  COUNT(DISTINCT inventory_id) AS unique_record_counts
FROM `younglapalma.dvd_rentals.inventory`
GROUP BY target_column_values
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT
  unique_record_counts,
  COUNT(target_column_values) as count_of_target_values
FROM counts_base
GROUP BY unique_record_counts
ORDER BY unique_record_counts;
```

| unique_record_counts | count_of_target_values |
|----------------------|------------------------|
| 2                    | 133                    |
| 3                    | 131                    |
| 4                    | 183                    |
| 5                    | 136                    |
| 6                    | 187                    |
| 7                    | 116                    |
| 8                    | 72                     |

**Findings**

We can confirm that there are indeed multiple unique `inventory_id` per `film_id` value in the `dvd_rentals.inventory` table.

### Returning to the 2 Key Questions

As we inspect the two tables in question to validate our hunches or hypotheses about the data - we can also cover those 2 key questions that we need answers to for every table join.

Do you remember the 2 questions that we need to answer when we are joining tables together?

> 1. How many records exist per `inventory_id` value in `rental` or `inventory` tables?
> 2. How many overlapping and missing unique `foreign key` values are there between the two tables?


One of the first places to start inspecting our datasets is to look at the distribution of foreign key values in each `rental` and `inventory` table used for our join.

The distribution and relationship within the table by the foreign keys is super important because it helps us inspect what our table joining inputs consist of and also determines what sort of outputs we should expect after joining.

**`rental` distribution analysis on `inventory_id` foreign key**

```sql
-- first generate group by counts on the foreign_key_values column
WITH counts_base AS (
SELECT
  inventory_id AS foreign_key_values,
  COUNT(*) AS row_counts
FROM `younglapalma.dvd_rentals.rental`
GROUP BY foreign_key_values
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT
  row_counts,
  COUNT(foreign_key_values) as count_of_foreign_keys
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4                     |
| 2          | 1,126                 |
| 3          | 1,151                 |
| 4          | 1,160                 |
| 5          | 1,139                 |

**`inventory` distribution analysis on `inventory_id` foreign key**

```sql
WITH counts_base AS (
SELECT
  inventory_id AS foreign_key_values,
  COUNT(*) AS row_counts
FROM `younglapalma.dvd_rentals.inventory`
GROUP BY foreign_key_values
)
SELECT
  row_counts,
  COUNT(foreign_key_values) as count_of_foreign_keys
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4,581                 |

**Findings**

We can see in the `rental` table - there exists different multiple row counts for some values of the foreign keys - this is not unexpected, in fact we did exactly the same analysis previously to validate our first data hypothesis!

To break it down - we can see that 4 of our foreign key values (the `inventory_id` in this case) will have only a single row in the `rental_table`.

Additionally in the `rental` table - there are 1,126 `inventory_id` values which exist in 2 different rows and 1,151 in 3 different rows and so on.

This can also be referred to as a a 1-to-many relationship for the `inventory_id` in this `rental` table, or in other words - there may exist 1 or more record for each unique `inventory_id` value in this table.

You may remember that I used this term frequently when we covered the marketing analytics case study data overview a few tutorials back - hopefully this makes things crystal clear now!

| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4                     |
| 2          | 1,126                 |
| 3          | 1,151                 |
| 4          | 1,160                 |
| 5          | 1,139                 |

When we inspect the `dvd_rentals.inventory` table foreign key analysis using the same approach - we notice that there is only 1 row in the analysis output:

| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4,581                 |

This essentially says that for every single unique `inventory_id` value in the `inventory` table - there exists only 1 table row record - this is the exact definition of a 1-to-1 relationship (also used in the previous tutorials!)

We can indeed confirm this is the case when we simply perform the simple group by count on `inventory_id` with a descending order by to confirm that the largest row count is 1.

```sql
SELECT
  inventory_id,
  COUNT(*) as record_count
FROM `younglapalma.dvd_rentals.inventory`
GROUP BY inventory_id
ORDER BY record_count DESC
LIMIT 5;
```

| inventory_id | record_count |
|--------------|--------------|
| 1489         | 1            |
| 273          | 1            |
| 3936         | 1            |
| 2574         | 1            |
| 951          | 1            |

Now onto the second question:

> 2. How many overlapping and missing unique `foreign key` values are there between the two tables?

### Foreign Key Overlap Analysis

After we look at the distribution of foreign key values and identify the relationship between the foreign key and the rows of each left and right table, we also need to identify the unique values for the foreign key that exist in each `left` and `right` table to answer question 2:

> 2. How many overlapping and missing unique `foreign key` values are there between the two tables?

One of the simplest ways to think about this problem is using a venn diagram.

<img width="557" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/dc02658b-3b4f-46fd-b795-7dfb39e0025c">

Let’s first try to find the foreign keys which only exist in our `left` table or the `dvd_rentals.rental` table in our case.

So just a refresher we should revisit - just why do we think of this as our `left` table and not the d`vd_rentals.inventory` table?

Since the `dvd_rentals.rental` table is the only table in our ERD which has `customer_id` as a column - we would like to keep all of the records from this table. This is usually referred to as the “base” table which is commonly used as the left side of a join operation as a typical `LEFT` join keeps all of the data from the base table!

I know it sounds confusing with terms like left and right tables and left join etc - but just bear with me! These are very common terms we use all the time at work and being exposed to it early is invaluable!

#### Left & Right Only Foreign Keys

We will employ the anti join or a `WHERE NOT EXISTS` to obtain the following foreign key details:

1. Which foreign keys only exist in the left table?
2. Which foreign keys only exist in the right table?

Firstly - let’s count how many unique keys are in both (1) and (2):

```sql
-- how many foreign keys only exist in the left table and not in the right?
SELECT
  COUNT(DISTINCT rental.inventory_id)
FROM `younglapalma.dvd_rentals.rental` as rental
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.inventory` as inventory
  WHERE rental.inventory_id = inventory.inventory_id
);
```

|count|
|-----|
|0|

Great we can confirm that there are no `inventory_id` records which appear in the `dvd_rentals.rental` table which does not appear in the `dvd_rentals.inventory` table.

Now onto the right side table:

```sql
-- how many foreign keys only exist in the right table and not in the left?
-- note the table reference changes
SELECT
  COUNT(DISTINCT inventory.inventory_id)
FROM `younglapalma.dvd_rentals.inventory` as inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.rental` as rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```

|count|
|-----|
|1|

Ok - we’ve spotted a single `inventory_id` record. Let’s inspect further:

```sql
SELECT *
FROM `younglapalma.dvd_rentals.inventory` as inventory
WHERE NOT EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.rental` as rental
  WHERE rental.inventory_id = inventory.inventory_id
);
```
| inventory_id | film_id | store_id | last_update         |
|--------------|---------|----------|----------------------|
| 5            | 1       | 2        | 2006-02-15 05:09:17 |

This single record might seem off at first - but let’s revisit what the inventory data actually represents.

It is linked to a specific film record which could be rented out by a customer. One such reason for this odd record could be that this specific rental inventory unit was never rented out by a customer. In a real world problem - we would try to validate this hunch by talking with other business stakeholders or team members to confirm this is the case!

Let’s now focus on the intersection of the foreign keys for the `left` and `right` tables.

#### Joint Foreign Keys

Since we already identified that 0 of the foreign key values in the `dvd_rentals.rental` do not exist in the `dvd_rentals.inventory` table - or in other words - all of the `inventory_id` values which exist in `dvd_rentals.rental` table also exists in the `dvd_rentals.inventory` dataset - we can now redraw our venn diagram from before with a representation of what our exact data looks like.

<img width="524" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/f777a386-5c52-445c-88f6-766090240370">

We can quickly perform a left semi join or a `WHERE EXISTS` to get the count of unique foreign key values that are in the intersection.

```sql
SELECT
  COUNT(DISTINCT rental.inventory_id)
FROM `younglapalma.dvd_rentals.rental` as rental
WHERE EXISTS (
  SELECT inventory_id
  FROM `younglapalma.dvd_rentals.inventory` as inventory
  WHERE rental.inventory_id = inventory.inventory_id
);
```

|count|
|-----|
|4,580|

We also know this number from before when we were looking at the distinct counts for the dvd_rentals.rental table from above!

### Implementing the Join(s)

After performing this analysis we can conclude there is in fact no difference between running a `LEFT JOIN` or an `INNER JOIN` in our example!

We can finally implement our joins and prove this is the case by inspecting the raw row counts from the resulting join outputs.

Let’s also confirm that the unique `inventory_id` records are the same too.

```sql
DROP TABLE IF EXISTS `younglapalma.dvd_rentals.left_rental_join`;
CREATE TEMP TABLE left_rental_join (customer_id INT64, inventory_id INT64, film_id INT64) 
AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM `younglapalma.dvd_rentals.rental` AS rental
LEFT JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id;

DROP TABLE IF EXISTS `younglapalma.dvd_rentals.inner_rental_join`;
CREATE TEMP TABLE inner_rental_join (customer_id INT64, inventory_id INT64, film_id INT64) AS
SELECT
  rental.customer_id,
  rental.inventory_id,
  inventory.film_id
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id;

-- check the counts for each output (bonus UNION usage)
-- note that these parantheses are not really required but it makes
-- the code look and read a bit nicer!
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT inventory_id) AS unique_key_values
  FROM left_rental_join
)
UNION DISTINCT
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

Great! In this example - we can indeed confirm that there is no difference between an inner join or left join for our datasets!

In practice however - running through all of these steps at this fine level of granularity is required to perfect table joining technique.

Over time you will develop strong intuition and likely you will skip a few of these steps after inspecting the data by hand - and this is totally fine!

Think of this as similar to learning classical music before trying to perform free-form jazz music - sometimes it pays to learn it all by the book before you go off and improvise!

###  In Summary

We have now successfully answered all 3 questions for our join - did you forget what the 3 questions were already?

> 1. What is the purpose of joining these two tables?

We need to keep all of the customer rental records from `dvd_rentals.rental` and match up each record with its equivalent `film_id` value from the `dvd_rentals.inventory` table.

> 2. What is the distribution of foreign keys within each table?

There is a 1-to-many relationship between the `inventory_id` and the rows of the `dvd_rentals.rental` table

| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4                     |
| 2          | 1,126                 |
| 3          | 1,151                 |
| 4          | 1,160                 |
| 5          | 1,139                 |

There is a 1-to-1 relationship between the `inventory_id` and the rows of the `dvd_rentals.inventory` table

| row_counts | count_of_foreign_keys |
|------------|-----------------------|
| 1          | 4,581                 |

> 3. How many unique foreign key values exist in each table?

All of the foreign key values in `dvd_rentals.rental` exist in `dvd_rentals.inventory` and only 1 record `inventory_id = 5` exists only in the `dvd_rentals.inventory` table.

There is an overlap of 4,580 unique inventory_id foreign key values which will exist after the join is complete.

# Returning to our Data Journey

Now that we’ve covered part 1 of our table join - yes I know - that was only part 1 - let’s revisit our table join journey again - I hope you haven’t forgotten already 😅

| Join Journey Part | Start         | End           | Foreign Key   |
|-------------------|---------------|---------------|---------------|
| Part 1            | `rental`        | `inventory`     | `inventory_id`  |
| Part 2            | `inventory`     | `film`          | `film_id`       |
| Part 3            | `film`          | `film_category` | `film_id`       |
| Part 4            | `film_category` | `category`      | `category_id`   |

Let’s also remind ourselves of where these parts come in via our mapped ERD in gif form below.

![downloadgif](https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/ceb3559f-fc0b-43a2-a73e-6b0b55a20ec9)

For the following sections - we will go through that same table joining checklist that we covered for part 1 but we will be moving at a more rapid rate as a lot of the steps will be repeated.

## Table Joining Checklist

1. What is the purpose of joining these two tables?

    a. What contextual hypotheses do we have about the data?

    b. How can we validate these assumptions?

2. What is the distribution of foreign keys within each table?

3. How many unique foreign key values exist in each table?

## Joins Part 2

For part 2 of our data joining journey we need to combine our outputs from part 1 and continue joining.

However when we are investigating the joining components - it is often best to deal with each table join component separately before combining all the joins.

Let’s perform the checklist steps for just the `dvd_rentals.inventory` and `dvd_rentals.film` tables.

> 1. What is the purpose of joining these two tables?

We want to match the films on film_id to obtain the title of each film.

> a. What contextual hypotheses do we have about the data?

There be 1-to-many relationship for `film_id` and the rows of the `dvd_rentals.inventory` table as one specific film might have multiple copies to be purchased at the rental store.

There should be 1-to-1 relationship for `film_id` and the rows of the `dvd_rentals.film` table as it doesn’t make sense for there to be duplicates in this `dvd_rentals.film` - yes that’s actually a legitimate reason to generate a hypothesis - we use this argument a lot when we are dealing with data problems!

The saying is - if something doesn’t make sense - you should test it, preferably with data!

> b. How can we validate these assumptions?

Generate the row counts for `film_id` for both the `dvd_rentals.inventory` and `dvd_rentals.film` tables - note that this also helps us with part 2 of the table joining checklist!

Let’s use a summarized group by with a CTE for the `dvd_rentals.inventory` table:

```sql
WITH base_counts AS (
SELECT
  film_id,
  COUNT(*) AS record_count
FROM `younglapalma.dvd_rentals.inventory`
GROUP BY film_id
)
SELECT
  record_count,
  COUNT(DISTINCT film_id) as unique_film_id_values
FROM base_counts
GROUP BY record_count
ORDER BY record_count;
```

| record_count | unique_film_id_values |
|--------------|-----------------------|
| 2            | 133                   |
| 3            | 131                   |
| 4            | 183                   |
| 5            | 136                   |
| 6            | 187                   |
| 7            | 116                   |
| 8            | 72                    |

Confirmed - we have a 1-to-many relationship for the `film_id` foreign key in our `dvd_rentals.inventory_table`

Now We just need to confirm that there are indeed only single records for the `dvd_rentals.film` table so we can use this query with a `LIMIT` as long as we are ordering by the `record_count` descending - if you see a number larger than 1 for the `record_count` field - it means that you have conclusive evidence that the table is not a 1-to-1 table!

```sql
SELECT
  film_id,
  COUNT(*) AS record_count
FROM `younglapalma.dvd_rentals.film`
GROUP BY film_id
ORDER BY record_count DESC
LIMIT 5;
```
| film_id | record_count |
|---------|--------------|
| 273     | 1            |
| 51      | 1            |
| 951     | 1            |
| 839     | 1            |
| 652     | 1            |


We can now also confirm that there is a 1-to-1 relationship in the dvd_rentals.film

> 2. What is the distribution of foreign keys within each table?

We already did this with our previouis summarized group by count when confirming our hypotheses for both tables.

> 3. How many unique foreign key values exist in each table?

We will use our same anti join approach to figure out the overlaps and exlusions as per the venn diagram previously.

```sql
-- how many foreign keys only exist in the inventory table
SELECT
  COUNT(DISTINCT inventory.film_id)
FROM `younglapalma.dvd_rentals.inventory` as inventory
WHERE NOT EXISTS (
  SELECT film_id
  FROM `younglapalma.dvd_rentals.film` as film
  WHERE film.film_id = inventory.film_id
);
```

**count: 0**

Ok great - we can conclude that all of the `film_id` records from the `dvd_rentals.inventory` table exists in the `dvd_rentals.film` table.

Let’s check the other side now.

```sql
-- how many foreign keys only exist in the film table
SELECT
  COUNT(DISTINCT film.film_id)
FROM `younglapalma.dvd_rentals.film` as film
WHERE NOT EXISTS (
  SELECT film_id
  FROM `younglapalma.dvd_rentals.inventory` as inventory
  WHERE film.film_id = inventory.film_id
);
```

**count: 42**

Alright we have a discrepancy between the foreign key values - here we see a much larger count of keys which exist in the dvd_rentals.film table than in the dvd_rentals.inventory table.

Finally - let’s check that total count of distinct foreign key values that will be generated when we use a left semi join on our dvd_rentals.inventory as our base left table.

```sql
SELECT
  COUNT(DISTINCT film_id)
FROM `younglapalma.dvd_rentals.inventory` as inventory
-- note how the NOT is no longer here for a left semi join
-- compared to the anti join!
WHERE EXISTS (
  SELECT film_id
  FROM `younglapalma.dvd_rentals.film` as film
  WHERE film.film_id = inventory.film_id
);
```

**count: 958**

We will be expecting a total distinct count of film_id values of 958 once we perform the final join between our 2 tables.

### Join Implementation

Now that we know that we have all of the key values from the base table, our dvd_rentals.inventory table - we now need to do our join.

In our example - we will use an INNER JOIN but note that for this specific task - there is no difference between left or right joins - we will confirm that this is the case with some more summaries.

```sql
CREATE TEMP TABLE left_join_part_2 (inventory_id INT64, film_id INT64, title STRING) 
AS
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM `younglapalma.dvd_rentals.inventory` AS inventory
LEFT JOIN `younglapalma.dvd_rentals.film` AS film
  ON film.film_id = inventory.film_id;


CREATE TEMP TABLE inner_join_part_2 (inventory_id INT64, film_id INT64, title STRING) AS
SELECT
  inventory.inventory_id,
  inventory.film_id,
  film.title
FROM `younglapalma.dvd_rentals.inventory` AS inventory
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON film.film_id = inventory.film_id;

-- check the counts for each output (bonus UNION usage)
-- note that these parantheses are not really required but it makes
-- the code look and read a bit nicer!
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM left_join_part_2
)
UNION DISTINCT
(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM inner_join_part_2
);
```

| join_type  | record_count | unique_key_values |
|------------|--------------|-------------------|
| inner join | 4581         | 958               |
| left join  | 4581         | 958               |

### Joining Part 1 & 2

Now that we’ve done part 1 and 2 for the joining journey - let’s demonstrate how to implement a 3 way join.

I could step through this at a very fine level but it would do more harm than good so we will just jump straight into the code from here:

Note that we only extract the data points which are used for our analysis and nothing else - you can feel free to select more columns if you need.

```sql
CREATE TEMP TABLE join_parts_1_and_2 (customer_id INT64, film_id INT64, title STRING) AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title
FROM `younglapalma.dvd_rentals.rental` AS rental
INNER JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id;

SELECT * FROM join_parts_1_and_2 limit 10;
```

| customer_id | film_id | title          |
|-------------|---------|----------------|
| 130         | 80      | BLANKET BEVERLY|
| 459         | 333     | FREAKY POCUS   |
| 408         | 373     | GRADUATE LORD  |
| 333         | 535     | LOVE SUICIDES  |
| 222         | 450     | IDOLS SNATCHERS|
| 549         | 613     | MYSTIC TRUMAN  |
| 269         | 870     | SWARM GOLD     |
| 239         | 510     | LAWLESS VISION |
| 126         | 565     | MATRIX SNOWMAN |
| 399         | 396     | HANGING DEEP   |

## Join Part 3 & 4

I would repeat the process with parts 3 and 4 but this tutorial would get a bit monotonous so instead I will implement the final complete join to obtain our base dataset.

The most important thing when gauging the types of joins you should use is that relationship between the foreign keys and the target tables.

Once you make your assumptions and hypotheses about the tables based off your contextual knowledge of what the data represents, the rest should fall into place and over time you will naturally approach future problems with the same mindset, and you’ll get faster and faster at this!

We can make the assumptions (and also confirm them!) that there is a 1-to-1 relationship for film_id in both left and right tables dvd_rentals.film and dvd_rentals.film_category forpart 3 of our table join journey.

For part 4 - we can do the same for the 1-to-many relationship between category_id and the left dvd_rentals.film_category and a 1-to-1 relationship for the dvd_rentals.category table.

After validating all of these assumptions - we confirmed that there will be no excessive lost data when we simply perform INNER JOIN for all of the parts in our table joining journey as shown:

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  film_category.category_id,
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

| customer_id | film_id | title          | category_id | category |
|-------------|---------|----------------|-------------|----------|
| 130         | 80      | BLANKET BEVERLY| 8           | Family   |
| 459         | 333     | FREAKY POCUS   | 12          | Music    |
| 408         | 373     | GRADUATE LORD  | 3           | Children |
| 333         | 535     | LOVE SUICIDES  | 11          | Horror   |
| 222         | 450     | IDOLS SNATCHERS| 3           | Children |
| 549         | 613     | MYSTIC TRUMAN  | 5           | Comedy   |
| 269         | 870     | SWARM GOLD     | 11          | Horror   |
| 239         | 510     | LAWLESS VISION | 2           | Animation|
| 126         | 565     | MATRIX SNOWMAN | 9           | Foreign  |
| 399         | 396     | HANGING DEEP   | 7           | Drama    |

Yes - we have finally arrived to the base dataset which we need to calculate some of our aggregations!

But there is more - you might thinking - what happens if I actually used a left join for all of these tables instead of an inner join?

Well - let’s see what the counts look like to compare to the inner join option:

```sql
CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  film_category.category_id,
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

CREATE TEMP TABLE complete_left_join_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  category.name AS category_name
FROM `younglapalma.dvd_rentals.rental` AS rental
LEFT JOIN `younglapalma.dvd_rentals.inventory` AS inventory
  ON rental.inventory_id = inventory.inventory_id
LEFT JOIN `younglapalma.dvd_rentals.film` AS film
  ON inventory.film_id = film.film_id
LEFT JOIN `younglapalma.dvd_rentals.film_category` AS film_category
  ON film.film_id = film_category.film_id
LEFT JOIN `younglapalma.dvd_rentals.category` AS category
  ON film_category.category_id = category.category_id;

(SELECT
  'left join' AS join_type,
  COUNT(*) AS final_record_count
FROM complete_left_join_dataset)
UNION DISTINCT
(SELECT
  'inner join' AS join_type,
  COUNT(*) AS final_record_count
FROM complete_joint_dataset);
```

| join_type   | final_record_count |
|-------------|--------------------|
| inner join  | 16044              |
| left join   | 16044              |

As we can see - in our example - there is no difference! But why???

We can see from the order of the tables that we joined and their respective relationships with the foreign keys of the downstream tables.

First we took the dvd_rentals.rental table and performed a join onto the dvd_rentals.inventory dataset.

We know that all of the foreign keys from the base table dvd_rentals.rental also exist in the right join table dvd_rentals.inventory.

This means that we should expect all of the rows from the dvd_rentals.rental table to remain as is - just with the additional film_id column being populated by the dvd_rentals.inventory table.

Next when we inspect the following left join onto the dvd_rentals.film table - we can see that this table is joined onto the dvd_rentals.inventory table - since we also know that all of the inventory_id values from the dvd_rentals.inventory exist in the dvd_rentals.film table - we can also expect all of the records to remain as is with only the additional title field being added to the resulting dataset.

Finally we do the same for the category_id and name columns from the dvd_rentals.film_category and dvd_rentals.category tables respectively.

This might seem like a redundant fact - but it is really important to understand this additive approach to multiple table joins for SQL.

We can think of this entire process as this chunking up of columns to obtain what we need via joins:

<img width="574" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/d76a5891-f3e9-4c6b-83d9-e53c4dee567c">

# Conclusion

Ok - so this multiple table joining tutorial is very long, so let’s end things here!

To summarise what we covered in this session:

1. More reverse engineering to identify the key columns we needed in our SQL output for our customer insight and recommendation calculations
2. Mapped out our table journey by linking the tables with our key columns via the foreign keys in the ERD
3. Introduced a framework for analyzing table joining operations
4. Implemented table joins between two tables, comparing expected outputs for LEFT JOIN vs INNER JOIN operations
5. Joined multiple tables in a single query

The framework for analyzing table joins:

1. What is the purpose of joining these two tables?

    a. What contextual hypotheses do we have about the data?

    b. How can we validate these assumptions?

2. What is the distribution of foreign keys within each table?

3. How many unique foreign key values exist in each table?

Performing multiple table joins is very very common and hopefully this framework and approach will help you when you need to extract data from multiple tables!

In the next tutorial we will start to explore what we can do with our base dataset in the following tutorial.

# Exercise

You might have noticed that there is also the dvd_rentals.film_actor and dvd_rentals.actor in our ERD that we didn’t cover in this tutorial.

There is a particular reason for avoiding these tables - can you figure out why? What impact would joining these additional tables to our existing finalized query between the other 5 tables result in? Also bonus question for fun - how many rows would appear if you literally just inner joined every single table by the relevant foreign keys?

Please have a go at analyzing the required table joins using our framework above to inspect the relationships for film_id and actor_id in the dvd_rentals.film_actor table.

# Appendix

A few additional thoughts and points about multiple table joins and EFFICIENCY.

Often when we are dealing with table joins - it is SUPER important to ensure that you are joining on index or primary key columns wherever possible.

The SQL optimizer works best with indexed columns and is the most efficiently during the matching phase of the joins (that ON condition) - outside of smaller tables where a sequential scan is just as fast if not faster than keeping a separate index locked in!

Some tips when working with this stuff in reality (especially with HUGE tables) is to create TEMP tables with only the required columns from a table, applying WHERE filters where relevant and also making sure to create a PRIMARY KEY or INDEX on the columns which you will be joining on in downstream sections of your script.

In PostgreSQL - you need to also run an ANALYZE TABLE step before using the temporary table in future joins otherwise the table statistics and indexes will not be collected for use by the optimizer!

We will cover this in more detail in the additional SQL component of this course but it’s definitely a very important point worth noting and keeping in mind when using SQL with huge datasets in practice!