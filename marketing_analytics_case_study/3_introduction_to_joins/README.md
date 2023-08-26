# Introduction To Table Joins

I remember the first time I was exposed to table joins and I spent an enormous amount of time trying to wrap my head around what was going on.

This tutorial will be my attempt to simplify all the things I wish I was shown when I first encountered joins - I really hope that this will help you!

Table joins help us access rows from multiple datasets in a single query - did you know that they are actually implementing low level relational algebra transformations?

Luckily for us, all that crazy math is abstracted away and we just need to understand the different types of joins we can use to combine different datasets.

In this following gentle introduction we will use 2 simple datasets to experiment and explore different types of joins. I’ve also created some visual representations of each type of join to help simplify the learning process as much as possible.

# Sample Datasets

For these following join examples let’s quickly generate some data about some popular influencers in the data world to demonstrate all of the various types of table joins we can perform.

Bonus points if you can remember where these “titles” originated from - let me know on LinkedIn by sharing a screenshot from this course and tagging me and I’ll be sure to respond to you!

**names**

| iid | first_name | title                 |
|-----|------------|-----------------------|
| 1   | Kate       | Datacated Visualizer  |
| 2   | Eric       | Captain SQL           |
| 3   | Danny      | Data Wizard Of Oz     |
| 4   | Ben        | Mad Scientist         |
| 5   | Dave       | Analytics Heretic     |
| 6   | Ken        | The YouTuber          |

**jobs**

| iid | occupation | salary     |
|-----|------------|------------|
| 1   | Cleaner    | High       |
| 2   | Janitor    | Medium     |
| 3   | Monkey     | Low        |
| 6   | Plumber    | Ultra      |
| 7   | Hero       | Plus Ultra |

You may also notice that the first column is not `id` but in fact it’s `iid` - you can think of it as standing for “Influencer ID” if you like!

I did this because I realised that the code highlighting for `id` didn’t look nice because it’s actually a SQL keyword - which you should technically avoid when naming columns in new tables or when you’re renaming columns using aliases!

## Create Sample Tables In SQL

You’ve seen this before earlier in the data exploration but here we see another `CREATE TEMP TABLE` statement combined with a CTE (it has so many uses!)

You can copy and paste the below queries directly into your SQLPad GUI and run them together, just note that you won’t see any outputs after running a `CREATE` statement - but you can check that the temporary tables do indeed exist by running a `SELECT *` on the newly created tables.

```sql
-- CREATE CTE
WITH input_data AS (
  SELECT *
  FROM
  UNNEST([STRUCT(1 AS iid, 'Kate' as first_name, 'Datacated Visualizer' as title),
 (2, 'Eric', 'Captain SQL'),
 (3, 'Danny', 'Data Wizard Of Oz'),
 (4, 'Ben', 'Mad Scientist'),
 (5, 'Dave', 'Analytics Heretic'),
 (6, 'Ken', 'The YouTuber')])
 )

SELECT * FROM input_data;
-- CREATE TEMP TABLE (NEED TO CREATE DATASET `temp` FIRST)
DROP TABLE IF EXISTS `younglapalma.temp.name`;
CREATE TABLE `younglapalma.temp.name` AS
SELECT *
FROM UNNEST([
    STRUCT(1 AS iid, 'Kate' AS first_name, 'Datacated Visualizer' AS title),
    (2, 'Eric', 'Captain SQL'),
    (3, 'Danny', 'Data Wizard Of Oz'),
    (4, 'Ben', 'Mad Scientist'),
    (5, 'Dave', 'Analytics Heretic'),
    (6, 'Ken', 'The YouTuber')
]);

SELECT * from `younglapalma.temp.name`;

-- CREATE TEMP TABLE
DROP TABLE IF EXISTS `younglapalma.temp.jobs`;
CREATE TABLE `younglapalma.temp.jobs` AS
SELECT *
FROM UNNEST([
    STRUCT(1 AS iid, 'Cleaner' AS occupation, 'High' AS salary),
 (2, 'Janitor', 'Medium'),
 (3, 'Monkey', 'Low'),
 (6, 'Plumber', 'Ultra'),
 (7, 'Hero', 'Plus Ultra')
]);
SELECT * from `younglapalma.temp.jobs`;
```

Inspecting the `names` table

```sql
SELECT * FROM `younglapalma.temp.name`
```

| iid | first_name | title                 |
|-----|------------|-----------------------|
| 1   | Kate       | Datacated Visualizer  |
| 2   | Eric       | Captain SQL           |
| 3   | Danny      | Data Wizard Of Oz     |
| 4   | Ben        | Mad Scientist         |
| 5   | Dave       | Analytics Heretic     |
| 6   | Ken        | The YouTuber          |

Inspecting the `jobs` table

```sql
SELECT * FROM `younglapalma.temp.jobs`
```

| iid | occupation | salary     |
|-----|------------|------------|
| 1   | Cleaner    | High       |
| 2   | Janitor    | Medium     |
| 3   | Monkey     | Low        |
| 6   | Plumber    | Ultra      |
| 7   | Hero       | Plus Ultra |

# Basic Table Joins

This section of the tutorial will cover the most common types of joins you will likely encounter on your SQL journey.

These include:

- `INNER JOIN` or `JOIN`
- `LEFT JOIN` or `LEFT OUTER JOIN`
- `FULL JOIN` or `FULL OUTER JOIN`

We also the various types of conditions we can use for the `ON` clause for each of our joins.

## The Inner Join

<img width="543" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/27db847f-097e-4435-a907-5f103fbd10e5">

An `INNER JOIN` or `JOIN` is used to get the intersection between two tables and return only the matching records.

In the following example - we join `names` and `jobs` on the `iid` column to get first names and occupation columns in the same query.

```sql
SELECT name.iid, name.first_name, name.title, jobs.occupation, jobs.salary 
FROM `younglapalma.temp.name` AS name
INNER JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| iid | first_name | title             | occupation | salary     |
|-----|------------|-------------------|------------|------------|
| 1   | Kate       | Datacated Visualizer | Cleaner    | High       |
| 2   | Eric       | Captain SQL       | Janitor    | Medium     |
| 3   | Danny      | Data Wizard Of Oz | Monkey     | Low        |
| 6   | Ken        | The YouTuber      | Plumber    | Ultra      |

Notice how only 4 records remain once we perform the `INNER JOIN` - this is essentially performing an intersection of the two datasets which meets the condition inside the ON clause.

When we inspect our original `names` and `jobs` tables - we can see that there are some mismatches between the missing `iid` records in the `jobs` table

In this case our `ON` clause or join condition is that the `iid` column is equal in both the `names` and the `jobs` table.

There is more information about join conditions in the (appendix)[#appendix] which goes into more depth about the different types of conditions we can use to match our table records.

## Table & Column References

Why did I have to list out all of the columns in the `SELECT` statement for the inner join example above?

The best way to answer this question is to demonstrate what happens when we won’t list out the columns and table references!

Let’s try to run a few SQL variations without these references just to see what happens and find the answer ourselves.

Firstly - let’s try an example which just has `SELECT *` without any table reference at all:

### SELECT *

```sql
SELECT *
FROM `younglapalma.temp.name` AS name
INNER JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| iid | first_name | title             | iid_1 | occupation | salary     |
|-----|------------|-------------------|-------|------------|------------|
| 1   | Kate       | Datacated Visualizer | 1     | Cleaner    | High       |
| 2   | Eric       | Captain SQL       | 2     | Janitor    | Medium     |
| 3   | Danny      | Data Wizard Of Oz | 3     | Monkey     | Low        |
| 6   | Ken        | The YouTuber      | 6     | Plumber    | Ultra      |

Do you realise that something doesn’t seem quite right…why are there 2 `iid` columns?

### SELECT table.*

Let’s try another example with `names.*` and `jobs.*` to tell the SQL engine that we want all the columns from both tables in the query:

```sql
SELECT name.*, jobs.*
FROM `younglapalma.temp.name` AS name
INNER JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| iid | first_name | title             | iid_1 | occupation | salary     |
|-----|------------|-------------------|-------|------------|------------|
| 1   | Kate       | Datacated Visualizer | 1     | Cleaner    | High       |
| 2   | Eric       | Captain SQL       | 2     | Janitor    | Medium     |
| 3   | Danny      | Data Wizard Of Oz | 3     | Monkey     | Low        |
| 6   | Ken        | The YouTuber      | 6     | Plumber    | Ultra      |

As you may have noticed already - both queries return a duplicated `iid` column in the output - which honestly *might* be fine for some people…

In my mind - having these duplicated columns is not really great so let’s try to figure out how we should get rid of them shall we?

### No Table References

For this test - let’s just select the columns from our original inner join example without specifying which table reference to select it from?

```sql
SELECT 
  iid,
  first_name,
  title,
  occupation,
  salary
FROM `younglapalma.temp.name` AS name
INNER JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```
When we run this query - it actually returns the following error:
`Column name iid is ambiguous at [505:3]`
So unfortunately this is not going to cut it!

### Additional Thoughts

Honestly this bad habit or poor practice of skipping the table reference in front of columns or using a `SELECT table.*` without specifying exact table columns in any SQL join query is a real pet peeve of mine!

When reading other people’s code - it is actually really difficult to track down just exactly where specific output columns come from, especially when there are lots of columns from multiple table joins!

To make things worse - when you omit the table references, it also makes it really difficult for you to debug your own table joins and having explicit column references can save you lots of time and help you deliver your work faster.

These table references or aliases in front of the `SELECT` columns in any SQL query are critical and I implore you to always take the time and care to always add them into your queries!

It is one of the more subtle traits of highly effective SQL developers and data professionals - so be sure to focus on these little things to really improve the quality of your code.

Ok enough preaching - let’s move onto our next basic join: the `LEFT JOIN`

# Left Join

<img width="546" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/83ef9f5e-da12-43b4-b43f-7df5ca835873">

The `LEFT JOIN` or `LEFT OUTER JOIN` is our second super common basic table join and it is used when you want to keep all the rows or records from the left table and return any matching records from the right table.

Note that these two terms are exactly the same but usually in practice I’ve seen it most commonly used as simply a `LEFT JOIN` but it is something to be aware of!

One of the fundamental concepts for this type of join is that the table on the left is also known as the “base” table where all of the rows are retained.

The “target” or right hand side table for the join will only return values when there is a match with records from the “base” table. When there is no match - the values for the specific “target” columns will be `null` for the non-matching rows.

In this following example, we will perform a left join to keep all the records from our base table: `names` and then add on any matching records in the target `jobs` table.

### Basic Left Join Implementation

```sql
SELECT 
  name.iid, 
  name.first_name, 
  name.title, 
  jobs.occupation, 
  jobs.salary 
FROM `younglapalma.temp.name` AS name
LEFT JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| iid | first_name | title             | occupation | salary     |
|-----|------------|-------------------|------------|------------|
| 1   | Kate       | Datacated Visualizer | Cleaner    | High       |
| 2   | Eric       | Captain SQL       | Janitor    | Medium     |
| 3   | Danny      | Data Wizard Of Oz | Monkey     | Low        |
| 4   | Ben        | Mad Scientist     | null       | null       |
| 5   | Dave       | Analytics Heretic | null       | null       |
| 6   | Ken        | The YouTuber      | Plumber    | Ultra      |

Note how there are `null` values for the `occupation` and `salary` columns for Ben and Dave with respective `iid` values of 4, 5.

Since their `iid` values do not exist in the `jobs` table - when we perform a left join `jobs` onto the base table `names` - there are `null` records for these 2 non-matching rows for both the `occupation` and `salary` columns which originate from the `jobs` table.

## Left Join Table Order
So you might be wondering - what happens when we swap the left and right tables, now jobs is on the left hand side as the “base” table and names is now the “target” table on the right?

```sql
SELECT 
  jobs.iid,
  jobs.occupation, 
  jobs.salary,
  name.first_name, 
  name.title 
FROM `younglapalma.temp.jobs` AS jobs
LEFT JOIN `younglapalma.temp.name` AS name
ON jobs.iid = name.iid
ORDER BY jobs.iid ASC;
```
| iid | occupation | salary      | first_name | title                |
|-----|------------|-------------|------------|----------------------|
| 1   | Cleaner    | High        | Kate       | Datacated Visualizer |
| 2   | Janitor    | Medium      | Eric       | Captain SQL          |
| 3   | Monkey     | Low         | Danny      | Data Wizard Of Oz    |
| 6   | Plumber    | Ultra       | Ken        | The YouTuber         |
| 7   | Hero       | Plus Ultra  | null       | null                 |

Notice how the rows of the resulting output has switched to match the new “base” `jobs` table when we swap the order of the tables as they appear in the left join.

I hope this example makes it ultra clear just how important it is to think about what “base” table you want to have when performing a left join.

## Inner Join Table Order

Another question that you might have could be - does this same ordering of tables matter for an for an `INNER JOIN` - in short the answer is no, there is actually no difference.

Let’s confirm this is definitely the case with our previous inner join example again.

The original inner join query looked like this with the following output:

```sql
SELECT 
  name.iid,
  name.first_name, 
  name.title,
  jobs.occupation, 
  jobs.salary  
FROM `younglapalma.temp.name` AS name
LEFT JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| Customer ID | First Name | Title                 | Occupation | Salary |
|-------------|------------|-----------------------|------------|--------|
| 1           | Kate       | Datacated Visualizer  | Cleaner    | High   |
| 2           | Eric       | Captain SQL           | Janitor    | Medium |
| 3           | Danny      | Data Wizard Of Oz    | Monkey     | Low    |
| 6           | Ken        | The YouTuber          | Plumber    | Ultra  |


Swapping the order of tables in the join so that `names` is now inner joined to `jobs`


```sql
SELECT 
  name.iid,
  name.first_name, 
  name.title,
  jobs.occupation, 
  jobs.salary  
FROM `younglapalma.temp.jobs` AS jobs
LEFT JOIN `younglapalma.temp.name` AS name
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```

| Customer ID | First Name | Title                 | Occupation | Salary |
|-------------|------------|-----------------------|------------|--------|
| 1           | Kate       | Datacated Visualizer  | Cleaner    | High   |
| 2           | Eric       | Captain SQL           | Janitor    | Medium |
| 3           | Danny      | Data Wizard Of Oz    | Monkey     | Low    |
| 6           | Ken        | The YouTuber          | Plumber    | Ultra  |

Final question I sometimes get from people new in the SQL world is about the order of the `ON` clause conditions.

What happens when we swap the order of the `names.iid` and `jobs.iid` inputs in the `ON` clause for the join condition?

```sql
SELECT 
  name.iid,
  name.first_name, 
  name.title,
  jobs.occupation, 
  jobs.salary  
FROM `younglapalma.temp.jobs` AS jobs
LEFT JOIN `younglapalma.temp.name` AS name
ON jobs.iid = name.iid
ORDER BY name.iid ASC;
```

| Customer ID | First Name | Title                 | Occupation | Salary |
|-------------|------------|-----------------------|------------|--------|
| 1           | Kate       | Datacated Visualizer  | Cleaner    | High   |
| 2           | Eric       | Captain SQL           | Janitor    | Medium |
| 3           | Danny      | Data Wizard Of Oz    | Monkey     | Low    |
| 6           | Ken        | The YouTuber          | Plumber    | Ultra  |

Here we see that there is indeed no difference between all of these inner join implementations.

Just don’t mix this up for left joins and you should be good to go!

## Full Join or Full Outer Join

<img width="564" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/68085288-19be-47a5-932a-cc351e2e1b54">

Sometimes there are occasions when you need to get the full combination of both tables to include all the rows and columns.

Sometimes you might also want the same behaviour as the previous left join but have both tables used as a “base” table where all the rows are kept from both tables.

This is where the full join or full outer join comes into play - one thing to note is that they mean exactly the same thing, similar to how the join and inner join are the same.

This is what the output looks like for a full join between `names` and `jobs` - notice the additional alias applied to `names.iid` and `jobs.iid` as well as the relationships between their raw values in the resulting output below:

```sql
SELECT 
  name.iid as name_id,
  jobs.iid as jobs_id,
  name.first_name, 
  name.title,
  jobs.occupation, 
  jobs.salary  
FROM `younglapalma.temp.name` AS name
FULL JOIN `younglapalma.temp.jobs` AS jobs
ON name.iid = jobs.iid
ORDER BY name.iid ASC;
```
| name_id | job_id | first_name | title                | occupation | salary      |
|---------|--------|------------|----------------------|------------|-------------|
| 1       | 1      | Kate       | Datacated Visualizer | Cleaner    | High        |
| 2       | 2      | Eric       | Captain SQL          | Janitor    | Medium      |
| 3       | 3      | Danny      | Data Wizard Of Oz   | Monkey     | Low         |
| 4       | null   | Ben        | Mad Scientist        | null       | null        |
| 5       | null   | Dave       | Analytics Heretic    | null       | null        |
| 6       | 6      | Ken        | The YouTuber         | Plumber    | Ultra       |
| null    | 7      | null       | null                 | Hero       | Plus Ultra  |

In the output above we can see how there are `null` values present in both the `name_id` and the `job_id` columns.

This behaviour mimics the same retention of rows in the `LEFT JOIN` we saw previously - but both sets of table rows are kept regardless whether they match or not with the adjoining table.

## Table Aliases For Joins

Let’s now introduce aliases into our table joining toolkit to make our SQL queries slightly less verbose.

These aliases are exactly the same as when we used aliases to rename our columns but this time we can do it with entire tables (as well as subqueries and CTEs too!)

There are a few different competing theories about how we should properly name our table aliases - so I’ll share with you a few different variations to help you decide which method you prefer most!

A word of caution though - in practice I recommend you work closely with your team members to agree upon a standard naming convention or practice so all your team’s code will look perfect and uniform.

It’s always a good discussion to have with your coworkers about coding standards as clean code is just like keeping your room clean - not everyone does it but it sure looks nice!

Ok before I guilt trip you further into keeping your room and your code clean - let’s jump into our various alias strategies!

### First Letter Aliasing

Sometimes we’ll be recommended to use the first letter to alias tables - however I’ve found that this gets rather interesting when you have multiple tables that start with the same letter as you cannot assign the same alias more than once.

```sql
SELECT 
  n.iid as name_id,
  j.iid as jobs_id,
  n.first_name, 
  n.title,
  j.occupation, 
  j.salary  
FROM `younglapalma.temp.name` AS n
FULL JOIN `younglapalma.temp.jobs` AS j
ON n.iid = j.iid
ORDER BY n.iid ASC;
```

Let’s imagine that I have now aliased both names and jobs as `n` for fun to see what happens:

```sql
SELECT 
  n.iid as name_id,
  n.iid as jobs_id,
  n.first_name, 
  n.title,
  n.occupation, 
  n.salary  
FROM `younglapalma.temp.name` AS n
FULL JOIN `younglapalma.temp.jobs` AS n
ON n.iid = n.iid
ORDER BY n.iid ASC;
```

Running this query returns an error: `table name "n" specified more than once`

More forms of first letter aliasing looks like this for tables with underscores in their names - which is super common when working with enterprise databases!

So imagine we have a table called `customer_sales_daily` - the first letter theory means that we should alias this table using `customer_sales_daily AS csd`

Another example when dealing with schemas that are included in the table reference might include `financial.customer_sales_daily AS fcsd`

As you can see this starts getting really really confusing once you have massive table names - especially when you need to include all of their references in the `SELECT` expressions like in the example query below:

```sql
SELECT
  fcsd.user_id,
  fcnd.first_name,
  fcnd,last_name
FROM financial.customer_sales_daily AS fcsd
INNER JOIN financial.customer_name_details AS fcnd
  ON fcsd.customer_id = fcnd.customer_id
```

So is there another better way?

### Logical Aliasing

Another school of thought is to use logical names to alias tables to make it easier for the person reading the code to comprehend what’s going on. In general - this is usually a great idea although it requires a bit of mental tracking to remember which table is linked to the logical name.

In our original example with the names and jobs - as the table names are straightforward we would probably just use the same names - however for our made up financial database example - we might want to use something like this instead:

```sql
SELECT
  sales.user_id,
  names.first_name,
  names.last_name
FROM financial.customer_sales_daily AS sales
INNER JOIN financial.customer_name_details AS names
  ON sales.customer_id = names.customer_id
  ```

Finally another option exists…

### Appearance Order Aliasing

Essentially - this method is to just pop an arbitrary letter of t in front of each table that is aliased and then just to number the tables that appear in order from 1 to the number of tables.

I’m actually not sure if there is an official recommendation for this type of aliasing - I can only share that I’ve used this many many times and I’ve found that this is often the most “logical” method for me and my mentors - especially when we had to deal with massive queries that would join 20+ tables!!!

The logic behind this aliasing is essentially a trade off between keeping an ordered list of tables whilst helping you manage the logic between joins of various tables. I’ve found that keeping the numbers only and ignoring the names of tables and their beginning letters helps me to read through code quickly as I can easily identify which table certain columns originate from and exactly where they were introduced based off the join ordering of tables.

For our example - here is what it would look like:

```sql
SELECT
  t1.iid AS name_iid,
  t2.iid AS job_iid,
  t1.first_name,
  t1.title,
  t2.occupation,
  t2.salary
FROM names AS t1
FULL JOIN jobs AS t2
  ON t1.iid = t2.iid;
```

Again all of this aliasing is not something that you need to FOCUS on so to speak - but rather sticking with a standardised aliasing strategy is something that will help you better understand your joins as well as increasing the readibility of your SQL code.

Now back to our different types of joins!

### Cross Join

<img width="552" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/3176f7a0-e95e-4c54-bf67-f46fe50776cf">

A cross join creates a full combination of all the rows in both tables that are being joined. This type of join is also referred to as a cartesian join which is taken from the original mathematical transformation - the cartesian product of two sets!

The cross join is not so common in practice as it is usually quite an expensive operation when joining large tables but it can be used quite effectively when you actually need all the combinations of 2 tables for specific purposes.

In our example - let’s imagine that for some reason, we wished to cross join the `names` and `jobs` table:

```sql
SELECT 
  n.iid as name_id,
  j.iid as jobs_id,
  n.first_name, 
  n.title,
  j.occupation, 
  j.salary  
FROM `younglapalma.temp.name` AS n
CROSS JOIN `younglapalma.temp.jobs` AS j
ORDER BY n.iid ASC;
```

| name_iid | job_iid | first_name | title               | occupation | salary      |
|----------|---------|------------|---------------------|------------|-------------|
| 1        | 1       | Kate       | Datacated Visualizer | Cleaner    | High        |
| 1        | 2       | Kate       | Datacated Visualizer | Janitor    | Medium      |
| 1        | 3       | Kate       | Datacated Visualizer | Monkey     | Low         |
| 1        | 6       | Kate       | Datacated Visualizer | Plumber    | Ultra       |
| 1        | 7       | Kate       | Datacated Visualizer | Hero       | Plus Ultra  |
| 2        | 1       | Eric       | Captain SQL          | Cleaner    | High        |
| 2        | 2       | Eric       | Captain SQL          | Janitor    | Medium      |
| 2        | 3       | Eric       | Captain SQL          | Monkey     | Low         |
| 2        | 6       | Eric       | Captain SQL          | Plumber    | Ultra       |
| 2        | 7       | Eric       | Captain SQL          | Hero       | Plus Ultra  |
| 3        | 1       | Danny      | Data Wizard Of Oz   | Cleaner    | High        |
| 3        | 2       | Danny      | Data Wizard Of Oz   | Janitor    | Medium      |
| 3        | 3       | Danny      | Data Wizard Of Oz   | Monkey     | Low         |
| 3        | 6       | Danny      | Data Wizard Of Oz   | Plumber    | Ultra       |
| 3        | 7       | Danny      | Data Wizard Of Oz   | Hero       | Plus Ultra  |
| 4        | 1       | Ben        | Mad Scientist        | Cleaner    | High        |
| 4        | 2       | Ben        | Mad Scientist        | Janitor    | Medium      |
| 4        | 3       | Ben        | Mad Scientist        | Monkey     | Low         |
| 4        | 6       | Ben        | Mad Scientist        | Plumber    | Ultra       |
| 4        | 7       | Ben        | Mad Scientist        | Hero       | Plus Ultra  |
| 5        | 1       | Dave       | Analytics Heretic   | Cleaner    | High        |
| 5        | 2       | Dave       | Analytics Heretic   | Janitor    | Medium      |
| 5        | 3       | Dave       | Analytics Heretic   | Monkey     | Low         |
| 5        | 6       | Dave       | Analytics Heretic   | Plumber    | Ultra       |
| 5        | 7       | Dave       | Analytics Heretic   | Hero       | Plus Ultra  |
| 6        | 1       | Ken        | The YouTuber        | Cleaner    | High        |
| 6        | 2       | Ken        | The YouTuber        | Janitor    | Medium      |
| 6        | 3       | Ken        | The YouTuber        | Monkey     | Low         |
| 6        | 6       | Ken        | The YouTuber        | Plumber    | Ultra       |
| 6        | 7       | Ken        | The YouTuber        | Hero       | Plus Ultra  |


Cross joins are also commonly seen with the following syntax also without the `CROSS JOIN` and instead just using a comma to separate the tables to be cross joined like so:

```sql
SELECT 
  n.iid as name_id,
  j.iid as jobs_id,
  n.first_name, 
  n.title,
  j.occupation, 
  j.salary  
FROM 
  `younglapalma.temp.name` AS n, 
  `younglapalma.temp.jobs` AS j
```

### Alternative Cross Inner Join Syntax

Additionally - this is actually quite a common usage to `INNER JOIN` tables implicitly using a `WHERE` filter instead of using the regular `INNER JOIN` and `ON` clauses.

However, although it is common - I would not recommend to implement your joins in this way as it is not always super clear to the reader just exactly what sort of join you are trying to implement and would recommend you to stick with the regular `INNER` and `LEFT` join implementations instead of the following example (but I thought it would be interesting to share with you the code anyway just so you can see the differences!)

```sql
SELECT 
  n.iid as name_id,
  j.iid as jobs_id,
  n.first_name, 
  n.title,
  j.occupation, 
  j.salary  
FROM 
  `younglapalma.temp.jobs` AS j
  `younglapalma.temp.name` AS n, 
WHERE n.iid = j.iid
ORDER BY n.iid ASC
```

| name_iid | job_iid | first_name | title                 | occupation | salary |
|----------|---------|------------|-----------------------|------------|--------|
| 1        | 1       | Kate       | Datacated Visualizer  | Cleaner    | High   |
| 2        | 2       | Eric       | Captain SQL           | Janitor    | Medium |
| 3        | 3       | Danny      | Data Wizard Of Oz     | Monkey     | Low    |
| 6        | 6       | Ken        | The YouTuber          | Plumber    | Ultra  |

# Advanced Joins

So I’ve labeled the following joins as “advanced” joins but in all honesty they are not that advanced and it’s more about how often they are used in the workplace.

Honestly most of the times where I’ve seen people use these “advanced” joins on various projects - these people were the experts and advanced practitioners who were serious about joining efficiency. And frankly - we should ALL be super serious about join efficiency as it is one of THE most sought after skills within the SQL world!

When going through these types of joins - try to compare them to the previous basic joins we discussed and understand the differences between them. This is the key for understanding these very important joins!

We will cover the following new types of joins in this section:

- `LEFT SEMI JOIN` or `WHERE EXISTS`
- `ANTI JOIN` or `WHERE NOT EXISTS`

The main purpose of using these advanced joins is to remove/exclude rows from a table where specific column values are based off a separate lookup table.

They also have the added benefit of helping eliminate some unexpected behaviour when there are duplicates in the “target” table or in join operations.

To better understand these new joins, we must first take a detour and revisit duplicates which we touched on earlier Data Exploration part of the Serious SQL course!

##  Left Semi Join

<img width="545" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/e391049a-3ff4-478f-8c1b-244df4dca1aa">

A left semi join is actually really similar to an `INNER JOIN` where it captures only the matching records between left and right tables BUT it differs in one very key way: it only returns records from the left table - no columns or rows from the right table are included in the output.

The following series of SQL snippets will demonstrate the differences and similarities very clearly.

In PostgreSQL Standard SQL, this `LEFT SEMI JOIN` can be expressed using a `WHERE EXISTS` clause but in some other SQL flavours `LEFT SEMI JOIN` can be called directly also.

Take very special attention in the syntax for the query below - it actually differs quite a bit from a regular join operation with the `ON` clause.

The `WHERE` clause inside the surrounding `WHERE EXISTS` is the equivalent of the ON clause for a regular join syntax.

**`WHERE EXISTS` Syntax**

The following code can be ran directly in BigQuery:

```sql
SELECT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
WHERE EXISTS (
  SELECT iid
  FROM `younglapalma.temp.new_jobs` as new_jobs
  WHERE names.iid = new_jobs.iid
)
ORDER BY iid ASC;
```

| iid | first_name |
|-----|------------|
| 1   | Kate       |
| 2   | Eric       |
| 3   | Danny      |
| 6   | Ken        |

One interesting tidbit is that it doesn’t matter whatsoever what you choose to select inside the subquery for the `WHERE EXISTS`

The same query will run if I simply select `1` instead of the `iid` column - you can try it yourself!

```sql
SELECT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
WHERE EXISTS (
  SELECT 1
  FROM `younglapalma.temp.new_jobs` as new_jobs
  WHERE names.iid = new_jobs.iid
)
ORDER BY iid ASC;
```

The key thing is that the query within the subquery part of the `WHERE EXISTS` does not actually return any data.

Some SQL practitioners will specifically use the `SELECT 1` in their `WHERE EXISTS` to make it clear that actually nothing is being returned - just make sure to add some comments in your code, just in case whoever is reading has never seen a `LEFT SEMI JOIN` before!

**`LEFT SEMI JOIN` Syntax**

The following code snippet will not run in PostgreSQL but might work in other flavours - it is the equivalent of the above `WHERE EXISTS` implementation.

Notice how it is exactly the same as any other regular join implementation with the `ON` clause.

```sql
SELECT
  names.iid,
  names.first_name
FROM names
LEFT SEMI JOIN new_jobs
ON names.iid = new_jobs.iid;
```

### Alternative Left Semi Understanding

Another way to think of this sort of join is in terms of a regular `LEFT JOIN` with a `DISTINCT` followed by a `WHERE` filter to remove all null values from the resulting joint dataset.

Note that the following query is quite inefficient so it is only shown here for explanation purposes - the important point is the DISTINCT that is in the first line below:

```sql
SELECT DISTINCT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
LEFT JOIN `younglapalma.temp.new_jobs` as new_jobs
  ON names.iid = new_jobs.iid
WHERE new_jobs.iid IS NOT NULL;
```

| iid | first_name |
|-----|------------|
| 6   | Ken        |
| 1   | Kate       |
| 3   | Danny      |
| 2   | Eric       |

OK cool - this looks exactly the same as our previous `WHERE EXISTS` query - but what happens when we drop that `DISTINCT` from the above query?

```sql
SELECT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
LEFT JOIN `younglapalma.temp.new_jobs` as new_jobs
  ON names.iid = new_jobs.iid
WHERE new_jobs.iid IS NOT NULL;
```

| iid | first_name |
|-----|------------|
| 1   | Kate       |
| 1   | Kate       |
| 2   | Eric       |
| 3   | Danny      |
| 3   | Danny      |
| 6   | Ken        |

### Left Semi Join Vs Inner Join
This is the final comparison and it is also the most important one!

Let’s now use an `INNER JOIN` to show this same `LEFT SEMI JOIN` operation, firstly with a `DISTINCT` and then without `DISTINCT` - note that now we no longer need to check for the null values coming from the right table since we’re doing an `INNER JOIN` and not a `LEFT JOIN`

```sql
SELECT DISTINCT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
INNER JOIN `younglapalma.temp.new_jobs` as new_jobs
  ON names.iid = new_jobs.iid;
```
| iid | first_name |
|-----|------------|
| 6   | Ken        |
| 1   | Kate       |
| 3   | Danny      |
| 2   | Eric       |

As expected this matches exactly with our original `WHERE EXISTS` implementation. What about without the `DISTINCT`?

```sql
SELECT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
INNER JOIN `younglapalma.temp.new_jobs` as new_jobs
  ON names.iid = new_jobs.iid;
```
| iid | first_name |
|-----|------------|
| 1   | Kate       |
| 1   | Kate       |
| 2   | Eric       |
| 3   | Danny      |
| 3   | Danny      |
| 6   | Ken        |

Hopefully this illustrates the difference between the two joins very clearly!

These last 2 snippets are the most important in understanding the difference between the `LEFT SEMI JOIN` and `INNER JOIN` and also provide some insight into some scenarios where you might want to use one versus another!

Now that we’ve covered the `LEFT SEMI JOIN` concept - we can now introduce the `ANTI JOIN` which is the exact opposite of the `LEFT SEMI JOIN`.

# Anti Join

<img width="433" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/2c8753a1-cf31-428c-ac15-ce94beb119ad">

An `ANTI JOIN` is the opposite of a `LEFT SEMI JOIN` where only records from the left which do not appear on the right are returned - it is expressed as a `WHERE NOT EXISTS` clause in PostgreSQL but similar to the `LEFT SEMI JOIN` some SQL flavours support a direct `ANTI JOIN` syntax also.

```sql
SELECT
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names


WHERE NOT EXISTS (
  SELECT 1
  FROM `younglapalma.temp.new_jobs` as new_jobs
  WHERE names.iid = new_jobs.iid
)
ORDER BY iid ASC;
```

The regular `ANTI JOIN` syntax also works for some SQL flavours too - this won’t run in PostgreSQL though!

```sql
SELECT
  names.iid,
  names.first_name
FROM names
ANTI JOIN new_jobs
  ON names.iid = new_jobs.iid
```

### A Popular Alternative

Here is an alternative way to represent a similar `ANTI JOIN` using a `LEFT JOIN` with a null `WHERE` filter - we actually see this a lot in practice too:

```sql
SELECT 
  names.iid,
  names.first_name
FROM `younglapalma.temp.name` as names
LEFT JOIN `younglapalma.temp.new_jobs` as new_jobs
ON names.iid = new_jobs.iid
WHERE new_jobs.iid IS NULL;
```

| iid | first_name |
|-----|------------|
| 4   | Ben        |
| 5   | Dave       |

So you might be thinking - but Danny, you just told me that a `LEFT SEMI JOIN` is not the same as a `LEFT JOIN` with a `IS NOT NULL` in a previous section - how come this doesn’t apply for the `ANTI JOIN`?

Let’s step through this and understand why it’s fine for an `ANTI JOIN`.

Here is the basic implementation of a left join between our original `names` table and our `new_jobs` table:

```SQL
SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM `younglapalma.temp.name` as names
LEFT JOIN `younglapalma.temp.jobs` as jobs  -- notice the aliases here!
  ON names.iid = jobs.iid
ORDER BY names.iid ASC;
```

| iid | first_name | title                 | occupation | salary    |
|-----|------------|-----------------------|------------|-----------|
| 1   | Kate       | Datacated Visualizer  | Cleaner    | Very High |
| 1   | Kate       | Datacated Visualizer  | Cleaner    | High      |
| 2   | Eric       | Captain SQL           | Janitor    | Medium    |
| 3   | Danny      | Data Wizard Of Oz     | Monkey     | Very Low  |
| 3   | Danny      | Data Wizard Of Oz     | Monkey     | Low       |
| 4   | Ben        | Mad Scientist         | null       | null      |
| 5   | Dave       | Analytics Heretic     | null       | null      |
| 6   | Ken        | The YouTuber          | Plumber    | Ultra     |

Scanning through the output rows - we can see those same duplicated rows for Kate and Danny which we did not like when we were doing the `LEFT JOIN` with the `IS NOT NULL` but what do you see when you inspect those last 3 rows for Ben, Dave and Ken?

Indeed - there are only single rows for each of these records which are not matching between the two tables!

So when we perform an additional filter `WHERE new_jobs.iid IS NULL` - it only returns us individual rows for these non-matching records.

In summary - this combination of `LEFT JOIN` and `IS NULL` implementation for anti joins are very popular in practice so it’s worth understanding what is going under the hood!

# Joining on Null Values
So what would happen if we try to join tables which have `null` in the joining columns?

In this episode of “Will It Join?” we will create 2 new adjusted tables to check if they will work as expected - to make this even more challenging, let’s keep in those duplicates that we just used for the previous `LEFT SEMI` and `ANTI` join section:

```sql
DROP TABLE IF EXISTS `younglapalma.temp.null_names`;
CREATE TABLE `younglapalma.temp.null_names` AS
SELECT *
FROM UNNEST([
    STRUCT(1 AS iid,    'Kate' AS first_name,   'Datacated Visualizer' AS title),
    (2,    'Eric',   'Captain SQL'),
    (3,    'Danny',  'Data Wizard Of Oz'),
    (4,    'Ben',    'Mad Scientist'),
    (5,    'Dave',   'Analytics Heretic'),
    (6,    'Ken',    'The YouTuber'),
    (null, 'Giorno', 'OG Data Gangster')
]);
SELECT * from `younglapalma.temp.null_names`;

DROP TABLE IF EXISTS `younglapalma.temp.null_jobs`;
CREATE TABLE `younglapalma.temp.null_jobs` AS
SELECT *
FROM UNNEST([
    STRUCT(1 AS iid,    'Cleaner' AS occupation,    'High' AS salary),
    (1,    'Cleaner',    'Very High'),
    (2,    'Janitor',    'Medium'),
    (3,    'Monkey',     'Low'),
    (3,    'Monkey',     'Very Low'),
    (6,    'Plumber',    'Ultra'),
    (7,    'Hero',       'Plus Ultra'),
    (null, 'Mastermind', 'Bank')
]);
SELECT * from `younglapalma.temp.null_jobs`;
```

**So will it join???**

##  Inner Join


```SQL
SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM `younglapalma.temp.null_names` as names
INNER JOIN `younglapalma.temp.null_jobs` as jobs
  ON names.iid = jobs.iid
ORDER BY names.iid ASC;
```

| iid | first_name | title                 | occupation | salary    |
|-----|------------|-----------------------|------------|-----------|
| 1   | Kate       | Datacated Visualizer  | Cleaner    | Very High |
| 1   | Kate       | Datacated Visualizer  | Cleaner    | High      |
| 2   | Eric       | Captain SQL           | Janitor    | Medium    |
| 3   | Danny      | Data Wizard Of Oz     | Monkey     | Very Low  |
| 3   | Danny      | Data Wizard Of Oz     | Monkey     | Low       |
| 6   | Ken        | The YouTuber          | Plumber    | Ultra     |


Result: NO - it does not join!


## Left Join

**`null_names` as the base**

```sql
SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM `younglapalma.temp.null_names` as names
LEFT JOIN `younglapalma.temp.null_jobs` as jobs
  ON names.iid = jobs.iid
ORDER BY names.iid ASC;
```

| iid  | first_name | title                 | occupation | salary    |
|------|------------|-----------------------|------------|-----------|
| 1    | Kate       | Datacated Visualizer  | Cleaner    | Very High |
| 1    | Kate       | Datacated Visualizer  | Cleaner    | High      |
| 2    | Eric       | Captain SQL           | Janitor    | Medium    |
| 3    | Danny      | Data Wizard Of Oz     | Monkey     | Very Low  |
| 3    | Danny      | Data Wizard Of Oz     | Monkey     | Low       |
| 4    | Ben        | Mad Scientist         | null       | null      |
| 5    | Dave       | Analytics Heretic     | null       | null      |
| 6    | Ken        | The YouTuber          | Plumber    | Ultra     |
| null | Giorno     | OG Data Gangster      | null       | null      |


**`null_jobs` as the base**

```sql
SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM `younglapalma.temp.null_jobs` as jobs
LEFT JOIN `younglapalma.temp.null_names` as names
  ON names.iid = jobs.iid
ORDER BY names.iid ASC;
```

| iid  | occupation | salary     | first_name | title                |
|------|------------|------------|------------|----------------------|
| 1    | Cleaner    | High       | Kate       | Datacated Visualizer |
| 1    | Cleaner    | Very High  | Kate       | Datacated Visualizer |
| 2    | Janitor    | Medium     | Eric       | Captain SQL          |
| 3    | Monkey     | Low        | Danny      | Data Wizard Of Oz    |
| 3    | Monkey     | Very Low   | Danny      | Data Wizard Of Oz    |
| 6    | Plumber    | Ultra      | Ken        | The YouTuber         |
| 7    | Hero       | Plus Ultra | null       | null                 |
| null | Mastermind | Bank       | null       | null                 |

Result: NO - it does not join (well not in the way that we were expecting anyway!)

## Left Semi Join

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE EXISTS (
  SELECT 1
  FROM `younglapalma.temp.null_jobs` as null_jobs
  WHERE null_names.iid = null_jobs.iid
)
ORDER BY iid ASC;
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 1   | Kate       | Datacated Visualizer |
| 2   | Eric       | Captain SQL          |
| 3   | Danny      | Data Wizard Of Oz    |
| 6   | Ken        | The YouTuber         |

Result: NO - it does not join!

## Anti Join

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE NOT EXISTS (
  SELECT 1
  FROM `younglapalma.temp.null_jobs` as null_jobs
  WHERE null_names.iid = null_jobs.iid
);
```

| iid  | first_name | title              |
|------|------------|--------------------|
| 4    | Ben        | Mad Scientist      |
| 5    | Dave       | Analytics Heretic  |
| null | Giorno     | OG Data Gangster   |

Result: NO - it does not join!

# IN and NOT IN

`IN` and` NOT IN` clauses are very popular ways to filter datasets when used with a `WHERE` clause - so why don’t we rely on these logical conditions instead of joins?
Logically an `IN` is essentially doing what a `WHERE EXISTS` is doing - and `NOT IN` is equivalent to an `ANTI JOIN` - in fact the underlying execution plan created by the PostgreSQL optimizer is exactly the same under certain conditions.
However there is one **VERY IMPORTANT** thing to note and it has to do with null values!

## IN instead of WHERE EXISTS

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE null_names.iid IN (
  SELECT iid
  FROM `younglapalma.temp.null_jobs` as null_jobs
)
ORDER BY iid ASC;
```

So this `IN` does work even in the presence of nulls - let’s see if it works with `NOT IN` too.

## NOT IN instead of WHERE NOT EXISTS

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE null_names.iid NOT IN (
  SELECT
    iid
  FROM `younglapalma.temp.null_jobs` as null_jobs
);
```

| iid | first_name | title |
|-----|------------|-------|
|     |            |       |

That’s odd…it actually returns us no records!

Is it something to do with the duplicates or null values?

When we look at the inner subquery used for the `NOT IN` clause - the outputs look like this:

```sql
SELECT
  iid
FROM `younglapalma.temp.null_jobs` as null_jobs
```
| iid  |
|------|
| 1    |
| 1    |
| 2    |
| 3    |
| 3    |
| 6    |
| 7    |
| null |

Let’s just try this out and see what happens if we manually input those values, keeping that `null` out of the `IN` clause:

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE null_names.iid NOT IN (
  1, 1, 2, 3, 3, 6, 7
);
```

| iid | first_name | title              |
|-----|------------|--------------------|
| 4   | Ben        | Mad Scientist      |
| 5   | Dave       | Analytics Heretic  |

When we manually enter those values (even if they are duplicated!) it works!

What happens if we add that `null` value back in?

```sql
SELECT
  null_names.*
FROM `younglapalma.temp.null_names` as null_names
WHERE null_names.iid NOT IN (
  1, 1, 2, 3, 3, 6, 7, null
);
```

| iid | first_name | title |
|-----|------------|-------|
|     |            |       |

Bingo - we are back at no rows!

It really seems like that `null` value that throws us off!

Technically - a `null` value does not actually represent a value itself - rather it is an unknown value!

When we are performing this `NOT IN` - the underlying problem is how the SQL engine tries to compare this unknown `null` value with other non-null values.

In ANSI SQL we can’t actually check if a value is equal to, greater than or less than `null` - we can only check if it `IS NULL` or `IS NOT NULL`.

ANSI stands for the American National Standards Institute and this managing body specifies set standards for SQL amongst many other things for use across different industries! You can find out more about the ANSI and SQL history by [clicking here!](https://www.whoishostingthis.com/resources/ansi-sql-standards/)"

To demonstrate the behaviour of what happens when we compare `null` values - consider the following query below:

```sql
SELECT
  5 > null AS col1,
  5 < null AS col2,
  5 != null AS col3,
  5 IN (null) AS col4,     -- this is our unknown value
  5 NOT IN (null) AS col5, -- this is our unknown value
  5 IS NULL AS col6,
  5 IS NOT NULL AS col7;
```

| col1 | col2 | col3 | col4 | col5 | col6  | col7 |
|------|------|------|------|------|-------|------|
| null | null | null | null | null | false | true |

So in our situation - our query short circuits when it hits that final `null` value and returns nothing!

Previously when we had the `IN` comparison with the `null` values - there was no issue because there were matching terms within the `IN` values we were checking against which returned us a true value.

However this is not quite the case for the `NOT IN` variation of our query!

To keep things simple and less prone to issues - try to always stick with `LEFT SEMI JOIN` and `ANTI JOIN` instead of `IN` and `NOT IN` where you do not know what exact values will appear for the IN subquery.

The behaviour when `null` values are present in the subquery for the filtering is not always stable and may lead to incorrect values!

Let’s move onto our final section of this table join tutorial - set combinations:

# Set Operations
There are 3 distinct set operations used to combine results from 2 or more `SELECT` statements.

- UNION (& UNION ALL)
- INTERSECT
- EXCEPT

These 3 methods are pretty straightforward however as it matches directly with set notation, however the only condition is that all of the `SELECT` result sets must have columns with the same data types - the column names have no impact.

Let’s examine the following sample queries:

## Union

<img width="544" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/0f195aa5-e4d7-439f-bc43-2ed8c7c5c5c4">

Simply put - `UNION` will be the union between two sets or `SELECT` results.

```sql
SELECT * FROM `younglapalma.temp.name` where first_name = 'Danny'
UNION DISTINCT
SELECT * FROM `younglapalma.temp.name` where first_name = 'Kate'
ORDER BY iid ASC;
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 1   | Kate       | Datacated Visualizer |
| 3   | Danny      | Data Wizard Of Oz    |


## Union All

<img width="548" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/1f38ffea-dcb9-4726-92b4-c999104663c0">

So the question you must have right now is: What is the difference between `UNION` and `UNION ALL`?

Simply - `UNION` runs a `DISTINCT` on the output result whilst `UNION ALL` does not.

This means that `UNION ALL` will be more performant as it does not run a relatively expensive `DISTINCT`

The above question is a popular SQL interview questions, so please keep this in mind!

Let’s demonstrate with a simple example:

If we change the queries above to just filter on `'Danny'` and try running `UNION` - what do we see?

```sql
SELECT * FROM `younglapalma.temp.name` where first_name = 'Danny'
UNION DISTINCT
SELECT * FROM `younglapalma.temp.name` where first_name = 'Danny'
ORDER BY iid ASC;
```

| iid | first_name | title            |
|-----|------------|------------------|
| 3   | Danny      | Data Wizard Of Oz|


Now let’s try running the `UNION ALL` instead

```sql
SELECT * FROM `younglapalma.temp.name` where first_name = 'Danny'
UNION ALL
SELECT * FROM `younglapalma.temp.name` where first_name = 'Danny'
ORDER BY iid ASC;
```

| iid | first_name | title            |
|-----|------------|------------------|
| 3   | Danny      | Data Wizard Of Oz|
| 3   | Danny      | Data Wizard Of Oz|

That’s all there is to it!

Now - a few things to consider when you need to combine the datasets in practice:

1. Do I have duplicates in my individual results set?
2. Do I have duplicates in my overall results set after `UNION ALL`?
3. Should I refactor the query to generate distinct values or use a `GROUP BY`?

## Intersect

<img width="544" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/7d8daa90-4133-4bb3-b7d8-0e39aad43f40">

Intersect is pretty straightforward in that only the records which exist in both tables will be returned - a little bit similar to an `INNER JOIN` don’t you think?

In the following example - let’s try to find the intersection between all the records in `names` and a filtered dataset where only the records where the first name starts with ‘K’ are included:

```sql
SELECT * FROM `younglapalma.temp.name`
INTERSECT DISTINCT
SELECT * FROM `younglapalma.temp.name` where LEFT(first_name, 1) = 'K'
ORDER BY iid ASC;
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 1   | Kate       | Datacated Visualizer |
| 4   | Ken        | The YouTuber         |

## Except

<img width="548" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/b8c85eb2-abd9-4860-b027-62fd5f41d7e0">

Except is the equivalent of the `ANTI JOIN` in set notation. Let’s try running `EXCEPT` with the same filtered names starting with ‘K’ again to see what we get:

```sql
SELECT * FROM `younglapalma.temp.name`
EXCEPT DISTINCT
SELECT * FROM `younglapalma.temp.name` where LEFT(first_name, 1) = 'K';
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 3   | Danny      | Data Wizard Of Oz    |
| 5   | Dave       | Analytics Heretic    |
| 2   | Eric       | Captain SQL          |
| 4   | Ben        | Mad Scientist        |

Also note that `EXCEPT` has a default `DISTINCT` built in so it’s doing the same deduplication as the `UNION`

## Multiple Combinations

We can also combine these with 2+ queries like so - just be aware that all of these set operations will be executed in order from top to bottom!

```sql
(SELECT * FROM `younglapalma.temp.name` WHERE first_name = 'Danny'
UNION DISTINCT
SELECT * FROM `younglapalma.temp.name` where LEFT(first_name, 1) = 'K')
EXCEPT DISTINCT
SELECT * FROM `younglapalma.temp.name` where first_name = 'Kate';
```

## Common `UNION` Mistakes
Here are a few common mistakes I’ve seen throughout the years related to `UNION`.

I’ve grouped them into not-so-silent mistakes and silent mistakes - the first ones will return you an error when you do something wrong - but the silent mistakes will not!

###  Not-So-Silent Mistakes

What happens when we have the wrong number of columns for a combination?

```SQL
SELECT first_name FROM names WHERE first_name = 'Danny'
UNION
SELECT iid, first_name FROM names where LEFT(first_name, 1) = 'K';
```
> ERROR:  each `UNION` query must have the same number of columns

What about when you have the wrong data types?

```SQL
SELECT first_name, iid FROM names WHERE first_name = 'Danny'
UNION
SELECT iid, first_name FROM names where LEFT(first_name, 1) = 'K';
```
> ERROR:  `UNION` types text and integer cannot be matched

As you can see these mistakes are relatively easy to identify and fix - so just keep an eye out to make sure the data types are and columns match!

### Silent Mistakes

However, what happens when we accidentally choose the wrong columns but their position and data types match?

However, what happens when we accidentally choose the wrong columns but their position and data types match?

```SQL
(SELECT iid, first_name, title FROM `younglapalma.temp.name` WHERE first_name = 'Danny'
UNION DISTINCT
SELECT iid, occupation, salary  FROM `younglapalma.temp.jobs` where iid = 2)
UNION DISTINCT
SELECT * FROM `younglapalma.temp.name` where first_name = 'Kate';
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 1   | Kate       | Datacated Visualizer |
| 2   | Janitor    | Medium               |
| 3   | Danny      | Data Wizard Of Oz    |

Oh…this is not quite what we expected! But note also that the column names for the final result are according to the first query that was used in the combination!

Let’s confirm this is the case by moving the second query to the first position

```SQL
(SELECT iid, occupation, salary  FROM `younglapalma.temp.jobs` where iid = 2
UNION DISTINCT
SELECT iid, first_name, title FROM `younglapalma.temp.name` WHERE first_name = 'Danny')
UNION DISTINCT
SELECT * FROM `younglapalma.temp.name` where first_name = 'Kate';
```

| iid | first_name | title                |
|-----|------------|----------------------|
| 1   | Kate       | Datacated Visualizer |
| 2   | Janitor    | Medium               |
| 3   | Danny      | Data Wizard Of Oz    |

This is why you must be very careful when using `UNION`, `INTERSECT` and `EXCEPT`

And that’s a wrap for all things combinations!

# Conclusion

In this super long tutorial - we covered the expected output for our SQL case study before diving into a Gentle Introduction to SQL Joins!

In the next tutorial we will be using some of our new knowledge of table joins to continue tackling our marketing case study!

If you need a TL;DR despite reaching the end of this tutorial, here it is:

- Use `LEFT JOIN` when you need to keep all rows on the left table but want to join on additional records from the right table
- Be wary of which table is on the left vs right side of the `LEFT JOIN` as all left table records will be retained regardless of the join condition
- Use `INNER JOIN` when you need the intersection of both left and right tables and you also need to access columns from the right table
- Use `FULL OUTER JOIN` or `FULL JOIN` when you need to combine both left and right tables completely (similar to a `UNION ALL` operation without deduplication)
- Use `CROSS JOIN` when you need the cartesian product or the complete combinations of left and right tables
- Be careful of duplicate join keys from left and right tables when using `LEFT` and `INNER` and `CROSS` joins
- Use `LEFT SEMI JOIN` or `WHERE EXISTS` when you only need to keep records from the left table that exist in the right table
- Use `ANTI JOIN` when you need to remove exclude records from the left table based off the right table
- Use `UNION` to get the deduplicated or distinct combination of two tables or SQL query outputs - column positions and data types must align!
- Use `UNION ALL` to get the unduplicated combination of two tables or SQL query outputs - column positions and data types must align!
- Use `INTERSECT` and `EXCEPT` for set operations between tables or SQL query outputs - column positions and data types must align!
- =, >=, <=, >, < or BETWEEN conditions can be used for the `ON` clause for table joins
- Use aliases and/or explicit references when joining tables and selecting columns to improve readability and comprehension of code, especially when joining multiple tables in one SQL statement

I hope this tutorial has helped you demystify some of the mystique around SQL joins - here is a quick GIF of all the joins to recap all that we’ve learned so far!

# Exercises
Imagine that you are in the middle of a SQL interview and your counterpart asks you the following questions:

1. How would you explain table joins to a senior executive who has zero knowledge of data?
> Imagine you have two separate address books:
>
>  **Address Book A** lists people's names and their phone numbers.
  **Address Book B** lists people's names and their email addresses.
  Now, you want to create a combined address book that shows each person’s name, phone number, and email address side by side.
>  
>  The process of merging these two address books based on the name (a common element in both) is akin to a "table join" in the world of data. Just as you might join or combine two address books to see related information side by side, in data we often need to combine tables to get a more complete view of our data.



2. What is the difference between a left join, inner join and left-semi join when would you use one over another?

> Inner Join (like a strict match): Only include people who are in **BOTH** address books. If someone is missing in either one, they won't be in the combined list.
>
> Left Join (include all from the left book): List everyone from **Address Book A**. If they also exist in **Address Book B**, include their email. If not, leave the email blank.
>
> Left-Semi Join (like a name-checker): Imagine you want to send a text message to everyone, but you only want to text people from **Address Book A** for whom you also have an email in **Address Book B**. A left-semi join would give you a list of names from **Address Book A** for whom there's a matching name in **Address Book B**. However, unlike the other joins, you won't see the email addresses. It's more about filtering **Address Book A** based on **Address Book B**'s contents.

3. How do duplicates in the foreign key affect joins - what are some common issues and some strategies to avoid them?

> Issues due to Duplicates in the Foreign Key:
>
> 1. **Inflated Results**: When you join on a foreign key that has duplicates, the resulting dataset might be larger than expected. This is because each record in the primary table will join to multiple records in the secondary table.
>
> 2. **Skewed Analysis**: If you're using the joined data for analytical purposes, the presence of duplicate foreign keys can distort your results, giving undue weight to certain records.
>
> 3. **Increased Complexity**: Duplicates can complicate data transformation and processing. For instance, aggregating data becomes more challenging when unexpected duplicates are present.
>
>4. **Storage Concerns**: Duplicates can lead to a rapid increase in the volume of data, especially in big data scenarios. This can have downstream impacts on storage, processing time, and costs.

> Strategies to Handle Duplicates:
>
> 1. **Data Cleaning Before Join**: Before performing the join, identify and remove or consolidate duplicate foreign keys. This can be done using tools or SQL queries that allow for de-duplication based on certain criteria.
>
> 2. **Use Distinct**: When fetching the foreign key values for a join, using the DISTINCT keyword can help in removing duplicates, ensuring each key is unique.
>
> 3. **Aggregate Before Join**: If you're interested in metrics or summaries, consider aggregating the data in the secondary table before the join. For instance, if the secondary table has sales data, and there are duplicate foreign keys, you can aggregate sales by the foreign key to get total sales per key.
>
> 4. **Understanding the Business Context**: Sometimes, duplicates in the foreign key might be valid from a business perspective. It's essential to understand the nature of the data and the reason for duplicates before deciding on a strategy to handle them.
>
> 5. **Use Semi Joins**: If you're only interested in filtering records in one table based on the existence of foreign key values in another, consider using semi-joins (like the left-semi join). These joins filter records without actually combining datasets, so the impact of duplicates is mitigated.
>
> 6. **Database Constraints**: On the database design side, consider using unique constraints or indices to ensure that only unique foreign key values are entered into the database. This might not be suitable for all scenarios, especially if duplicates are valid, but it's a powerful preventative measure when applicable.

4. What is the ON condition for table joins and what different varieties are available to use with data?

> The `ON` condition in table joins specifies the criteria based on which two tables should be joined. It's essentially the rule or set of rules that dictate how rows from one table relate to rows in another table. The `ON` condition is central to ensuring the relevance of the merged data.

5. What are aliases and how might they be used to speedup SQL development in teams?

## 10.2. SQL Join Performance Tips

A few more pointers on speeding up SQL joins are included below but are out of scope of this gentle introduction - we might touch on this in a future case study where SQL optimization is required to actually complete the solution implementation.

- Be wary of table sizes during joins - the general rule of thumb is to only ever join the minimal amount of data possible and make use of indexes and partitions wherever possible
- Use `LEFT SEMI JOIN` instead of `WHERE column_name IN` clauses when dealing with a large dataset on the left that needs to be filtered based off a smaller lookup table on the right - avoid using `INNER JOIN` for these situations unless you specifically require columns from the right table
- Always check for null values and count distribution of uniqe values for each foreign key to avoid query skew - remove null values where relevant (i.e. 80% of values are null and get sent to a single executor/node for processing causing slower query speeds)
- If you are creating your own tables - you can consider using clustered indexes or partitioned tables to optimize access speeds for most queries but you will need to factor in costs of managing certain overheads such as deletions and updates
- Use temporary tables for pre-filtered and re-indexed datasets where possible, this is most likely the case unless you have database admin rights to alter existing tables
- Use functional indexes for type casting if available when temporary tables are not an option like in the case of views
- Avoid casting columns to a new data type within an `ON` condition for a join - create temporary tables with a new index on the casted data type foreign key instead
- Multi-level table indexes containing foreign keys should be stated in order of high to low cardinality (i.e. order indexes by the columns with most number of unique records first, followed by the following columns with less unique values)
- PostgreSQL requires table statistics to be collected before the optimizer can use them for performance improvements - `run ANALYZE TABLE <indexed-temp-table-name>;` after creating any temporary tables to enable execution speed-ups

Checkout [this article](https://explainextended.com/2009/09/16/not-in-vs-not-exists-vs-left-join-is-null-postgresql/) which goes into a ton of depth about the optimization choices for all the different types of joins - it is out of scope for our gentle introduction into SQL joins but it makes for some interesting reading once you are comfortable with some of the different concepts introduced in this tutorial!

This is [another article](https://sqlperformance.com/2012/12/t-sql-queries/left-anti-semi-join) that goes into the explain and execution plans for `IN` vs `WHERE EXISTS` and is really interesting - I actually used this as a reference for parts of this tutorial.

`EXPLAIN` and `EXPLAIN ANALYZE` will be covered in more depth later in the Serious SQL course!

Partitioning strategies are also very effective when thinking about using data at scale but is sadly outside of scope for this course - also it’s most likely that you will be dealing with data engineers or database administrators when tackling these sorts of tasks so it’s best to learn from them directly when the time is right. Think of it as Just-In-Time learning!

