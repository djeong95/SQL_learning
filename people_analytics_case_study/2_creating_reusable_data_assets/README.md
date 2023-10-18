# 1. Reusable Data Assets

When we are dealing with large datasets and with other team members in a work environment - there is often pressure to produce data analysis outputs faster and more efficiently, reducing the amount of new code that has to be written and also reducing the total raw amount of data computations and calculations.

This is exactly what we will focus on in this tutorial as we cover the important concept of database views, and we will also touch upon User Defined Functions (also known as stored procedures in T-SQL flavours such as SQL-Server or Teradata.

But firstly - what is a view and how is different to what we’ve been using so far in this course?

A word of warning about this tutorial - it is a bit heavier on the theoretical side of SQL so I apologise in advance if this knowledge is a bit dry!

# 2. What is a View?

Up until this point so far in the Serious SQL course - we’ve mostly focused on creating temporary tables and making use of CTEs to help us structure our queries and generate interim inputs.

These aren’t the only tools we have in our SQL reproducibility toolkit to generate interim datasets that we can refer to later on in our SQL scripts and data analysis - we also have the humble VIEW and the versatile MATERIALIZED VIEW that we can use to spice up our SQL scripts to improve our efficiency and productivity.

Whilst our temporary tables and CTEs actually run the queries we specify and materialises the records inside our new temporary tables when we run our CREATE TEMP TABLE step - a database view does not actually run the queries but instead - it saves the SQL query as a reference which only gets ran when the view is used in a downstream query.

So if we already have temporary tables and CTEs - why do we need something else?

Let’s take a look at our case study data where we’ll need to correct the young intern’s mistake of incorrectly entering our date values!

# 3. Date Column Updates

In our previous tutorial - we’ve identified a few tables where we will need to take action and make some changes to the date columns.

The following tables and their respective columns are listed below to adjust the date forward by 18 years except for the cases when any of the date columns has the arbitrary end date 9999-01-01.

To effectively handle these two date cases - we must avoid updating records where this value is present for certain columns, something we will demonstrate once we start diving into our SQL scripts to do this!

## 3.1. Target Tables

We need to update the employee.employees table for both the birth_date and hire_date columns.

The following tables require the from_date and to_date to be updated:

1. employees.department_employee
1. employees.department_manager
1. employees.salary
1. employees.title

Let’s now carefully proceed to update these tables - but of course, there is a catch!

# Updating Tables
So let’s say our client, HR Analytica very specifically DO NOT want their source data to be modified for any reason.

This is actually quite common for many companies as they would like to keep data integrity of their “source” data as clear and raw as possible. This sometimes does mean that mistakes that happen are left as they are!

So if we are given this constraint: we must not make any changes at all to the tables within the employee schema - we need to think laterally to continue with our case study.

There are a few options we could take at this point - ranging from simple through to more involved, but potentially more scalable, reproducible and also efficient!

## 4.1. Option 1 - Temporary Tables

We could stick with what we know and simply create a few temporary tables with the updated datasets and make the fixes explicitly through our SQL scripts.

This is totally a valid option and would likely do the trick - but let’s throw an additional spanner into the works to mess things up further…

Let’s say that we do not want to always re-create our temporary tables everytime we want to simply query our updated datasets. As with many things in life - there is a cost involved!

Everytime we run our CREATE TEMP TABLE steps from our SQL scripts - there is some computation going on in the background.

So if we wanted to create a few temp table versions of our tables by appending a temp_ in front of each table reference like so:

Note: we also make a copy of the employees.department table just for completeness and uniformity in our approach!

```sql

-- employee
DROP TABLE IF EXISTS temp_employee;
CREATE TABLE temp_employee AS
SELECT * FROM employees.employee;

-- temp department employee
DROP TABLE IF EXISTS temp_department;
CREATE TEMP TABLE temp_department AS
SELECT * FROM employees.department;

-- temp department employee
DROP TABLE IF EXISTS temp_department_employee;
CREATE TEMP TABLE temp_department_employee AS
SELECT * FROM employees.department_employee;

-- department manager
DROP TABLE IF EXISTS temp_department_manager;
CREATE TEMP TABLE temp_department_manager AS
SELECT * FROM employees.department_manager;

-- salary
DROP TABLE IF EXISTS temp_salary;
CREATE TEMP TABLE temp_salary AS
SELECT * FROM employees.salary;

-- title
DROP TABLE IF EXISTS temp_title;
CREATE TEMP TABLE temp_title AS
SELECT * FROM employees.title;
```

Now we’ll need to actually go in and update some of our table values - we’ve seen this in our previous marketing analytics case study, the only additional piece below is the extra WHERE filters to only update records where there is no 9999-01-01 values for the to_date column.

Note how there are multiple UPDATE steps for each table - one statement for each column that needs to be changed. We also need to use the + INTERVAL '18 YEARS' syntax to update our date values.

```sql
-- update temp_employee
-- ??? Why don't we need to check the date = '9999-01-01' for these columns
UPDATE temp_employee
SET hire_date = hire_date + INTERVAL '18 YEARS';

UPDATE temp_employee
SET birth_date = birth_date + INTERVAL '18 YEARS';

-- update temp_department_employee
-- ??? Why don't we need to check the date = '9999-01-01' for this column
UPDATE temp_department_employee SET
from_date = from_date + INTERVAL '18 YEARS';

UPDATE temp_department_employee
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update temp_department_manager
UPDATE temp_department_manager
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE temp_department_manager
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update temp_salary
UPDATE temp_salary
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE temp_salary
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update temp_title
UPDATE temp_title
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE temp_title
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';
```

## 4.2. Option 2 - New Schema
So now let’s say that we’re going to pass on the temporary table method - and luckily we have the ability to create an entirely new schema in our database!

We can now run commands very similar to our CREATE TEMP TABLE statements previously - but this time we can remove the TEMP part and write the data directly into our newly created adjusted_employees schema.

Note how we can also drop the schema if it exists just like our tables - we just need to add the CASCADE option to the end of it so it will look for all of the different tables and dependent views inside the schema and help us drop them too - needless to say - don’t ever try this on a production schema at work unless you want to get hurt by your local DBA!

```SQL
DROP SCHEMA IF EXISTS `younglapalma.adjusted_employees` CASCADE;
CREATE SCHEMA `younglapalma.adjusted_employees`;

-- employee
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.employee`;
CREATE TABLE `younglapalma.adjusted_employees.employee` AS
SELECT * FROM `younglapalma.employees.employee`;

-- department employee
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.department`;
CREATE TABLE `younglapalma.adjusted_employees.department` AS
SELECT * FROM `younglapalma.employees.department`;

-- department employee
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.department_employee`;
CREATE TABLE `younglapalma.adjusted_employees.department_employee` AS
SELECT * FROM `younglapalma.employees.department_employee`;

-- department manager
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.department_manager`;
CREATE TABLE `younglapalma.adjusted_employees.department_manager` AS
SELECT * FROM `younglapalma.employees.department_manager`;

-- salary
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.salary`;
CREATE TABLE `younglapalma.adjusted_employees.salary` AS
SELECT * FROM `younglapalma.employees.salary`;

-- title
DROP TABLE IF EXISTS `younglapalma.adjusted_employees.title`;
CREATE TABLE `younglapalma.adjusted_employees.title` AS
SELECT * FROM `younglapalma.employees.title`;

-- update employee
UPDATE `younglapalma.adjusted_employees.employee`
SET hire_date = DATE_ADD(hire_date, INTERVAL 18 YEAR) 
WHERE TRUE;

UPDATE `younglapalma.adjusted_employees.employee`
SET birth_date = DATE_ADD(birth_date, INTERVAL 18 YEAR)
WHERE TRUE;

-- update adjusted_employees.department_employee
UPDATE `younglapalma.adjusted_employees.department_employee` 
SET from_date = DATE_ADD(from_date, INTERVAL 18 YEAR)
WHERE TRUE;

UPDATE `younglapalma.adjusted_employees.department_employee` 
SET to_date = DATE_ADD(to_date, INTERVAL 18 YEAR)
WHERE to_date <> '9999-01-01';

-- update department_manager
UPDATE `younglapalma.adjusted_employees.department_manager` 
SET from_date = DATE_ADD(from_date, INTERVAL 18 YEAR)
WHERE TRUE;

UPDATE `younglapalma.adjusted_employees.department_manager`
SET to_date = DATE_ADD(to_date, INTERVAL 18 YEAR)
WHERE to_date <> '9999-01-01';

-- update salary
UPDATE `younglapalma.adjusted_employees.salary`
SET from_date = DATE_ADD(from_date, INTERVAL 18 YEAR)
WHERE TRUE;

UPDATE `younglapalma.adjusted_employees.salary`
SET to_date = DATE_ADD(to_date, INTERVAL 18 YEAR)
WHERE to_date <> '9999-01-01';

-- update title
UPDATE `younglapalma.adjusted_employees.title`
SET from_date = DATE_ADD(from_date, INTERVAL 18 YEAR)
WHERE TRUE;

UPDATE `younglapalma.adjusted_employees.title`
SET to_date = DATE_ADD(to_date, INTERVAL 18 YEAR)
WHERE to_date <> '9999-01-01';
```

All of the steps are exactly the same as before - just now the schema reference is changed and all tables will be permanent tables instead of temporary tables - this helps us achieve a few things:

1. We no longer need to re-run the temporary tables every time we fire up a new SQL session and
1. Everyone else who has access to our newly created adjusted_employees schema can query our new tables

This is going to save us huge amounts of time and resources when the datasets blow up to billions of rows - but there is still one glaring issue…

What happens if we get new employees who join the company and their records enter into the database - or an employee gets a raise and subsequently new records are inserted into the original source employees.salary table?

One simple solution is to drop our existing tables we created in the adjusted_employees schema on a regular basis and simply re-run our table creation statements from above - but I’m sure you can already see the issue with this approach already!

The cost would be enormous if the tables were substantially large!

There must be another way? Well…there are at least 3 more ways we can deal with this problem - let’s discuss the 1st and most inefficient way first before we make our way onto the main topic of this tutorial - views!

## 4.3. Delta Updates
So when we think of reusable data assets and updating for new data points that might come in naturally over the course of a day or month - there is a concept known as “the delta” or the change in a dataset.

Simply put - instead of dropping the table and rerunning the entire table creation step each time new fresh data comes in, we can look for the records which changed in the source data - and only update the rows or records in our downstream derived tables.

Let’s just take a very simple example of our very first employee id = 10001 Georgi Facello - if we inspect Georgi’s salary records - here’s what the current status of the records would look like:

```sql
SELECT *
FROM `younglapalma.adjusted_employees.salary`
WHERE employee_id = 10001
ORDER BY FROM_DATE DESC
LIMIT 5;
```
| employee_id | amount | from_date  | to_date    |
|-------------|--------|------------|------------|
| 10001       | 88,958 | 2020-06-22 | 9999-01-01 |
| 10001       | 85,097 | 2019-06-22 | 2020-06-22 |
| 10001       | 85,112 | 2018-06-22 | 2019-06-22 |
| 10001       | 84,917 | 2017-06-23 | 2018-06-22 |
| 10001       | 81,097 | 2016-06-23 | 2017-06-23 |

Now let’s say that on the 1st of January 2021 - Georgo was given a raise to $95,000, the dataset should be updated to the following - keep a close eye on the first 2 rows!

| employee_id | amount | from_date  | to_date    |
|-------------|--------|------------|------------|
| 10001       | 95,000 | 2021-01-01 | 9999-01-01 |
| 10001       | 88,958 | 2020-06-22 | 2021-01-01 |
| 10001       | 85,097 | 2019-06-22 | 2020-06-22 |
| 10001       | 85,112 | 2018-06-22 | 2019-06-22 |
| 10001       | 84,917 | 2017-06-23 | 2018-06-22 |

This is an example of what we would call a “Slowly Changing Dimension” or an SCD table for short in data engineering language - although it is slightly outside of our scope for this Serious SQL course - think of this as a brief primer on data engineering design!

When we look at each record in this salary table - we notice that there are from_date and to_date columns - these signify the periods of time when specific records are valid - hence this concept of “slowly changing” comes from.

Although our current datasets are just pure snapshots of a specific point in time - when we deal with real live datasets that are continuously updated with new data daily, hourly or even real-time - knowledge of these engineering designs are critical for effective and efficient SQL data processing.

When new data comes in - for example like our new $95,000 raise for Georgi - these SCD tables will often have what we call “UPSERT” statements - a combination of UPDATE and INSERT data manipulation statements.

In our example - we can see the UPDATE statement used for the second row where the to_date is being set to the latest date where the $88,958 salary is valid until, and an INSERT statement is used to add that first row with the $95,000 value and a new from_date for the date of the salary increase and to_date set to 9999-01-01.

So we can obviously see if there are any new records being inserted into our existing tables - however there is a catch (again…) - is it really efficient and scalable to do this with many many many new records? (spoiler alert - probably not!)

## 4.4. Updating Downstream Tables
Warning: this section is slightly heavier on data engineering so just keep this in mind as you’re reading!

So say for example - we notice some changes in our source datasets where new data is arriving, what are the ways we can check that there are new values coming in? Here are some suggestions:

1. Check the GROUP BY counts by from_date to see if any data has been input for the latest date - this is easy enough
1. Keep a track of which records are being updated in all the tables so we can update our downstream derived tables - this is not so easy…

Entire projects can be created to develop complex extract-transform-load ETL pipelines to perform this same function - so keep in mind that these complex systems do exist - but it is often not where we want to spend our time focusing when we are starting off in the data world.

Depending on how complex the data processing systems are - we could actually implement these steps into our pipeline and use other SQL tools such as user defined functions (UDFs) and event-based processing (triggers) to run all of these steps automagically so our datasets in the adjusted_employees schema that we created can be updated accordingly - however this leads to an excessive amount of complex SQL development…

Important Note: many ETL processes in large organisations actually follow this SCD and upsert pattern as it is inefficient to perform some of the upcoming view based methods on seriously seriously huge datasets as it’s just not feasible to keep huge copies of datasets in production systems - so please keep this in mind!

Although this is technically not a data engineering focused course, these are some of the more complex data design situations or scenarios you might encounter as you start working on more advanced data projects in general!

Here is a light pseudo-code for what one such process might look like for this employees example where a new day’s worth of data is loaded into the system:

For the UPDATE records:

1. For each table with to_date records - check which ones have a to_date set to today’s date
1. Identify the relevant from_date records which need to have the update applied - usually most SCD style tables are optimised or partitioned (stored) on these date columns to reduce the amount of data required to be scanned during update steps
1. Update relevant records in the downstream tables via some type of join condition (usually inner join) on the subset of relevant partitions

Done!

For the INSERT records:

1. Check the count of records added for each table with the from_date equal to the current date of data processing
1. Insert these new records directly into the downstream tables

Done!

More often than not - the update steps are the most annoying and difficult parts. Some data engineering design patterns may also remove the need to UPDATE records and will only operate with an INSERT only paradigm. This helps to keep the data reproducible as well as immutable and idempotent - unchangeable and able to be rebuilt from scratch respectively - these are very common terms used in data engineering to design robust and high integrity data assets!

OK - so because these steps are a bit too involved for what we’re after - surely there is a simpler method to obtain a similar result without expending all this mental energy just to add some new data records?

This is exactly where our database views come into the picture!

# 5. Database Views

As we’ve mentioned before - a database VIEW is essentially a reference which stores a query that can be ran and accessed just like any table - however the catch is that the data is not preprocessed and saved anywhere until the view is referenced in a query.

I can see the wheels turning in your mind as you figure out why this is a good solution for our updating data situation!

If we used views for our datasets instead of having permanent tables - we can effectively remove the need to create all those additional complex systems to check for changes and update all the downstream data sources accurately!

When we access our created views - the raw data will be requested only at that point in time - meaning that all of the newly updated and inserted records in our original source data will be available when we SELECT FROM our view. This is a very common approach for datasets which are constantly updated with new data - imagine a daily sales table which adds new transactions for example.

An additional benefit of using a VIEW instead of temporary tables is that the index information is retained from the base tables referenced in the view query. This sometimes leading to faster performance than storing datasets as permanent tables, and especially if you do not specify indexes for the new permanent tables (more on this later in the optimisation part of the Serious SQL course!)

```SQL
DROP SCHEMA IF EXISTS `younglapalma.v_employees` CASCADE;
CREATE SCHEMA `younglapalma.v_employees`;

-- department
DROP VIEW IF EXISTS `younglapalma.v_employees.department`;
CREATE VIEW `younglapalma.v_employees.department` AS
SELECT * FROM `younglapalma.employees.department`;

-- department employee
DROP VIEW IF EXISTS `younglapalma.v_employees.department_employee`;
CREATE VIEW `younglapalma.v_employees.department_employee` AS
SELECT
  employee_id,
  department_id,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
    END AS to_date
FROM `younglapalma.employees.department_employee`;

-- department manager
DROP VIEW IF EXISTS `younglapalma.v_employees.department_manager`;
CREATE VIEW `younglapalma.v_employees.department_manager` AS
SELECT
  employee_id,
  department_id,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
    END AS to_date
FROM `younglapalma.employees.department_manager`;

-- employee
DROP VIEW IF EXISTS `younglapalma.v_employees.employee`;
CREATE VIEW `younglapalma.v_employees.employee` AS
SELECT
  id,
  DATE_ADD(birth_date, INTERVAL 18 YEAR) AS birth_date,
  first_name,
  last_name,
  gender,
  DATE_ADD(hire_date, INTERVAL 18 YEAR) AS hire_date,
FROM `younglapalma.employees.employee`;

-- salary
DROP VIEW IF EXISTS `younglapalma.v_employees.salary`;
CREATE VIEW `younglapalma.v_employees.salary` AS
SELECT
  employee_id,
  amount,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
    END AS to_date
FROM `younglapalma.employees.salary`;

-- title
DROP VIEW IF EXISTS `younglapalma.v_employees.title`;
CREATE VIEW `younglapalma.v_employees.title` AS
SELECT
  employee_id,
  title,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
    END AS to_date
FROM `younglapalma.employees.title`;
```

| employee_id | amount | from_date               | to_date                 |
|-------------|--------|-------------------------|-------------------------|
| 10001       | 88,958 | 2020-06-22 00:00:00     | 9999-01-01 00:00:00     |
| 10001       | 85,097 | 2019-06-22 00:00:00     | 2020-06-22 00:00:00     |
| 10001       | 85,112 | 2018-06-22 00:00:00     | 2019-06-22 00:00:00     |
| 10001       | 84,917 | 2017-06-23 00:00:00     | 2018-06-22 00:00:00     |
| 10001       | 81,097 | 2016-06-23 00:00:00     | 2017-06-23 00:00:00     |

So we’ve covered how we can use views to simplify our queries and also obtain the most update version of the original source data by using database views to store our queries. Have you figured out what the catch is yet?

Everytime we hit a reference which is actually a view - we have to run the query. This has some cost implications - especially when some of these queries are massive (which they usually are!)

So there is one final thing we can look into to help solve our new issue - enter the materialized view!

# 6. Materialized Views

We can think of a materialized view as a combination of a permanent table and a regular view - many SQL developers say it’s the best of both worlds as you get the flexibility of a view with the efficiency of persisted (or materialized) data!

The SQL implementation of a materialized view is literally by using CREATE MATERIALIZED VIEW instead of CREATE VIEW - simple right!

The key difference is that materialized views are populated with data - in the same way that a CREATE TABLE or CREATE TEMP TABLE statement would also populate the table reference with data once they are ran.

There is also a catch with materialized views too - it seems like every single thing we learn in SQL always has edge cases or things to catch you out if you’re not careful!

