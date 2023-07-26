# How To Deal With Duplicates

Duplication of data might occur due to many different reasons, often these are related to upstream issues with data collection, data processing or even manual data entry mistakes - think of all those times you made typos when filling in an Excel spreadsheet!

In SQL there are a few different ways to deal with these duplicate records:

- Remove them in a `SELECT` statement
- Recreating a “clean” version of our dataset
- Identify exactly which rows are duplicated for further investigation or
- Simply ignore the duplicates and leave the dataset alone

Firstly, we need to figure out if we even have duplicate records in our table!


## Detecting Duplicate Records

Before we think about removing duplicate records - we need a systematic way to check whether our table has any duplicate records first!

There are actually a few different ways to do this but I will show you first a simple method before jumping to the most efficient solution.

The first ingredient for this recipe is the basic record count for our table - plain and simple using the `COUNT(*)` just like we did above.

```sql
SELECT COUNT(*)
FROM health.user_logs;
```

**count: 43891**

The next step is to apply this same `COUNT(*)` method - but on a deduplicated version of the dataset.

So in the end - we actually need to remove duplicates first to even figure out if we have duplicates…talk about redundancy!

## Remove All Duplicates

Although that previous roundabout notion of removing duplicates to find out if we have duplicates is slightly confusing - actually removing duplicates from a table is not confusing at all.

We can apply the same `DISTINCT` keyword we’ve used previously to do this like so:

```sql
SELECT DISTINCT *
FROM health.user_logs;
```

I’m not going to show the output for this query because we’re not quite finished yet with this transformation.

The next part of the duplicate identification recipe is to count the number of rows in this deduplicated dataset.

So, you’re probably thinking - no worries, all I need to do is something like this:

```sql
SELECT COUNT(DISTINCT *)
FROM health.user_logs;
```

Unfortunately for us - PostgreSQL does not allow for this style of `COUNT(DISTINCT *)` syntax like we can use on a single column!

There are some other flavours of SQL that actually allow for this syntax namely Teradata, however it is not a standard operation which we can use everywhere.

However, it is relatively straightforward to get around this!

There are a few schools of thought on how exactly this `COUNT DISTINCT` should be done - I’ll show you a few different ways and then suggest you which one I recommend to use so you can gain exposure to more SQL techniques.

Take special note of the differences between the SQL syntax for each method.

## Subqueries

A subquery is essentially a query within a query - in this case we want to use our `DISTINCT *` output in the innermost nested query as a data source for the outer query.

Take note of the syntax - especially the `AS` component because subqueries must always have an alias!

```sql
SELECT COUNT(*)
FROM (
  SELECT DISTINCT *
  FROM health.user_logs
) AS subquery;
```
**count: 31004**

## Common Table Expression

We have briefly seen CTEs in an earlier tutorial but here we will dive a little bit deeper and also demonstrate another way to use them.

### What is a CTE?
CTE stands for `Common Table Expression` - and when we compare it to something simple like Excel, we can think of CTEs as transformations applied to raw data inside an existing Excel sheet.

A CTE is a SQL query that manipulates existing data and stores the data outputs as a new reference, very similar to storing data in a new temporary Excel sheet (following the Excel analogy!)

Subsequent CTEs can refer to existing datasets, as well as previously generated CTEs. This allows for quite complex nested queries and operations to be performed, whilst keeping the code nice and readable!

We will be using CTEs a lot throughout this course - if you find yourself being overwhelmed by this, just remember that these are just like new Excel sheets generated with new data. You will definitely get used to this after we solve 100+ questions using them :)

### CTE Example Query

```sql
WITH deduped_logs AS (
  SELECT DISTINCT *
  FROM health.user_logs
)
SELECT COUNT(*)
FROM deduped_logs;
```
**count: 31004**

## Temporary Tables
Whilst we could use subqueries and CTEs to capture the output directly in a single query - we can also create a temporary table with only the unique values of our dataset after we run the `DISTINCT` query.

This is a very common approach when you know that you will only be analyzing the deduplicated dataset, and you will ignore the original one with duplicates.

The main benefit of using temporary tables is removing the need to always run the same `DISTINCT` command everytime you want to run a query on the deduplicated records.

Temporary tables can also be used with indexes and partitions to speed up performance of our SQL queries - something which we will cover later!

There is a lengthier process to dealing with temporary tables, which is a multi-step ordeal!

- First we run a `DROP TABLE IF EXISTS` statement to clear out any previously created tables

In practice - we like to make sure all the temporary tables we create are “clean” and often we will clear out any tables with the same target name as our new temporary table, just in case or better safe than to be sorry!

Be very super careful when running this following `DROP TABLE` statement as you can’t really undo things when you drop an actual table…

Well… I guess you can always restart your Docker environment and a fresh version of this database will be ready for you - but just know that you can really do some damage if you carelessly drop production tables in the workplace. You’ve been warned!!!

`DROP TABLE IF EXISTS deduplicated_user_logs;`
Although the table might not exist yet - this `DROP TABLE` should still run fine because we used the `IF EXISTS` parameter which will stop it from throwing an error.

After running the above query - you should see the message `Query finished No rows returned` in the SQLPad GUI output.

- Next let’s create a new temporary table using the results of the query below

```sql
CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;
```
Take note of the syntax for the first line in the above query, particularly the AS at the end of the line which tells SQL to use the output from the following query to populate the newly created table - more on this soon!

This query will also not return any output, just like the previous DROP TABLE IF EXISTS statement.

- Let’s query this newly created temporary table to make sure we can access it
```sql
SELECT *
FROM deduplicated_user_logs
LIMIT 10;
```

- Finally, let’s run that same `COUNT` on this deduplicated temp table to confirm it did the right thing!

```sql
SELECT COUNT(*)
FROM deduplicated_user_logs;
```
**count: 31004**

So I’m guessing that you might be wondering - why are temporary tables temporary?

Temporary tables are automagically deleted once a session is shut down i.e hitting control + c on the terminal instance running the docker-compose command.

This is good for our purposes but sometimes you might actually want to persist some of your newly created tables also!

In general - I would recommend getting familiar with temporary tables as there are usually less restrictions in regards to access and space/storage issues.

The best bet is to always ask your local database administrator for further guidance as they might have a preference on which schemas or databases you should be using to persist your tables.

To be honest though - the difference in functionality when starting off in SQL is almost negligible and in-fact I would argue that temporary tables are considered best practice for writing efficient SQL at production grade!

## Recommendation

So now you’re probably thinking - ok thanks for showing 3 ways to get a distinct count of records from a dataset Danny - but which one should I use?

I would suggest the following decision making process by asking yourself this question:

> Will I need to use the deduplicated data later?

If yes - opt for temporary tables.

If no - CTEs are your friend.

Usually we would not recommend subqueries as they are less readable than CTEs - making it more difficult for others to quickly understand your code!

However - there is an exception to this rule. if your coworkers are using subqueries throughout their SQL scripts, sometimes it is best to fit in first before trying to make any changes.

I learned this first hand many many times trying to force others to code how I like to code…and in a nutshell - it never really went down well - even if I had a valid point!

So in closing - try to pick your battles wisely and use temp tables and CTEs wherever you can :)

## Comparing Counts

OK so going back to our original purpose of detecting the presence of duplicate records in our dataset - can you figure out the logical conclusion to this exercise?

We now have the row counts of the original table **43,891** and of our deduplicated table **31,004**

It’s pretty safe to say that we have some deduplicate records!

By comparing the counts of the original and the deduplicated, we can prove the presence of duplicates.

But is that all we can do with these duplicate values…what if we wanted to know more?

Also what do you think of this labourious effort of having to calculate counts for both tables and manually comparing them?

What if there was another way to do this…

# Identifying Duplicate Records

This takes us to the final part of this tutorial where we will aim to write a final SQL statement to efficiently return us all rows which are duplicated in our target table.

However, we run into another issue - what exactly do we want for our final output?

Let’s take this dummy example table for instance to describe what we want to see from our query:

**Dummy Example Table**
| id | log_date   | measure      | measure_value | systolic | diastolic |
|----|------------|--------------|---------------|----------|-----------|
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |
| b  | 2020-01-03 | blood_glucose| 224           | 0        | 0         |
| c  | 2020-01-01 | blood_glucose| 110           | 0        | 0         |
| c  | 2020-01-02 | blood_glucose| 100           | 0        | 0         |
| c  | 2020-01-03 | blood_glucose| 291           | null     | null      |

**All Duplicate Rows**

If we were to return all of the duplicate rows we would expect the first 5 rows from our dataset:

| id | log_date   | measure      | measure_value | systolic | diastolic |
|----|------------|--------------|---------------|----------|-----------|
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |

**Unique Duplicate Rows**

And further to this, if we wanted to only see the unique records which were duplicated, the output might just be the distinct of the above table!

| id | log_date   | measure      | measure_value | systolic | diastolic |
|----|------------|--------------|---------------|----------|-----------|
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |

**Duplicate Counts**

Often we will also be interested in how many times a single record appears in the dataset - when the count is greater than 1 it signifies that the record is duplicated!

| id | log_date   | measure      | measure_value | systolic | diastolic | count |
|----|------------|--------------|---------------|----------|-----------|-------|
| a  | 2020-01-01 | blood_glucose| 93            | null     | null      |   3   |
| b  | 2020-01-02 | blood_glucose| 135.11691     | 0        | 0         |   2   |


## Group By Counts On All Columns

As we can see - there are different ways to approach this simple problem of returning duplicate rows depending on what is required.

We can implement a single SQL statement that will help us achieve both outputs and also something even more useful!

The trick is to use a `GROUP BY` clause which has every single column in the grouping element and a `COUNT` aggregate function - this is an elegant solution to quickly find the unique combinations of all the rows.

Hang on a second…isn’t that exactly what the `DISTINCT` keyword is supposed to do?

Yes - that’s correct! The only difference here is that we can also apply the aggregate function with the `GROUP BY` clause to find the counts for the unique combinations which we touched upon in the previous tutorial!

Elegant, right? Well - here is the SQL statement to do this:

```sql
SELECT
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
ORDER BY frequency DESC
```

*Note that output is limited to first 10 and last 10 rows by descending frequency and some column names and values are truncated to make the table look nicer!*

| id       | log_date   | measure       | value | sys | dia | frequency |
|----------|------------|---------------|-------|-----|-----|-----------|
| 054250c  | 2019-12-06 | blood_glucose | 401   | null| null| 104       |
| 054250c  | 2019-12-05 | blood_glucose | 401   | null| null| 77        |
| 054250c  | 2019-12-04 | blood_glucose | 401   | null| null| 72        |
| 054250c  | 2019-12-07 | blood_glucose | 401   | null| null| 70        |
| 054250c  | 2020-09-30 | blood_glucose | 401   | null| null| 39        |
| 054250c  | 2020-09-29 | blood_glucose | 401   | null| null| 24        |
| 054250c  | 2020-10-02 | blood_glucose | 401   | null| null| 18        |
| 054250c  | 2019-12-10 | blood_glucose | 140   | null| null| 12        |
| 054250c  | 2019-12-11 | blood_glucose | 220   | null| null| 12        |
| 054250c  | 2020-04-15 | blood_glucose | 236   | null| null| 12        |
| ...      | ...        | ...           | ...   | ... | ... | ...       |
| 054250c6 | 2020-01-10 | blood_glucose | 93    | null| null| 1         |
| d696925d | 2020-06-02 | blood_glucose | 307   | 0   | 0   | 1         |
| 29874263 | 2020-03-24 | blood_glucose | 135   | 0   | 0   | 1         |
| bca46f69 | 2020-05-30 | blood_glucose | 120   | 0   | 0   | 1         |
| d696925d | 2020-05-07 | blood_glucose | 224   | 0   | 0   | 1         |
| fba135f6 | 2020-04-18 | blood_glucose | 110   | 0   | 0   | 1         |
| 832e45ce | 2019-11-24 | blood_glucose | 100   | 0   | 0   | 1         |
| 054250c6 | 2020-05-07 | blood_glucose | 291   | null| null| 1         |
| 576fdb52 | 2020-06-20 | blood_glucose | 98    | null| null| 1         |
| 6d9c9595 | 2020-02-06 | blood_pressure| 122   | 122 | 81  | 1         |

Notice how the frequency for some of these values is 1 - whilst some are greater than 1?

This is exactly how we know which unique combinations of the columns have duplicates!

Now there is a final piece of the puzzle which will help us extract the duplicate records only.

## Having Clause For Unique Duplicates

Now the final step is to use the `HAVING` clause to further trim down our output by applying a condition on the same `COUNT(*)` expression we were using for the frequency column we created in our previous query.

Since we only want the duplicate records to be returned - we would like that `COUNT(*)` value to be greater than 1.

I will also show you an extra hack here for our SQL query where we can actually use the same * as we’ve used previously to replace all those columns that we specified in the `SELECT` expressions in our query.

This is totally optional and non-standard as it’s actually more difficult to read than listing out all the columns in the order that they appear in the dataset, but it does make for slightly neater looking code so I thought I might as well show you another option!

Note that we cannot use the same `*` within the `GROUP BY` grouping elements as it will throw a syntax error.

To drill our previous knowledge - let’s go ahead and create a new temporary table called `duplicate_record_counts` in our following query.

```sql
-- Don't forget to clean up any existing temp tables!
DROP TABLE IF EXISTS unique_duplicate_records;

CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;

-- Finally let's inspect the top 10 rows of our temp table
SELECT *
FROM unique_duplicate_records
LIMIT 10;
```

| id        | log_date   | measure       | measure_value | systolic | diastolic |
|-----------|------------|---------------|---------------|----------|-----------|
| 054250c69 | 2020-10-04 | blood_glucose | 170           | null     | null      |
| 054250c69 | 2019-12-15 | blood_glucose | 79            | 0        | 0         |
| 054250c69 | 2020-10-05 | blood_glucose | 323           | null     | null      |
| 054250c69 | 2020-01-03 | blood_glucose | 128           | null     | null      |
| 054250c69 | 2020-01-01 | blood_glucose | 214           | null     | null      |
| 054250c69 | 2020-01-04 | blood_glucose | 131           | null     | null      |
| 0f7b13f3f | 2020-08-17 | blood_glucose | 155           | 0        | 0         |
| 054250c69 | 2020-10-02 | blood_glucose | 182           | null     | null      |
| 054250c69 | 2019-12-25 | blood_glucose | 170           | null     | null      |
| 054250c69 | 2020-05-04 | blood_glucose | 285           | null     | null      |

## Retaining Duplicate Counts

Let’s say for example - we want to know which exact records are duplicated - but also how many times they appeared. The output would look exactly like our output from the Group By Counts On All Columns section - but just without the rows where the frequency count was equal to 1.

We can use our CTE approach with a `WHERE` filter condition to do this - note how the CTE component looks exactly same as the previous query - the only difference is the following `SELECT` statement.

For completeness - let’s also limit to the first 10 rows ordered by the output by descending `frequency` counts to inspect the duplicates with the greatest number of occurences.

```sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT *
FROM groupby_counts
WHERE frequency > 1
ORDER BY frequency DESC
LIMIT 10;
```

*Some column names and values are truncated to make the table look nicer!*

So now that we’ve found all the duplicate rows of data there is just one final question we need to ask ourselves:

> Do we actually need to care about duplicate values?

## Ignoring Duplicate Values

By now you are probably sick of this talk about duplicates and how to deal with them - why am I now telling you that maybe we should just ignore them?

Let’s think back to the context of our dataset here.

> For context, this real world messy dataset captures data taken from individuals logging their health measurements via an online portal throughout the day.

> For example, multiple measurements can be taken on the same day at different times, but you may notice this information is missing as the log_date column does not show timestamp values! 

> Welcome to the real world of messy datasets :)
