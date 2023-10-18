
# 1. HR Case Study Problem Solving
Now that we’ve fixed our data issue using some reproducible methods - let’s quickly recap what we’ve done so far and also revisit technical outputs we need to complete this HR analytics case study.

# 2. Background Recap
We’ve been tasked by our client HR Analytica to create 2 analytical views to power their dashboard outputs and further enable their in-house analytics team to generate additional reports and answer basic business questions using our data assets.

So far we have inspected the raw source data HR Analytica have shared with us inside the employees schema and we’ve applied corrections for a simple data input error by adding 18 years to each historic date record.

We now have a new schema mv_employees complete with like-for-like materialized view versions of each of the tables from the employees with all corrected data.

The solution for the exercise in our previous tutorial: we also needed to apply an explicit data type cast from the TIMESTAMP to a DATE after adding our interval of 18 years!

# 3. Data Generation
Please make sure you run the following SQL script directly in your SQLPad GUI otherwise you will not be able to run any of the following SQL code snippets later in this tutorial - the explicit type cast back to a DATE data type is also applied throughout this following code snippet below and we’ve also included the index creation steps to match our original datasets at the end of each view creation step.

## 3.1. Original Indexes
We can inspect the original indexes from our employees schema using the following query - UNIQUE indexes literally mean that each index value will be unique and there is no risk of duplicates when using either a single column or a combination of multiple columns as shown in the most right indexdef column values.

We will cover this in more detail and also the types of indexes we can create in the Additional SQL component part of the Serious SQL course - but for now we will use these original index definitions to help us recreate these indexes for our materialized views.

## 3.2. Materialized View Script
The following is the complete script we will need to use for the rest of this case study tutorial - note that all of the explicit date casts is applied throughout the code and the final index creation steps will be kept right at the end of the snippet below.

```sql
DROP SCHEMA IF EXISTS `younglapalma.mv_employees` CASCADE;
CREATE SCHEMA `younglapalma.mv_employees`;

-- department
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.department` AS
SELECT * FROM `younglapalma.employees.department`;

-- department employee
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.department_employee` AS
SELECT
  employee_id,
  department_id,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> DATE "9999-01-01" THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
  END AS to_date
FROM `younglapalma.employees.department_employee`;

-- department manager
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.department_manager` AS
SELECT
  employee_id,
  department_id,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> DATE "9999-01-01" THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
  END AS to_date
FROM `younglapalma.employees.department_manager`;

-- employee
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.employee` AS
SELECT
  id,
  DATE_ADD(birth_date, INTERVAL 18 YEAR) AS birth_date,
  first_name,
  last_name,
  gender,
  DATE_ADD(hire_date, INTERVAL 18 YEAR) AS hire_date
FROM `younglapalma.employees.employee`;

-- salary
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.salary` AS
SELECT
  employee_id,
  amount,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> DATE "9999-01-01" THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
  END AS to_date
FROM `younglapalma.employees.salary`;

-- title
CREATE OR REPLACE MATERIALIZED VIEW `younglapalma.mv_employees.title` AS
SELECT
  employee_id,
  title,
  DATE_ADD(from_date, INTERVAL 18 YEAR) AS from_date,
  CASE
    WHEN to_date <> DATE "9999-01-01" THEN DATE_ADD(to_date, INTERVAL 18 YEAR)
    ELSE to_date
  END AS to_date
FROM `younglapalma.employees.title`;

-------- BigQuery doesn't allow users to manually create indexes on tables as traditional RDBMS systems do ------
-- Index Creation 
-- NOTE: we do not name the indexes as they will be given randomly upon creation!
CREATE UNIQUE INDEX ON mv_employees.employee USING btree (id);
CREATE UNIQUE INDEX ON mv_employees.department_employee USING btree (employee_id, department_id);
CREATE INDEX        ON mv_employees.department_employee USING btree (department_id);
CREATE UNIQUE INDEX ON mv_employees.department USING btree (id);
CREATE UNIQUE INDEX ON mv_employees.department USING btree (dept_name);
CREATE UNIQUE INDEX ON mv_employees.department_manager USING btree (employee_id, department_id);
CREATE INDEX        ON mv_employees.department_manager USING btree (department_id);
CREATE UNIQUE INDEX ON mv_employees.salary USING btree (employee_id, from_date);
CREATE UNIQUE INDEX ON mv_employees.title USING btree (employee_id, title, from_date);



```

# 4. Technical Requirements
For this case study request - we’ve explicitly been asked to generate reproducible analytical views that the in-house HR Analytica team can access to answer more questions.

We will also be tasked with generating valid data points and insights for the following two data products:

1. A current view of the company, department and title level insights
1. A deep dive tool to investigate all information on a single employee

## 4.1. People Analytics Dashboard
An example company level dashboard that includes the following insights:

- Total number of employees
- Average company tenure in years
- Gender ratios
- Average payrise percentage and amount

We will also need to generate similar insights for both department and title levels where we need employee count and average tenure for each level instead of company wide numbers!

## 4.2. Employee Deep Dive

Below is a example employee deep dive output from our second output that includes the following insights for a single employee over time:

- See all the various employment history ordered by effective date including salary, department, manager and title changes
- Calculate previous historic payrise percentages and value changes
- Calculate the previous position and department history in months with start and end dates
- Compare an employee’s current salary, total company tenure, department, position and gender to the average benchmarks for their current position

# 5. Visual Examples

# 6. Current vs Historic
Firstly we need to address the key difference between our 2 technical requirements - namely the difference between a current snapshot vs a historic dataset.

The dashboard components are all based off current data whilst the deep dive employee analysis must be able to look at both current and the historic data.

This concept of current vs historic is super important when working with real world messy data - and especially when working with slowly changing dimensions (SCD) tables which are pretty much everywhere throughout all databases in medium to large companies!

Up until this point - perhaps you have encountered data which has date columns and lots of metrics - SCD tables we touched upon lightly in the previous tutorial but we are yet to really dig into exactly why they are useful!

Let’s revisit Georgi’s salary (employee_id = 10001) to reiterate this difference between current and historic before we roll into more SQL for our case study!

## 6.1. Georgi’s Salary Revisited

Let’s take another look at Georgi’s salary using our new materialized view version of the salary: mv_employees.salary

This mv_employees.salary is a perfect example of a slow-changing dimension table also known as a historic dataset where there is a “valid” period for each data point - the from_date and to_date columns signify the end of a specific period of time where the amount is active.

```sql
SELECT *
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001
ORDER BY from_date DESC
LIMIT 5;
```

## 6.2. Exercises
The following questions have solutions at the end of this tutorial in the appendix - please give these a try to test your understanding of this historic and current concept as we will be leveraging this throughout the rest of this tutorial heavily!

1. What was Georgi’s starting salary at the beginning of 2009?
```sql
SELECT *
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001 
AND '2009-01-01' BETWEEN from_date and to_date
ORDER BY from_date DESC
```
1. What is Georgi’s current salary?

```sql
SELECT *
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001 
AND CURRENT_DATE('America/Los_Angeles') BETWEEN from_date and to_date
ORDER BY from_date DESC
```
1. Georgi received a raise on 23rd of June in 2014 - how much of a percentage increase was it?
```sql
WITH cte AS (SELECT ROUND(100* (amount - LAG(AMOUNT) OVER (ORDER BY from_date))
  / (LAG(AMOUNT) OVER (ORDER BY from_date)) , 2)  AS percentage_difference
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001 
AND (
    from_date = '2014-06-23'
    OR to_date = '2014-06-23'
  )
ORDER BY from_date DESC)
SELECT *
FROM cte
WHERE percentage_difference IS NOT NULL;
```

1. What is the dollar amount difference between Georgi’s salary at date '2012-06-25' and '2020-06-21'

```sql
SELECT t2.amount_2020 - t1.amount_2012 AS dollar_difference
FROM (SELECT amount AS amount_2012
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001
AND '2012-06-25' BETWEEN from_date and to_date
) AS t1
CROSS JOIN
(SELECT amount AS amount_2020
FROM `younglapalma.mv_employees.salary`
WHERE employee_id = 10001
AND '2020-06-21' BETWEEN from_date and to_date
) AS t2
```

# 7. Data Exploration


Given our improved understanding of the differences between a current and historical view of the data - let’s now investigate our ERD to see how we can combine our tables together to generate our base datasets and also how we can apply specific strategies to obtain a current or historic view for our downstream data analysis.

In particular - let’s try to map our various columns from each table to the different technical requirements we need for our insights.

We’ll first tackle the current dashboard requirements before moving onto the employee deep dive.

## 7.1. Entity Relationship Diagram

<img width="465" alt="image" src="https://github.com/djeong95/Reddit_wsb_datapipeline/assets/102641321/fb767d45-d28c-4577-bf59-0bf4d8d26d5d">

## 7.2. Employee Mapping

If we bundle together our required insights - we can see some information will be contained inside the employee dataset, namely the gender and the hire_date columns which we can use for the gender ratio and company tenure insights respectively.

Let’s also inspect the GROUP BY counts by id to confirm that there will only ever be a single row per employee id in our employee dataset:

```sql
WITH id_cte AS (
  SELECT
    id,
    COUNT(*) AS row_count
  FROM `younglapalma.mv_employees.employee`
  GROUP BY id
)
SELECT
  row_count,
  COUNT(DISTINCT id) AS employee_count
FROM id_cte
GROUP BY row_count
ORDER BY row_count;
```

We can see that there are 300,024 unique employees from our employee table so is that the final count of employees that are currently working for the company?

Let’s dive a bit deeper into some of the other tables to inspect how many unique employee_id values are valid - we’ll need to inspect our to_date columns from our other tables.

## 7.3. Valid Data Points

Since we require a current view for our initial dashboard reporting piece - let’s quickly perform additional GROUP BY distinct count sof the employee_id field by the from_date values for our relevant tables.

For our examples - since we will need that current view, we should only be inspecting the values where the to_date = '9999-01-01' - our arbitrary end date for each record in our datasets.

### 7.3.1. Salary

Since we’ll need to obtain the current salary values to calculate our various statistical metrics - let’s first take a look at the salary table:

```sql
SELECT
  to_date,
  COUNT(*) AS record_count,
  COUNT(DISTINCT employee_id) AS employee_count
FROM `younglapalma.mv_employees.salary`
GROUP BY 1
ORDER BY 1 DESC;
```

| `to_date`  | `record_count` | `employee_count` |
|------------|----------------|------------------|
| 9999-01-01 | 240,124        | 240,124          |
| 2020-08-01 | 686            | 686              |
| 2020-07-31 | 641            | 641              |
| 2020-07-30 | 673            | 673              |
| 2020-07-29 | 679            | 679              |

There are less records for our to_date = '9999-01-01' field? Why might this be the case - was this expected?

Let’s also check how many total distinct employee_id values we have in this table:

```sql
SELECT
  COUNT(DISTINCT employee_id) AS distinct_count
FROM mv_employees.salary;
```
distinct_count: 300024

Alright! We are back on track with our original employee count value we were expecting!

Now that we’ve covered our salary records - let’s now dive a bit deeper into our department, department_employee and title tables to obtain further data points for our insights.

#### 7.3.1.1. Exercises

All answers to the following questions are also included at the end of this tutorial in the appendix!

> 5. For each employee who no longer has a valid salary data point - which year had the most employee churn and how many employees left that year?

```sql
WITH cte AS (SELECT
  employee_id,
  MAX(to_date) AS final_date
FROM `younglapalma.mv_employees.salary`
GROUP BY employee_id
)

SELECT
  EXTRACT(YEAR FROM final_date) AS churn_year,
  COUNT(*) AS employee_count
FROM cte
WHERE final_date != '9999-01-01'
GROUP BY churn_year
ORDER BY employee_count DESC;
```

This final salary to_date should be the theoretical last day that an employee was paid - so we can think of it as their final day before the left the company for greener pastures!

> 6. What is the average latest percentage and dollar amount change in salary for each employee who has a valid current salary record?

We actually need this output for the final case study solution but it can be calculated directly with just the salary table!

```sql
WITH salary_cte AS (SELECT
  employee_id,
  to_date,
  amount, 
  LAG(amount) OVER (PARTITION BY employee_id ORDER BY from_date) AS previous_amount,
FROM `younglapalma.mv_employees.salary`
)
SELECT
  ROUND(AVG( 100 * (amount - previous_amount)/ previous_amount), 2) as average_latest_percentage,
  ROUND(AVG(amount - previous_amount), 2) as average_dollar_change
FROM salary_cte;
```

Hint: you will need to make use of a specific window function which can compare rows within each PARTITION BY grouping of records!

### 7.3.2. Department Information

Let’s take a look at the department_employee table first:

```sql
SELECT * FROM mv_employees.department_employee LIMIT 5;
```

# Employee Department Assignment

| `employee_id` | `department_id` | `from_date` | `to_date`    |
|---------------|-----------------|-------------|--------------|
| 10001         | d005            | 2004-06-26  | 9999-01-01   |
| 10002         | d007            | 2014-08-03  | 9999-01-01   |
| 10003         | d004            | 2013-12-03  | 9999-01-01   |
| 10004         | d004            | 2004-12-01  | 9999-01-01   |
| 10005         | d003            | 2007-09-12  | 9999-01-01   |


If we apply the same analysis to inspect the distribution of to_date values - we can see a similar story to the salary table.

```sql
SELECT
  to_date,
  COUNT(*) AS record_count,
  COUNT(DISTINCT employee_id) AS employee_count
FROM mv_employees.department_employee
GROUP BY 1
ORDER BY 1 DESC
LIMIT 5;
```

# Table Summary

| `to_date`   | `record_count` | `employee_count` |
|-------------|----------------|------------------|
| 9999-01-01  | 240124         | 240124           |
| 2020-08-01  | 20             | 20               |
| 2020-07-31  | 28             | 28               |
| 2020-07-30  | 32             | 32               |
| 2020-07-29  | 27             | 27               |

This 240,124 value seems to be present here too.

Let’s also quickly check how many unique employee_id records there are in this department_employee table:

```sql
SELECT
  COUNT(DISTINCT employee_id) AS distinct_count
FROM mv_employees.department_employee;
```

distinct_count: 300024


We can also notice how this department_employee gives us the department_id only and not the name of the department that we’d actually need for our outputs - let’s take a look also at the department table to see if we need to apply any analysis on this table too:

```sql
 SELECT * FROM mv_employees.department;
```
# Department Names

| `id`  | `dept_name`          |
|------|----------------------|
| d009 | Customer Service     |
| d005 | Development          |
| d002 | Finance              |
| d003 | Human Resources      |
| d001 | Marketing            |
| d004 | Production           |
| d006 | Quality Management   |
| d008 | Research             |
| d007 | Sales                |

It looks like we don’t need to do anything with our department table as there are only 9 unique rows. Let’s move onto our title table next!

### 7.3.3. Title Information

We will also need to generate the summary statistics values and counts for employees for our title level dashboard report so let’s run the same valid snapshot analysis for our title table:

```sql
SELECT
  to_date,
  COUNT(*) AS record_count,
  COUNT(DISTINCT employee_id) AS employee_count
FROM `younglapalma.mv_employees.title`
GROUP BY 1
ORDER BY 1 DESC
LIMIT 5;
```

| `to_date`   | `record_count` | `employee_count` |
|-------------|----------------|------------------|
| 9999-01-01  | 240124         | 240124           |
| 2020-08-01  | 40             | 40               |
| 2020-07-31  | 65             | 65               |
| 2020-07-30  | 53             | 53               |
| 2020-07-29  | 52             | 52               |

We have that same 240,124 valid records number here also!

And again - let’s confirm our distinct employee_id count.

```sql
SELECT
  COUNT(DISTINCT employee_id) AS distinct_count
FROM `younglapalma.mv_employees.title`;
```

distinct_count: 300024

Now that we’ve completed our analysis into each individual tables - let’s take a look at how we can incorporate these tables so we can proceed with the analysis steps for our case study outputs.

# 8. Join Historic Tables

One key skill we really need to focus on is the ability to join multiple tables with multiple historic data points which all have different validity at different points in time.

This is one of the main differentiators between a junior/intermediate data analyst to an advanced level data analyst so please make sure this next following section is really well understood before moving on!

We have seen all of those from_date and to_date columns throughout this case study in many of the tables except the static employee and department tables.

Now we will demonstrate how to combine these tables using 2 separate methods and compare the results to help further our understanding.

## 8.1. Naive Joins

Firstly we will naively join our tables using the foreign key columns identified by our ERD without taking into account the from_date and to_date columns - then we will perform a simple COUNT(*) on the resulting joint table output to see how many rows and inspect a few results to check for inconsistencies.

We will be using a lot of column aliases as we have multiple tables with the same from_date and to_date column names so we will be prepending each of those columns with their specific table source, e.g. salary_from_date and salary_to_date

Please make sure you run the following SQL scripts for our different joins below in the SQLPad GUI so you can check some of the data outputs yourself as this is really is the key to understanding why we shouldn’t do this with our historic datasets!

Just a word of caution also - the following tables outputs in the following section will be really really wide as there are lots of columns included in the final join table - another strong reason for running these queries directly in your SQLPad GUI so you can scroll and see the outputs yourself on a larger screen!

We will be using the employee table as the base and perform multiple INNER JOIN to all of our tables via the join keys shown in the ERD and ommitting the department_manager table for now - we will be using this table during the second deep dive employee analysis, however it needs something extra to make it usable for our analysis!

```sql
DROP TABLE IF EXISTS `younglapalma.mv_employees.naive_join_table`;
CREATE TABLE `younglapalma.mv_employees.naive_join_table` AS
SELECT
  employee.id,
  employee.birth_date,
  employee.first_name,
  employee.last_name,
  employee.gender,
  employee.hire_date,
  -- we do not need title.employee_id as employee.id is already included!
  title.title,
  title.from_date AS title_from_date,
  title.to_date AS title_to_date,
  -- same goes for the title.employee_id column
  salary.amount,
  salary.from_date AS salary_from_date,
  salary.to_date AS salary_to_date,
  -- same for department_employee.employee_id
  -- shorten department_employee to dept for the aliases
  department_employee.department_id,
  department_employee.from_date AS dept_from_date,
  department_employee.to_date AS dept_to_date,
  -- we do not need department.department_id as it is already included!
  department.dept_name
FROM `younglapalma.mv_employees.employee` AS employee
INNER JOIN `younglapalma.mv_employees.title` AS title
  ON employee.id = title.employee_id
INNER JOIN `younglapalma.mv_employees.salary` AS salary
  ON employee.id = salary.employee_id
INNER JOIN `younglapalma.mv_employees.department_employee` AS department_employee
  ON employee.id = department_employee.employee_id
-- NOTE: department is joined only to the department_employee table!
INNER JOIN `younglapalma.mv_employees.department` AS department
  ON department_employee.department_id = department.id;
```


### 8.1.2. Exercise

> 7. Why do we use INNER JOIN instead of LEFT JOIN for our naive join situation - does it make any difference?

Hint: why did we look at those distinct employee_id counts for each table?

### 8.1.3. Inspecting Individuals

#### 8.1.3.1. Georgi Facello

Let’s also inspect good old Georgi’s records again by applying a WHERE id = 10001 and sorting by his salary_to_date descending to see if it matches up with what we are expecting:

```sql
SELECT *
FROM `younglapalma.mv_employees.naive_join_table`
WHERE id = 10001
ORDER BY salary_to_date DESC;
```

Here we can see that there seems to be no issue with Georgi’s records as we can see his title and department does not change at all throughout his entire career at the company - the only things that are changing is his salary - which we can see changing with the salary_from_date and salary_to_date records.

#### 8.1.3.2. Leah Anguita
Let’s look at another employee with a bit more career activity to see if we can find any data issues - we apply another basic where filter WHERE id = 11669 in the script below:

```sql
SELECT *
FROM `younglapalma.mv_employees.naive_join_table`
WHERE id = 11669
ORDER BY salary_to_date DESC;
```

Ok - we have a lot of stuff going on here, and we may well have some issues too!

Let’s further drill down into Leah’s current salary record to see if we can identify exactly what is going on with our naive left join dataset:

```sql
SELECT *
FROM `younglapalma.mv_employees.naive_join_table`
WHERE id = 11669
  AND salary_to_date = '9999-01-01'
ORDER BY salary_to_date DESC;
```

Here we can spot out first issue - there seems to be duplicates in both the title and dept_name related data leading not 4 records when there should only be 1 because Leah shouldn’t have 2 different titles or be in 2 different departments at the same time!

What can we do to remove all of these issues? Luckily there is a simply way!

### 8.1.4. Current Records Only

One way to deal with these sorts of duplicates in historical dimensions is to simply filter out anything which is not current.

In our example - we will need to only need to extract the title and dept_name for the current time which we can easily do using a basic filter WHERE <to_date> = '9999-01-01'

Let’s apply this simple fix to just Leah’s record to see if we return the only row we are expecting!

```sql
SELECT *
FROM naive_join_table
WHERE id = 11669
  AND salary_to_date = '9999-01-01'
  AND title_to_date = '9999-01-01'
  AND dept_to_date = '9999-01-01';
```

### 8.1.5. Assessment
So we can see that our naive join method seems to introduce many many duplicates into our dataset which we don’t need for our basic current snapshot analysis that we need for our initial dashboard view for the case study.

In fact - this is actually a very common mistake made by beginner SQL practitioners who do not understand the concept of historic vs current data, so please take special attention whenever you see any columns which represent the “slow changing dimension” of an effective/from and an expiry/to date!

So let’s now improve our original naive solution and apply further snapshot logic to only capture the current records that we need for our analytics dashboard.

## 8.2. Current Record Join
Let’s improve our original naive join method by applying those same filters in our previous query to view Leah’s current records.

We simply need to add that same block of WHERE filters directly at the bottom of our previous query:
```sql
WHERE salary_to_date = '9999-01-01'
  AND title_to_date = '9999-01-01'
  AND dept_to_date = '9999-01-01'
```

Note how these filters are applied in the WHERE step with the original materialized view reference columns and not afterwards in a CTE - also we do not specify the filtering criteria within the JOIN criteria - this is for a few reasons

1. If we were to use an additional CTE - we would need to calculate ALL of the 5 million rows prior to the filtering occuring (slow)
1. We would have to write more code which is verbose and more difficult to read if we include all the filters within the ON conditions
1. Philosophically - we want to perform the join and then apply the filters, so the logic is easier to understand when written this way

```sql
DROP TABLE IF EXISTS `younglapalma.mv_employees.current_join_table`;
CREATE TABLE `younglapalma.mv_employees.current_join_table` AS
SELECT
  employee.id,
  employee.birth_date,
  employee.first_name,
  employee.last_name,
  employee.gender,
  employee.hire_date,
  -- we do not need title.employee_id as employee.id is already included!
  title.title,
  title.from_date AS title_from_date,
  title.to_date AS title_to_date,
  -- same goes for the title.employee_id column
  salary.amount,
  salary.from_date AS salary_from_date,
  salary.to_date AS salary_to_date,
  -- same for department_employee.employee_id
  -- shorten department_employee to dept for the aliases
  department_employee.department_id,
  department_employee.from_date AS dept_from_date,
  department_employee.to_date AS dept_to_date,
  -- we do not need department.department_id as it is already included!
  department.dept_name
FROM `younglapalma.mv_employees.employee` AS employee
INNER JOIN `younglapalma.mv_employees.title` AS title
  ON employee.id = title.employee_id
INNER JOIN `younglapalma.mv_employees.salary` AS salary
  ON employee.id = salary.employee_id
INNER JOIN `younglapalma.mv_employees.department_employee` AS department_employee
  ON employee.id = department_employee.employee_id
-- NOTE: department is joined only to the department_employee table!
INNER JOIN `younglapalma.mv_employees.department` AS department
  ON department_employee.department_id = department.id
-- NOTE: we use the original table.column references to help the optimizer
-- We DO NOT want to use a CTE for this extra filter step!!!
WHERE salary.to_date = '9999-01-01'
  AND title.to_date = '9999-01-01'
  AND department_employee.to_date = '9999-01-01';
```

Let’s take a quick look at the counts to see if this matches with what we are expecting!

```sql
SELECT
  COUNT(*) AS row_count
FROM current_join_table;
```

row_count: 240124

This seemed to have worked well and now we should be able to access all of the relevant current data for our employees - but we have a few more requirements for our analytics dashboard - that average salary increase, and also we still have yet to cover our final historical employee deep dive!

Let’s try to answer a few exercise questions before we take a quick look at that salary increase step as it’s something we’ve already covered before in our Window Functions section of the course!

### 8.2.1. Exercises

All of these questions are based off the current dataset that we’ve just created - I would suggest you to save your queries somewhere because we might just be using these somewhere later on in our case study…

Write SQL queries to calculate the average salary for the following segments:

- Years of tenure: you can calculate tenure = current date - hire date in years
- Title
- Department
- Gender
Using the outputs or scripts from the above - answer the following questions:

8. What is the average salary of someone in the Production department?
```sql
SELECT
  AVG(amount) AS average_salary
FROM current_join_table
WHERE dept_name = 'Production';
```

8. Which position has the highest average salary?

```sql
SELECT
  title,
  AVG(amount) AS avg_salary
FROM current_join_table
GROUP BY title
ORDER BY avg_salary DESC;
```
8. Which department has the lowest average salary?

```sql
SELECT
  dept_name,
  AVG(amount) AS avg_salary
FROM current_join_table
GROUP BY dept_name
ORDER BY avg_salary;
```

## 8.3. Average Salary Increase

One of the current reporting dashboards insights that we need to generate is the average latest payrise percentage split by gender.

Since we already have the gender records directly in our current join table - we just need to calculate the difference in salary so we can calculate our percentage difference.

We can employ one of our trusty window function tools to solve this problem - LAG

Firstly - let’s prototype our solution with just the salary table for our favourite employee Georgi. Since we only need to have a single record with the employee_id record so we can join back onto our current snapshot view - we can apply the same to_date filter to make sure we capture the latest records and the LAG salary amount for our calculations:

Do you remember why we can’t use the WHERE filter directly without the CTE step below?

```sql
WITH lag_data AS (
SELECT
  employee_id,
  to_date,
  amount AS current_amount,
  LAG(amount) OVER (PARTITION BY employee_id ORDER BY to_date) AS previous_amount
FROM mv_employees.salary
WHERE employee_id = 10001
)
SELECT
  employee_id,
  current_amount - previous_amount AS salary_amount_change,
  100 * (current_amount - previous_amount) / previous_amount::NUMERIC AS salary_pc_change
FROM lag_data
WHERE to_date = '9999-01-01';
```


We will re-use this logic once we compile our entire SQL solution in the following tutorial - now let’s move onto the juicy part of this tutorial, Now that we’ve tackled how to easily obtain the current snapshot data - let’s now move our attention to the employee deep dive.

# 9. Historic View

One of the shortfalls of the current snapshot view is that we won’t be able to inspect the full history of a certain employee that we need for our second case study output. We will need to improve upon our existing SQL join to incorporate these historic data points for all our employees in the dataset.

We will need to populate the various events that have occured for each employee to show the timeline view on the left as well compare our target employee’s details to the average benchmark for their tenure, position/title, department and gender, as well as obtaining who their current manager is.

Let’s split up our analysis into a few steps so we can better understand the process for generating our historic data view.

1. Identify the current details required in the deep dive report:
- Name, gender, age, birthday, company tenure
    - Department name, department tenure, department manager
    - Salary amount
2. Perform comparison calculations for the following:
- Latest salary change (this should be the same as the current company metric too)
    - Salary comparison by tenure, position/title, department and gender benchmarks (remember the exercise question?)
3. The last 5 employee events - we’ll need to identify the following types of events:
- Salary increase/decrease with the new salary amount, $ change amount and percentage change
    - Title change from old to new title
    - Department transfer from old to new department name
    - Reporting Line Change from old to new manager (not shown in the template!)

## 9.1. Effective Expiry

Up until now we’ve been dealing with all of the from_date and to_date columns separately for our salary, department_employee and title materialized views as we’ve only needed to apply the final basic filter of WHERE to_date = '9999-01-01' to identify the final current record for our records.

However for this challenge - we will need to clearly split out when each event occurs and also clearly capture the state of all the other metrics to validate that nothing else has changed.

Honestly - the first time I was faced with similar problems in the workplace I was in a state of total confusion because I had no idea what to do!

For this example - the easiest way would be to investigate a single customer and figure out exactly what happened for them - before coming up with new effective and expiry date columns to capture the overall validity of each record.

Let’s revisit our new employee spotlight Leah!

### 9.1.1. Leah’s Events
If we were to capture all of Georgi’s data using the simple naive join method - it will mean we will show all of his records.

Let’s explicitly only select the following columns to narrow our focus:

- title, title_from_date, title_to_date
- amount, salary_from_date, salary_to_date
- dept_name, dept_from_date and dept_to_date

```sql
SELECT
  title,
  title_from_date,
  title_to_date,
  amount,
  salary_from_date,
  salary_to_date,
  dept_name,
  dept_from_date,
  dept_to_date
FROM naive_join_table
WHERE id = 11669
ORDER BY
  title_to_date DESC,
  dept_to_date DESC,
  salary_to_date DESC;
```
# Employee Records

| title           | title_from_date | title_to_date | amount | salary_from_date | salary_to_date | dept_name        | dept_from_date | dept_to_date  |
|-----------------|-----------------|---------------|--------|------------------|----------------|------------------|----------------|---------------|
| Senior Engineer | 2020-05-12      | 9999-01-01    | 47373  | 2020-05-11       | 9999-01-01     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 47046  | 2019-05-11       | 2020-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43681  | 2018-05-11       | 2019-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43930  | 2017-05-12       | 2018-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43577  | 2016-05-12       | 2017-05-12     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 41183  | 2015-05-12       | 2016-05-12     | Customer Service | 2019-06-12     | 9999-01-01    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 47373  | 2020-05-11       | 9999-01-01     | Production       | 2015-05-12     | 2019-06-12    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 47046  | 2019-05-11       | 2020-05-11     | Production       | 2015-05-12     | 2019-06-12    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43681  | 2018-05-11       | 2019-05-11     | Production       | 2015-05-12     | 2019-06-12    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43930  | 2017-05-12       | 2018-05-11     | Production       | 2015-05-12     | 2019-06-12    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 43577  | 2016-05-12       | 2017-05-12     | Production       | 2015-05-12     | 2019-06-12    |
| Senior Engineer | 2020-05-12      | 9999-01-01    | 41183  | 2015-05-12       | 2016-05-12     | Production       | 2015-05-12     | 2019-06-12    |
| Engineer        | 2015-05-12      | 2020-05-12    | 47373  | 2020-05-11       | 9999-01-01     | Customer Service | 2019-06-12     | 9999-01-01    |
| Engineer        | 2015-05-12      | 2020-05-12    | 47046  | 2019-05-11       | 2020-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Engineer        | 2015-05-12      | 2020-05-12    | 43681  | 2018-05-11       | 2019-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Engineer        | 2015-05-12      | 2020-05-12    | 43930  | 2017-05-12       | 2018-05-11     | Customer Service | 2019-06-12     | 9999-01-01    |
| Engineer        | 2015-05-12      | 2020-05-12    | 43577  | 2016-05-12       | 2017-05-12     | Customer Service | 2019-06-12     | 9999-01-01    |
| Engineer        | 2015-05-12      | 2020-05-12    | 41183  | 2015-05-12       | 2016-05-12    

Here we can see very clearly there are basic separations between Leah’s company activity based off the way we’ve sorted the data:

1. Leah had a title change from Engineer to Senior Engineer starting from 2020-05-12
1. He also had salary changes in May from 2015 each year through to 2020 with his latest increase on the 2020-05-11 from $47,046 to $47,373
1. He changed departments from Productuction to Customer Service on 2019-06-12

So our challenge is - how can we perform this same analysis using only SQL and not needing to do these manual steps?

Additionally - there are going to be a ton of redundant unnecessary data points which are not valid, what sort of filter can we apply to remove these records in a similar way that we removed all our non-current records in the current snapshot view?

There is one logical leap that we need to do in order to solve both of these problem - we’ll need to perform some row-wise operations on the date columns!

## 9.2. Row-wise Date Calculations
This is actually a difficult concept to grasp at first - but we will want to perform MAX and MIN calculations on the from_date and to_date columns in order to calculate the latest and earliest effective and expiry dates respectively for each row of data.

However - we can’t really do this with multiple column inputs into the MAX and MIN functions - instead we will need to use their rowwise equivalent functions GREATEST and LEAST

This is best explained by inspecting a few example rows from the different cases:

### 9.2.1. Current Data Points
Scenario 1 is the most current valid data point with to_date = '9999-01-01' but different from_date columns for title, salary and department.

| title           | title_from_date | title_to_date | amount | salary_from_date       | salary_to_date | dept_name        | dept_from_date | dept_to_date  |
|-----------------|-----------------|---------------|--------|-----------------------|----------------|------------------|----------------|---------------|
| Senior Engineer | 2020-05-12      | 9999-01-01    | 47373  | 2020-05-11 00:00:00   | 9999-01-01     | Customer Service | 2019-06-12     | 9999-01-01    |

Here we actually want to take the latest from_date as logically - it is the most recent point in time where all of these data points were valid. We can think of this calculation as calculating the latest effective date.

Naturally - the expiry date for this specific data point will be the 9999-01-01 as it is all equal in the to_date column values.

For this transformation - we will need to apply the GREATEST function on our from_date columns to acquire this latest effective date - we will also apply the LEAST function to our to_date columns but this will not actually change any of the values as all the to dates are equal like we mentioned above:

```sql
SELECT
  title,
  dept_name,
  amount,
  GREATEST(
    title_from_date,
    salary_from_date,
    dept_from_date
  ) AS effective_date,
  LEAST(
    title_to_date,
    salary_to_date,
    dept_to_date
  ) AS expiry_date,
  title_from_date,
  salary_from_date,
  dept_from_date,
  title_to_date,
  salary_to_date,
  dept_to_date
FROM `younglapalma.mv_employees.naive_join_table`
WHERE id = 11669
  AND title_to_date = '9999-01-01'
  AND dept_to_date = '9999-01-01'
  AND salary_to_date = '9999-01-01';
```


If we now apply the same logic to all of the records and not just those with the to_date = '9999-01-01' record - we can start identifying the next filter logic we need to apply.

### 9.2.2. All Data Points

If we simply remove the WHERE filters on the dates we had in our previous query - we’ll be able to see a few more rows of data from Leah’s records.

Additionally if we think about our effective and expiry paradigm - we shouldn’t have have any records where the effective date is after the expiry date, if we now apply this filter and then order by our effective_date field - we can see if we need to apply any further filters to our analysis.

```sql
SELECT *
FROM (
  SELECT
    title,
    dept_name,
    amount,
    GREATEST(
      title_from_date,
      salary_from_date,
      dept_from_date
    ) AS effective_date,
    LEAST(
      title_to_date,
      salary_to_date,
      dept_to_date
    ) AS expiry_date,
    title_from_date,
    salary_from_date,
    dept_from_date,
    title_to_date,
    salary_to_date,
    dept_to_date
  FROM `younglapalma.mv_employees.naive_join_table`
  WHERE id = 11669) subquery
WHERE effective_date <= expiry_date
ORDER BY effective_date;
```


| title           | dept_name        | amount | effective_date | expiry_date | title_from_date | salary_from_date | dept_from_date | title_to_date | salary_to_date | dept_to_date  |
|-----------------|------------------|--------|----------------|-------------|-----------------|------------------|----------------|---------------|----------------|---------------|
| Engineer        | Production       | 41183  | 2015-05-12     | 2016-05-12  | 2015-05-12      | 2015-05-12       | 2015-05-12     | 2020-05-12    | 2016-05-12     | 2019-06-12    |
| Engineer        | Production       | 43577  | 2016-05-12     | 2017-05-12  | 2015-05-12      | 2016-05-12       | 2015-05-12     | 2020-05-12    | 2017-05-12     | 2019-06-12    |
| Engineer        | Production       | 43930  | 2017-05-12     | 2018-05-11  | 2015-05-12      | 2017-05-12       | 2015-05-12     | 2020-05-12    | 2018-05-11     | 2019-06-12    |
| Engineer        | Production       | 43681  | 2018-05-11     | 2019-05-11  | 2015-05-12      | 2018-05-11       | 2015-05-12     | 2020-05-12    | 2019-05-11     | 2019-06-12    |
| Engineer        | Production       | 47046  | 2019-05-11     | 2019-06-12  | 2015-05-12      | 2019-05-11       | 2015-05-12     | 2020-05-12    | 2020-05-11     | 2019-06-12    |
| Engineer        | Customer Service | 47046  | 2019-06-12     | 2020-05-11  | 2015-05-12      | 2019-05-11       | 2019-06-12     | 2020-05-12    | 2020-05-11     | 9999-01-01    |
| Engineer        | Customer Service | 47373  | 2020-05-11     | 2020-05-12  | 2015-05-12      | 2020-05-11       | 2019-06-12     | 2020-05-12    | 9999-01-01     | 9999-01-01    |
| Senior Engineer | Customer Service | 47373  | 2020-05-12     | 9999-01-01  | 2020-05-12      | 2020-05-11       | 2019-06-12     | 9999-01-01    | 9999-01-01     | 9999-01-01    |


This looks great now and we can easily see that thereis a clear chronological order of events with no overlaps between the effective and expiry dates as we wanted!

Now we need to focus on our final piece of the puzzle for these events - how can we classify these events to match with our requested event types?

### 9.2.3. Exercise
Notice how I’ve used a subquery in the above query:

Is there any difference between this and the common CTE that we’ve seen throughout this course?
Hint: you may want to try running an EXPLAIN on the query to see if the execution plan!

## 9.3. Logic Clauses
For this section of the tutorial - we will need to apply some logic to identify what specific types of events occured for our employees after we’ve combined their data points and assigned the effective_date and expiry_date columns.

Let’s create an employee_events table with more information to simplify our analysis using the previous logic that we worked on with the GREATEST and LEAST functions calculated on each row with the WHERE effective_date <= expiry_date filter applied also:

```sql
DROP TABLE IF EXISTS `younglapalma.mv_employees.employee_events`;
CREATE TABLE `younglapalma.mv_employees.employee_events` AS
SELECT * FROM (
  SELECT
    id,
    title,
    dept_name,
    amount,
    GREATEST(
      title_from_date,
      salary_from_date,
      dept_from_date
    ) AS effective_date,
    LEAST(
      title_to_date,
      salary_to_date,
      dept_to_date
    ) AS expiry_date
  FROM `younglapalma.mv_employees.naive_join_table`
) subquery
WHERE effective_date <= expiry_date
ORDER BY effective_date;
```

For this part - let’s revisit what sort of logical flags we need to apply again to define our events:

> The last 5 employee events - we’ll need to identify the following types of events: * Salary increase/decrease with the new salary amount, $ change amount and percentage change * Title change from old to new title * Department transfer from old to new department name * Reporting Line Change from old to new manager (not shown in the template!)

For all of these events there is a common thread - there must be a comparison to the previous state of the employee. It’s clear that something has changed as each row in our events table however - we will need to compare them somehow, if only we had a useful function to do just that :)

Let’s demonstrate how to apply this logic using our super useful tool - the LAG function!

## 9.4. Salary Events
In the following example - we will implement the logic for the salary increase and decrease events before we tackle the other events.

For our LAG window function we will need to compare the amount again just like in our previous example - we just need to be careful with our PARTITION BY and ORDER BY clauses.

We will also use the LAG window function with a CASE WHEN so we can differentiate between an increase or decrease event by comparing the amount and the LAG(amount) values for each record - there is also one question - what do we do when the amount values are the same? I’m going to leave them as NULL for now - but see if you can figure out why before the next section.

For the following SQL snippet - let’s look at Leah’s records again just so we can nail the logic before moving onto multiple employees at once!

```sql
WITH lag_data AS (
SELECT
  id,
  title,
  dept_name,
  amount,
-- we have `id` in our table not employee_id!
  LAG(amount) OVER (
    PARTITION BY id
    ORDER BY effective_date
  ) AS previous_amount,
  effective_date,
  expiry_date
FROM employee_events
-- we have `id` in our table not employee_id!
WHERE id = 11669
)
SELECT
  id,
  title,
  dept_name,
  amount,
  previous_amount,
  CASE
    WHEN amount > previous_amount
      THEN 'Salary Increase'
    WHEN amount < previous_amount
      THEN 'Salary Decrease'
    ELSE NULL
  END AS event_name,
  effective_date,
  expiry_date
FROM lag_data
ORDER BY effective_date;
```


| id    | title           | dept_name        | amount | previous_amount | event_name      | effective_date | expiry_date |
|-------|-----------------|------------------|--------|-----------------|-----------------|----------------|-------------|
| 11669 | Engineer        | Production       | 41183  | null            | null            | 2015-05-12     | 2016-05-12  |
| 11669 | Engineer        | Production       | 43577  | 41183           | Salary Increase | 2016-05-12     | 2017-05-12  |
| 11669 | Engineer        | Production       | 43930  | 43577           | Salary Increase | 2017-05-12     | 2018-05-11  |
| 11669 | Engineer        | Production       | 43681  | 43930           | Salary Decrease | 2018-05-11     | 2019-05-11  |
| 11669 | Engineer        | Production       | 47046  | 43681           | Salary Increase | 2019-05-11     | 2019-06-12  |
| 11669 | Engineer        | Customer Service | 47046  | 47046           | null            | 2019-06-12     | 2020-05-11  |
| 11669 | Engineer        | Customer Service | 47373  | 47046           | Salary Increase | 2020-05-11     | 2020-05-12  |
| 11669 | Senior Engineer | Customer Service | 47373  | 47373           | null            | 2020-05-12     | 9999-01-01  |


See how the null values actually line up to other types of events - we can see that the final row is a change in title and the 3rd last row is a switch in department.

We can add more LAG fields into our lag_data CTE and use them in additional conditions on our CASE WHEN statement to identify these 2 events also - but before we do that, we don’t seem to have our department manager details in this cut of the data do we?

Let’s go and fix that first before we come back and apply our logic for the title and department transfers.

## 9.5. Manager Issues

In our previous naive_join_table we omitted the department_manager view on purpose for this specific part!

This is slightly trickier than normal as we need to obtain the manager’s name for our final visual output - and the only place to get the first_name and last_name values are from the employee view.

```sql
SELECT * FROM mv_employees.department_manager LIMIT 5;
```

How can we refer to this same view when we’ve already using in our join conditions for our employees - effectively using it to base all of our analysis?

Simple - we just join onto it again! But this time we can join onto a different column to avoid duplicating our analysis.

Let’s add onto our previous naive_join_table logic with our effective_date and expiry_date components and then also add on our department_manager and another join onto the employee view to grab the name data.

Also we will concatenate first and last names to create a full_name column and also a manager column.

And let’s throw in the LAG values for our amount, title, dept_name and manager_name columns:

```sql
DROP TABLE IF EXISTS `younglapalma.mv_employees.historic_join_table`;
CREATE TABLE `younglapalma.mv_employees.historic_join_table` AS 
WITH join_data AS (
  SELECT
    employee.id AS employee_id,
    employee.birth_date,
    CONCAT(employee.first_name, ' ', employee.last_name) AS full_name,
    employee.gender,
    employee.hire_date,
    title.title,
    salary.amount AS salary,
    department.dept_name AS department,
    -- use the `manager` aliased version of employee table for manager
    CONCAT(manager.first_name, ' ', manager.last_name) AS manager,
    GREATEST(
      title.from_date,
      salary.from_date,
      department_employee.from_date,
      department_manager.from_date
    ) AS effective_date,
    LEAST(
      title.to_date,
      salary.to_date,
      department_employee.to_date,
      department_manager.to_date
    ) AS expiry_date
  FROM `younglapalma.mv_employees.employee` AS employee
  INNER JOIN `younglapalma.mv_employees.title` AS title
    ON employee.id = title.employee_id
  INNER JOIN `younglapalma.mv_employees.salary` AS salary
    ON employee.id = salary.employee_id
  INNER JOIN `younglapalma.mv_employees.department_employee` AS department_employee
    ON employee.id = department_employee.employee_id
  -- NOTE: department is joined only to the department_employee table!
  INNER JOIN `younglapalma.mv_employees.department` AS department
    ON department_employee.department_id = department.id
  -- add in the department_manager information onto the department table
  INNER JOIN `younglapalma.mv_employees.department_manager` AS department_manager
    ON department.id = department_manager.department_id
  -- join again on the employee_id field to another employee for manager's info
  INNER JOIN `younglapalma.mv_employees.employee` AS manager
    ON department_manager.employee_id = manager.id
)
SELECT
  employee_id,
  birth_date,
  full_name,
  gender,
  hire_date,
  title,
  LAG(title) OVER w AS previous_title,
  salary,
  LAG(salary) OVER w AS previous_salary,
  department,
  LAG(department) OVER w AS previous_department,
  manager,
  LAG(manager) OVER w AS previous_manager,
  effective_date,
  expiry_date
FROM join_data
WHERE effective_date <= expiry_date
-- define window frame
WINDOW
  w AS (PARTITION BY employee_id ORDER BY effective_date);
```


| employee_id | birth_date | full_name   | gender | hire_date  | title           | previous_title | salary | previous_salary | department       | previous_department | manager        | previous_manager | effective_date | expiry_date |
|-------------|------------|-------------|--------|------------|-----------------|----------------|--------|-----------------|------------------|---------------------|----------------|------------------|----------------|-------------|
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | null           | 41183  | null            | Production       | null                | Oscar Ghazalie | null             | 2015-05-12     | 2016-05-12  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 43577  | 41183           | Production       | Production          | Oscar Ghazalie | Oscar Ghazalie   | 2016-05-12     | 2017-05-12  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 43930  | 43577           | Production       | Production          | Oscar Ghazalie | Oscar Ghazalie   | 2017-05-12     | 2018-05-11  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 43681  | 43930           | Production       | Production          | Oscar Ghazalie | Oscar Ghazalie   | 2018-05-11     | 2019-05-11  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 47046  | 43681           | Production       | Production          | Oscar Ghazalie | Oscar Ghazalie   | 2019-05-11     | 2019-06-12  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 47046  | 47046           | Customer Service | Production          | Yuchang Weedman| Oscar Ghazalie   | 2019-06-12     | 2020-05-11  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Engineer        | Engineer       | 47373  | 47046           | Customer Service | Customer Service    | Yuchang Weedman| Yuchang Weedman  | 2020-05-11     | 2020-05-12  |
| 11669       | 1975-03-03 | Leah Anguita | M      | 2004-04-07 | Senior Engineer | Engineer       | 47373  | 47373           | Customer Service | Customer Service    | Yuchang Weedman| Yuchang Weedman  | 2020-05-12     | 9999-01-01  |

This effectively brings us to the end of this entire tutorial for the snapshot vs historic - we will cover the rest of the logical blocks and events, comparisons to benchmarks and all the other metrics for both the current dashboards and the employee deep dive in the following final solution tutorial.

# 10. Summary
In this tutorial we covered the following:

1. Identify and recreate original indexes for materialized views using the pg_indexes table
1. Compare current and historic values for our SCD (slow changing dimension) tables
1. Naively join our tables and apply a WHERE filter to extract current values with to_date = '9999-01-01'
1. Use LAG window function to compare previous records for changing information
1. Use rowwise GREATEST and LEAST functions to calculate effective and expiry dates for each record in our join tables
1. Use CASE WHEN statements to identify employee events based off changing lag values
1. Join onto 2 instances of the employee materialized view to obtain manager information without introducing duplicates
1. Notice how throughout this tutorial - we have been creating temporary tables instead of using views - this will be implemented properly in the next tutorial as we cover the final SQL solution in a portfolio project style!

