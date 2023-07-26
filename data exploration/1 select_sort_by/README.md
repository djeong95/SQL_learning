# Select and Sort SQL Queries

## Order By Exercises

1. What is the `name` of the category with the highest `category_id` in the `dvd_rentals.category` table?
```sql
SELECT
  name,
  category_id
FROM dvd_rentals.category
ORDER BY category_id desc
```
Answer: Travel

2. For the films with the longest `length`, what is the `title` of the “R” rated film with the lowest `replacement_cost` in `dvd_rentals.film` table?

```sql
SELECT
  title,
  replacement_cost,
  length,
  rating
FROM dvd_rentals.film
ORDER BY length DESC, replacement_cost
```
Answer: HOME PITY	

3. Who was the `manager` of the store with the highest `total_sales` in the `dvd_rentals.sales_by_store` table?

```sql
SELECT manager, total_sales FROM `dvd_rentals.sales_by_store`
ORDER BY total_sales DESC;
```

Answer: Jon Stephens

4. What is the `postal_code` of the city with the 5th highest `city_id` in the `dvd_rentals.address` table?

```sql
SELECT city_id, postal_code FROM `dvd_rentals.address`
ORDER BY city_id DESC
LIMIT 5;
```

Answer: 31390

## Splitting Statements With `;`
This `;` is known as a “statement terminator” and is used to tell the SQL engine that this is the end of a SQL statement.

This is important when we start writing multiple SQL statements as part of a complete SQL script.

It is good practice to get into the habit of having this `;` at the end of every statement you wish to run as some SQL flavours will not actually execute commands until there is a terminator present!

## Order By Additional Notes

Text fields will be sorted in alphabetical order, but be careful when some of these text fields start with numbers or non-alphabetical characters.

```sql 
WITH test_data (sample_values) AS (
VALUES
(null),
('0123'),
('_123'),
(' 123'),
('(abc'),
('  abc'),
('bca')
)
SELECT * FROM test_data
ORDER BY 1;
```

When we put the `NULLS FIRST` expression at the end of the `ORDER BY` clause we will see a different output:

```sql
WITH test_data (sample_values) AS (
VALUES
(null),
('0123'),
('_123'),
(' 123'),
('(abc'),
('  abc'),
('bca')
)
SELECT * FROM test_data
ORDER BY 1 NULLS FIRST;
```

This also stay true when we use an `ORDER BY DESC`:
```sql
WITH test_data (sample_values) AS (
VALUES
(null),
('0123'),
('_123'),
(' 123'),
('(abc'),
('  abc'),
('bca')
)
SELECT * FROM test_data
ORDER BY 1 DESC NULLS FIRST;
```

