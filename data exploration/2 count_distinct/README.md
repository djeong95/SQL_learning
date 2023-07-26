
## Column Aliases
```sql
SELECT
  COUNT(*) AS row_count
FROM dvd_rentals.film_list;
```
Notice in the query above how there is an extra `AS row_count` after that `COUNT(*)`?

This is known as a “column alias” which is used to specify a name for an expression in the select statement. In this case, we are renaming our `COUNT(*)` expression to “row_count” instead of the default name “count”.

Aliases can also be used to name other SQL expressions such as database tables, subqueries and common table expressions (CTEs).

Aliases are really important when joining tables and performing other operations with CTEs as they vastly improve SQL code readability - reducing the time it takes for you to write and debug the code, as well as for others to quickly scan and understand your code!

There is a saying - code is written once, read multiple times! This is exactly why we must strive to always write simple understandable code (that also looks nice!)

One more additional note - sometimes you might see SQL record count queries written with `COUNT(1)` instead of `COUNT(*)` - in essence there is no difference, it is purely a stylistic choice!

My recommendation would be to use the same convention as whatever your team uses in the workplace - however my personal preference would be to use `COUNT(*)` where possible as it is the most clear for anyone else reading your code!

## `DISTINCT` For Unique Values

### Show Unique Column Values
We can use the `DISTINCT` keyword to obtain unique values from a deduplicated target column.

You might hear the terms dedupe, deduplicate, distinct or unique interchangeably in the workplace but just know that these all mean the same thing!

> What are the unique values for the `rating` column in the `film` table?

```sql
SELECT DISTINCT
  rating
FROM dvd_rentals.film_list
```
Answer: PG, NC-17, PG-13, R, G

### Count of Unique Values
Maybe you’re not interested in the actual unique values themselves but rather, you might want to know how many of them there are - in other words, you want to know the count of distinct values within a column.

We can use the `COUNT` function with the `DISTINCT` keyword to find the number of unique values of a specific column.

> How many unique `category` values are there in the `film_list` table?

```sql
SELECT
  COUNT(DISTINCT category) AS unique_category_count
FROM dvd_rentals.film_list
```

Answer: 16

## Group By Counts
Although the unique values or a distinct count of one column can be very useful when answering certain types of data questions - we can take this style of analysis further by using the `GROUP BY` clause with a `COUNT` aggregate function to help us generate a basic frequency value counts output.

One way to think of the `GROUP BY` is to imagine our dataset being divided into different groups based off the values of selected columns.

Say for example - we would like to answer the following question:

> What is the frequency of values in the `rating` column in the `film_list` table?

Before diving right into the query to demonstrate how to implement this `GROUP BY` - let’s have play with a simplified table example first showing the first 10 rows of a few columns from the `dvd_rentals.film_list` table

```sql
-- Group by query using common table expression
WITH example_table AS (
  SELECT
    fid,
    title,
    category,
    rating,
    price
  FROM dvd_rentals.film_list
  LIMIT 10
)
SELECT
  rating,
  COUNT(*) as record_count
FROM example_table
GROUP BY rating
ORDER BY record_count DESC;
```
### Dividing Rows
The first step would be to evaluate the different values within the column expression we’re using for the grouping element used for the `GROUP BY` clause.

In our simplified 10 row sample dataset - we can see following distinct `rating` values:

- PG-13
- G
- NC-17
- PG
- R

Under the hood - we will actually be splitting our 10 row sample dataset into separate groups based off this `rating` value at each individual row level.

### Apply Aggregate Count Function
Once we have split the dataset into the groups specified for the `GROUP BY` we then apply the aggregate function within each of these grouped datasets to condense our output to a single row from each group.

In our simple example - when we apply a simple `COUNT` on each group of rows, it acts just like it does for a regular old query returning 4 and 2 respectively for our PG-13 and G rating groups and 1 for every other value in our sample dataset.

In future tutorials, we will use this exact same construct to apply different type of mathematical aggregate functions to `GROUP BY` example such as `SUM`, `MEAN`, `STDDEV`, `MAX` and `MIN`.

The important thing to note for `GROUP BY` aggregate functions is this:

> Only 1 row is returned for each group

This is a super important concept - true mastery of SQL requires a really strong understanding of this `GROUP BY` usage. If you don’t retain anything else from this section - please remember that only 1 row will ever be returned for each individual group from a `GROUP BY`!

Only the expression that is used in the `GROUP BY` grouping elements will be returned along with a single column value for each aggregate function used in the column expressions for the `SELECT` statement.

### Combining Condensed Outputs
Once the calculations are complete for each separate group, the condensed 1 row from each group is then combined together to form the final output for the original SQL statement with the `GROUP BY` clause.

| rating	| record_count |
|:--:|:--:|
| PG-13 | 4 |
| G |	2 |
| NC-17	| 2 |
| PG	| 1 |
| R	| 1 |
Finally - we got back to where we started! I hope this simple example helps you to understand the underlying concept for `GROUP BY` clauses.

###  Single Column Value Counts
Now that we understand well what is going on under the hood with our simple but powerful `GROUP BY` clause - let’s apply it to the real complete dataset!

For the following code snippet - take special attention to the order of the syntax.

The `GROUP BY` must be used after the `FROM` statement otherwise you will get a syntax error and your SQL code will not run.

> What is the frequency of values in the rating column in the film table?
```sql 
SELECT
  rating,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating;
```
| rating | frequency |
|--|--|
|PG|194|
|NC-17|	210|
|PG-13|	223|
|G	|178|
|R	|195|


Let’s take this one step further and sort our `GROUP BY` output by using an `ORDER BY DESC` clause to arrange the rating values by highest to lowest occurences.

We often use this sorted output when analyzing data to see which values dominate the frequency count.

Take special attention that any `ORDER BY` clause must occur after the `GROUP BY` - these are the SQL syntax rules we must abide by!

```sql
SELECT
  rating,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```
|rating|frequency|
|--|--|
|PG-13|	223|
|NC-17|	210|
|R|	195|
|PG| 194|
|G|	178|

### Adding a Percentage Column
This following section is an optional bonus component as it will touch on some SQL techniques which we have yet to cover so far in this course!

Sometimes the frequency is just not enough to really understand the frequency at a quick glance, so we like to create an additional percentage column to our dataset.

There are actually various ways to perform this operation but I will keep things simple and show you the most efficient way using a modified window function combining both `SUM` and `COUNT` functions with an `OVER()` clause.

Take note of that `::NUMERIC` right after the `COUNT(*)` - this is to avoid the dreaded integer floor division which is covered in much more depth in the appendix!

```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```
| rating | frequency | percentage |
|--------|-----------|------------|
| PG-13  |   223     |  0.22367101303911735 |
| NC-17  |   210     |  0.21063189568706118 |
| PG     |   194     |  0.1945837512537613  |
| R      |   193     |  0.19358074222668004 |
| G      |   177     |  0.17753259779338014 |

This percentage decimal seems a bit ugly and annoying to look at it so to further spice it up - we can also multiple this percentage value by 100 and round it to 1 decimal place using a `ROUND` function.

Take note of the `100 *` is used in the query below and notice the comma within the round brackets for the `ROUND` function.

```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

| Rating | Frequency | Percentage |
|--------|-----------|------------|
| PG-13  |    223    |    22.37   |
| NC-17  |    210    |    21.06   |
| PG     |    194    |    19.46   |
| R      |    193    |    19.36   |
| G      |    177    |    17.75   |

## Counts For Multiple Column Combinations
Previously we have been looking at the unique values for just 1 column. In this section we will demonstrate how best to analyse combinations of 2+ columns.

The simplest way to do this is to apply the same `GROUP BY` clause and just specify additional columns in the grouping element expressions at the bottom of the SQL statement.

When we use `GROUP BY` on 2+ columns, the subsequent `COUNT` function will aggregate the records based off the unique combination of values in these columns instead of just a single 1.

It is quite common to see queries to profile specific columns by descending frequency like the following example.

Note that the syntax is very similar to the previous `GROUP BY` example but with the addition of more columns in both the `SELECT` statement and the following `GROUP BY` clause.

> What are the 5 most frequent `rating` and `category` combinations in the `film_list` table?

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY frequency DESC
LIMIT 5;
```

### Using Positional Numbers Instead of Column Names
This is actually quite an important note because you will run into this sooner or later in any SQL situation!

Some SQL developers like to refer to target columns used in `GROUP BY` and `ORDER BY` clauses by the positional number that the columns appear in the `SELECT` statement.

Be mindful that this is usually just a stylistic choice made by different developers in different teams and there is no right or wrong when it comes to this!

For example using our previous code snippet with just a `GROUP BY` clause to demonstrate what we mean by this:

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2;
```
Although this may look quite clean - often times it could become actually harder to read and very prone to making mistakes when you are writing the code!

This is especially the case when there is some funky stuff going on with non-sequential ordering within the `ORDER BY` clause like the following example:

**Not so great example**

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY 2,1,3 DESC
LIMIT 5;
```
I would recommend sticking with complete column names where possible, sometimes you can use numbers in the `GROUP BY` clause for a huge amount of columns instead of listing them all out explicitly - but be sure to be very clear with your expression inputs for the `ORDER BY` clause to maximise readibility and comprehension for anyone reading your code!

**Not so great example**

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 5;
```

**OK example**

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1,2
ORDER BY frequency DESC
LIMIT 5;
```

**Ideal example**

```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY frequency DESC
LIMIT 5;
```

## Exercises
These exercises will take a quick look at the other tables within the dvd_rentals schema.

1. Which `actor_id` has the most number of unique `film_id` records in the `dvd_rentals.film_actor` table?

```sql
SELECT actor_id, 
  COUNT( DISTINCT film_id ) AS unique_number_film_id 
FROM `dvd_rentals.film_actor`
GROUP BY actor_id
ORDER BY COUNT( DISTINCT film_id ) DESC
LIMIT 5;
```
Answer: 107

2. How many distinct `fid` values are there for the 3rd most common `price` value in the `dvd_rentals.nicer_but_slower_film_list` table?

```sql
SELECT price, 
  COUNT(DISTINCT fid) as distinct_fid 
FROM `dvd_rentals.nicer_but_slower_film_list`
GROUP BY price
ORDER BY COUNT(DISTINCT fid);
```

Answer: 323

3. How many unique `country_id` values exist in the `dvd_rentals.city` table?

```sql
SELECT COUNT(DISTINCT country_id) 
FROM `dvd_rentals.city`;
```

Answer: 109

4. What percentage of overall `total_sales` does the Sports `category` make up in the `dvd_rentals.sales_by_film_category` table?

```sql
SELECT category, 
  ROUND(100 * CAST(total_sales AS DECIMAL)/ SUM(total_sales) OVER (), 2) AS percentage 
FROM `dvd_rentals.sales_by_film_category`
GROUP BY category, total_sales
ORDER BY percentage DESC;
```


5. What percentage of unique `fid` values are in the Children `category` in the `dvd_rentals.film_list` table?

```sql
SELECT category, 
  ROUND(100 * COUNT(DISTINCT fid)/ SUM(COUNT(DISTINCT fid)) OVER (), 2) AS percentage 
FROM `dvd_rentals.film_list`
GROUP BY category;
```