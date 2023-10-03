
## 6.2.1 Advanced Window Functions Exercises
For the following questions - they are not compulsory but you can use this new dataset as an opportunity to further develop your SQL skills.

I suggest you to write all your code in a separate file so you can refer to them later - it might be useful for a future portfolio project!

The answers to the following questions will be available in an optional quiz at the end of this marketing case study section in case you would like to check your answers!

1. What is the earliest and latest market_date values?

```sql
SELECT 
  max(market_date),
  min(market_date)
FROM `younglapalma.trading.daily_btc`;
```

2. What was the historic all-time high and low values for the close_price and their dates?

```sql
(
SELECT
market_date,
close_price
FROM `younglapalma.trading.daily_btc`
ORDER BY close_price DESC NULLS LAST
LIMIT 1
)
UNION ALL
(
SELECT
market_date,
close_price
FROM `younglapalma.trading.daily_btc`
ORDER BY close_price ASC NULLS LAST
LIMIT 1
);
```


3. Which date had the most volume traded and what was the close_price for that day?

```sql
WITH ordered_cte AS (
SELECT
market_date,
volume,
close_price,
ROW_NUMBER() OVER (ORDER BY volume DESC NULLS LAST) AS _row_number
FROM `younglapalma.trading.daily_btc`
)
SELECT * FROM ordered_cte
WHERE _row_number = 1;
```

4. How many days had a low_price price which was 10% less than the open_price?
5. What percentage of days have a higher close_price than open_price?
6. What was the largest difference between high_price and low_price and which date did it occur?
7. If you invested $10,000 on the 1st January 2016 - how much is your investment worth in 1st of February 2021? Use the close_price for this calculation

Hint: remember to decide whether you want NULLS FIRST or NULLS LAST in any ORDER BY queries and make use of CASE WHEN statements where possible!
