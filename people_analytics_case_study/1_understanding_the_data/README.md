# 1. Data Exploration

Just like our previous marketing analytics case study - let’s first investigate the entity relationship diagram and take a look at the first few observations from each table to better understand what data we will be using for this exercise.

# 2. Entity Relationship Diagram

![image](https://github.com/djeong95/SQL_learning/assets/102641321/f9b968e2-f2f6-4167-a118-49d7052fd7f3)

# 3. Database Tables Overview

All of the data required for this case study lives within the employees schema within our Docker environment, however there is one glaring issue with all of the data…

The date fields in the HR Analytica tables were accidentally input with incorrect year values due to a young intern accidentally messing up their data entry process!

We will definitely need to fix this data issue before we start tackling the following business requirements from HR Analytica.

## 3.1. employee

Let’s start with the employees.employee table in the middle of the ERD.

After inspecting a few rows from this table - we can see that there is a unique row of personal information for each employee in our database. There is primary key on the id column which we can later join onto our other tables via the employee_id field.

There is the issue with the dates where our young unlucky intern accidentally input the year which is 18 years behind what it should be - so we will need to keep an eye on this one as we think about our data solutions for the case study!

```sql
 SELECT *
 FROM `younglapalma.employees.employee`
 ORDER BY id ASC
 LIMIT 5;
```
| id    | birth_date | first_name | last_name | gender | hire_date  |
|-------|------------|------------|-----------|--------|------------|
| 10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
| 10002 | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
| 10003 | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
| 10004 | 1954-05-01 | Chirstian  | Koblick   | M      | 1986-12-01 |
| 10005 | 1955-01-21 | Kyoichi    | Maliniak  | M      | 1989-09-12 |

## 3.2. title

Our second table is the employees.title table which contains the employee_id which we can join back to our employees.employee table.

After inspecting the data - we notice that there is in fact a many-to-one relationship between the employees.title and employees.employee tables.

In simple terms - in the employees.employee table there is only ever 1 row for each unique id value. However in the employees.title table - each employee_id value can have multiple title values over their career with the company.

If we reverse the direction of the table relationship - we can also say that employees.employee has a one-to-many relationship with the employees.title table. This is important to keep in mind as it is really easy to get confused when throwing around tables and terms during everyday conversations in the SQL heavy workplace!

Let’s inspect the titles for the 5th row of data from our employee table for and employee named Kyoichi Maliniak with employee_id = 10005:

```sql
 SELECT *
 FROM `younglapalma.employees.title`
 WHERE employee_id = 10005
 ORDER BY from_date;
```
| employee_id | title         | from_date  | to_date    |
|-------------|---------------|------------|------------|
| 10005       | Staff         | 1989-09-12 | 1996-09-12 |
| 10005       | Senior Staff  | 1996-09-12 | 9999-01-01 |


Note how there is both a from_date column and a to_date column in this dataset. We commonly refer to these sorts of tables as slow changing dimension tables or SCDs in data engineering terms.

This essentially means that certain records in an SCD table are “expired” once the relationship ceases to exist and the dates where these new relationships are effective and expire are recorded in this table as from_date and to_date respectively.

For our example employee_id = 10005 Kyoichi Maliniak’s title was originally “Staff” from 1989-09-12 to 1996-09-12 when he was then promoted to “Senior Staff” which is his current position until the “arbitrary” end date of 9999-01-01 in our dataset.

Oh wait….I forgot - the intern also messed up the date fields in this table too - they actually messed up everywhere but hey - making mistakes is one of the best ways to learn, as long as you learn from your mistakes!

We will need to figure out what to do with these date fields as it’s usually recommended to keep that “arbitrary” end date 9999-01-01 as is - but we might need to add on our 18 years to every other single value.

## 3.3. salary
The third table is the all-important employees.salary table - it also has a similar relationship with the unique employees.employee table in that there are many-to-one or one-to-many records for each employee and their salary amounts over time.

The same from_date and to_date columns exist in this table, along with it’s arbitrary end date of 9999-01-01 which we will need to deal with later.

Let’s also continue to check employee_id 10005’s records for this table ordered by the from_date ascending from earliest to latest to checkout his salary growth over the years with the company:

```sql
 SELECT *
 FROM `younglapalma.employees.salary`
 WHERE employee_id = 10005
 ORDER BY from_date;
```

| employee_id | amount  | from_date  | to_date    |
|-------------|---------|------------|------------|
| 10005       | 78,228  | 1989-09-12 | 1990-09-12 |
| 10005       | 82,621  | 1990-09-12 | 1991-09-12 |
| 10005       | 83,735  | 1991-09-12 | 1992-09-11 |
| 10005       | 85,572  | 1992-09-11 | 1993-09-11 |
| 10005       | 85,076  | 1993-09-11 | 1994-09-11 |
| 10005       | 86,050  | 1994-09-11 | 1995-09-11 |
| 10005       | 88,448  | 1995-09-11 | 1996-09-10 |
| 10005       | 88,063  | 1996-09-10 | 1997-09-10 |
| 10005       | 89,724  | 1997-09-10 | 1998-09-10 |
| 10005       | 90,392  | 1998-09-10 | 1999-09-10 |
| 10005       | 90,531  | 1999-09-10 | 2000-09-09 |
| 10005       | 91,453  | 2000-09-09 | 2001-09-09 |
| 10005       | 94,692  | 2001-09-09 | 9999-01-01 |

## 3.4. department_employee
We now take a look at the employees.department_employee table which captures information for which department each employee belongs to throughout their career with our company.

In the same vain as the previous tables - we have the same slow changing dimension (SCD) style data design with a many-to-one relationship with the base employees.employee table (and vice-versa!)

This time instead of the salary or the title records - we now have the department_id value for where each employee was situated during various periods of their career.

Let’s continue with Kyoichi’s records to see if he’s moved around the company:

```sql
 SELECT *
 FROM `younglapalma.employees.department_employee`
 WHERE employee_id = 10005
 ORDER BY from_date;
```
| employee_id | department_id | from_date  | to_date    |
|-------------|---------------|------------|------------|
| 10005       | d003          | 1989-09-12 | 9999-01-01 |

It seems that Kyoichi has been loyal to department_id = 'd003' throughout his career so far. Let’s quickly look for another employee who has been in the most departments!

```sql
 SELECT
  employee_id,
  COUNT(DISTINCT department_id) AS unique_departments
 FROM `younglapalma.employees.department_employee`
 GROUP BY employee_id
 ORDER BY unique_departments DESC
 LIMIT 5;
```

| employee_id | unique_departments |
|-------------|--------------------|
| 10029       | 2                  |
| 10040       | 2                  |
| 10010       | 2                  |
| 10018       | 2                  |
| 10050       | 2                  |

It looks like the most number of departments for any employee is a max of 2 - so let’s just quickly take a look at employee_id = 10029:

```sql
 SELECT *
 FROM `younglapalma.employees.department_employee`
 WHERE employee_id = 10029
 ORDER BY from_date;
```

| employee_id | department_id | from_date  | to_date    |
|-------------|---------------|------------|------------|
| 10029       | d004          | 1991-09-18 | 1999-07-08 |
| 10029       | d006          | 1999-07-08 | 9999-01-01 |

Great we can see that they’ve changed departments from d004 to d006 in 1999-07-08 (well, we’ll add 18 years to this date later!)

This department_id value is all good and well though - but wouldn’t it be more useful if we were to actually use the department name…

Before we cover the actual department name - let’s also take a look at the department manager too, this time still with the random looking department_id values!

## 3.5. department_manager

In the same way that the employees.department_employee table shows the relationship between employees and their respective departments throughout time - the employees.department_manager table shows the employee_id of the manager of each department throughout time.

To inspect this dataset - how about we take a look at that department_id = 'd004' record:

```sql
 SELECT *
 FROM `younglapalma.employees.department_manager`
 WHERE department_id = 'd004'
 ORDER BY from_date;
```

| employee_id | department_id | from_date  | to_date    |
|-------------|---------------|------------|------------|
| 110303      | d004          | 1985-01-01 | 1988-09-09 |
| 110344      | d004          | 1988-09-09 | 1992-08-02 |
| 110386      | d004          | 1992-08-02 | 1996-08-30 |
| 110420      | d004          | 1996-08-30 | 9999-01-01 |

Nice - now we know the current and previous managers of department_id d004 - well at least we know their employee_id, we’ll need to join back onto the employees.employee table to grab out more of their personal details.

Ok finally let’s move onto the final table in our employees data schema, employees.department !

## 3.6. department

The employees.department table is just like the employees.employee table where there is a 1:1 unique relationship between the id or department_id and the dept_name.

This also happens to be the only table where our unfortunate intern did not make a data entry mistake - but that was only because there were no date fields in this table!

Let’s query the entire department table to inspect how many departments we are dealing with here sorting by that id string field just so we can see that final number increasing:

```sql
SELECT * 
FROM `younglapalma.employees.department` 
ORDER BY id ASC;
```
| id    | dept_name          |
|-------|--------------------|
| d001  | Marketing          |
| d002  | Finance            |
| d003  | Human Resources    |
| d004  | Production         |
| d005  | Development        |
| d006  | Quality Management |
| d007  | Sales              |
| d008  | Research           |
| d009  | Customer Service   |

# 4. Conclusion

This completes the quick data exploration of all the tables we will be using for this case study - there’s not too many tables but the amount of work we are going to have to do is no joke!

In the next tutorial we will start tackling our problem - first we will remedy our young intern’s data entry mistake and explore how we can create some reusable data assets to store this case study’s results to scale out our solution.