# Summary Statistics

 Looking at the raw data isn’t always the best way to intuitively solve most of the problems!

 We will focus on further analyzing and exploring our raw data through the use of powerful summary statistics.

The core ideas will become additional layers to our previous SQL knowledge but will also have a laser sharp focus on how to interpret some of these outputs that come out of our statistical analysis.

To start off, we will cover the base level of what you need to know about statistics.

--------

When we use summary statistics in SQL, we can compute our statistics on individual columns, but more generally we often combine the functions used to generate certain statistics with the powerful core foundational concept of `GROUP BY` to calculate statistics within certain groups that we are interested in.

For the entirety of this tutorial - we will continue inspecting our `health.user_logs` dataset.

Once we’ve covered some of these initial statistical concepts, we’ll return to some of our past decisions in the previous duplicates tutorial to demonstrate their impact on our summary statistics.

But first things first - what even are summary statistics?

# Statistics 101

Statistics is full of is what I like to call “threshold concepts” whereby once you learn their meaning and understand the concept - you will find it very difficult to forget them!

This is exactly the case with some of the following terms I will introduce below - chances are you’ve probably used these same terms before in school, Excel or some other data/math related applications.

As we introduce each statistical concept - I will demonstrate how to calculate each summary statistic on the `measure_value` column in the `health.user_logs` table. I wonder if you can already see any fundamental issues with this…?

I’ll also provide some mathematical notation so you can start familiarising yourself with some equations. If you are aspiring to become a machine learning guru or data science experimentation boss - you kinda need to know how to read equations. Preferably very well.

Let’s get our statistics on!

# Central Location Statistics

Location statistics are something I’m sure you’ve come across - mainly the mean, median and mode values.

They are all measures of the central location summary statistics and are often used in analytical reports everywhere!

The implementation of each metric is different so be sure to read through the code snippets and run them in your SQL environment!

## Arithmetic Mean or Average

The **Arithmetic Mean or Average** is something I’m sure you’ve seen in the past. It’s definition is simply the sum of all values divided by the total count of values for a set of numbers.

The mean is commonly used as a location summary statistic to show the central tendancy for a set of observations. Note that the mean can only be calculated for numbers and cannot be used on any other data type.

The following mathematical equation is commonly used to show the mean calculation.

$$\mu = \frac{\sum^N_{i=1} X_i}{N}$$

The `mu` greek letter $\mu$ on the left is the most commonly used mathematical symbol to represent the mean and you will see this very often in future!

For a set of observations containing a total of N numbers: $x_1,x_2,x_3,...,x_N$ - the mean equals the [ sum of all xi from i = 1 to i = N ] divided by N.

The little `i` subscript of the x value is what is known as a dummy variable and any letter can be used in this equation. Often `i` and `j` are used for most mathematical equations you’ll encounter, as well as in `for loops` in programming languages.

The SQL implementation is relatively simple but can change depending on the flavour of SQL you are using!

In PostgreSQL the mean is calculated using the `AVG` function like so:
```sql
SELECT
  AVG(measure_value)
FROM health.user_logs;
```
**avg: 1986.2288605267076**

Wait a moment…what were our measures called again and how many record counts were there?

```sql
SELECT
  measure,
  COUNT(*) AS counts
FROM health.user_logs
GROUP BY measure
ORDER BY counts DESC;
```
| measure        | counts |
|---------------|-------:|
| blood_glucose  |  38692 |
| weight         |   2782 |
| blood_pressure |   2417 |

Do you notice something fishy going on? What happens if we also take a look at the `AVG` value across each `measure` too?

| measure        | average   | counts |
|---------------|----------|-------|
| blood_glucose  |   177.35  |  38692 |
| **weight**         | **28786.85**  |   2782 |
| blood_pressure |    95.40  |   2417 |

LOOKS FISHY TO ME!

Let’s park this analysis for now but we will definitely come back to this!

# Median & Mode

These are two more central tendancy statistics that are different to the mean.

Whilst the mean is computed by summing all the values and then taking the average number, the median shows the central location based off the values of the middle values, whilst the mode focuses on the most frequent values.

The mode is simply calculated as the value that appears the most number of times - but a given set of observations can have more than one mode!

## The Median Concept

The concept of the median goes like this:

Firstly imagine that all of your N observations of numbers were lined up from smallest to largest in a single line.

To calculate the median we look at the number directly in the middle of this ordered line up.

This is straightforward for N numbers where the N is odd, for example the middle of 7 total numbers is the 4th position.

But what happens when N is an even number…won’t there be two numbers in the middle? For example in a row of 10 numbers, both positions 5 and 6 are both considered the middle values.

When this occurs - we take the average of those two numbers as the median value.

In probability theory - the median is also known as the 50th percentile value. It is the middle value which separates the bottom and top halves of all the values hence why it makes sense that the word median is derived from the latin word medius which means middle!

This 50th percentile will become important once we go into the implementation of the median calculation in our SQL environment!

## Basic Example of Mean, Median & Mode

Let’s use the following example to demonstrate all 3 central tendancy summary statistics of the mean, median and mode.

Consider the following data set with 10 numbers:

$${82,51,144,84,120,148,148,108,160,86}$$

The mean is the sum of all the numbers divided by 10:
$$\mu = 82+51+144+84+120+148+148+108+160+8610=113.1$$

To calculate the median value, we must first sort the dataset from smallest to largest.

$${51,82,84,86,108,120,144,148,148,160}$$

Because there are 10 total numbers in this dataset - we need to calculate the average of the 5th and 6th elements.

$$Median=(108+120)/2=114$$

To calculate the model, we need to tally up the occurences of each unique value in our dataset before taking the most frequent value. Does this remind you of something that we did in the record counts tutorial before…

The mode value is 148 because it appears twice in this dataset.

$$Mode=148$$

Note that you can actually have 1 or more mode values in a dataset! For example, say that there were 2 occurences of 160 in our above dataset - then the mode(s) of our dataset would be both 148 AND 160.

There’s also one more hitch to these 2 summary statistics…the implementation of both median and mode values in PostgreSQL are not as simple as just using the `AVG` function for the mean!

## Ordered Set Aggregate Functions

When you think of the underlying steps of calculating the median or the mode - there are a few steps involved in the “algorithm” required to calculate the values (yes - you can think of these algorithms!)

**Median Algorithm**
1. Sort all N values from smallest to largest
2. Inspect the central values of the sorted set:
    - if N is odd:
        - the median is the value in the $\frac{N+1}{2}$
        th position
    - else if N is even:
        - the median is the average of values in the (N/2) th and 1+(N/2) th positions

**Mode Algorithm**
1. Calculate the tally of values similar to a `GROUP BY` and `COUNT`
2. The mode is the values with the highest number of occurences

To implement these “algorithms” we need to use what are called Ordered Set Aggregate Functions in PostgreSQL.

Some flavours of SQL actually have implemented median and mode values as regular functions - but we will demonstrate how to implement them in our current PostgreSQL flavour for completeness!

Let’s use that same example data from above in a CTE to demonstrate the PostgreSQL implementations of the median and mode functions.

Do you remember how I mentioned that the median is also known as the 50th percentile value? Well - you will see why below!

```sql
with sample_data AS (
  SELECT *
  FROM
  UNNEST([82, 51, 144, 84, 120, 148, 148, 108, 160, 86]) as example_value
),

stats AS (
  SELECT
    APPROX_QUANTILES(example_value, 100) AS quantiles,
    APPROX_TOP_COUNT(example_value, 1) AS mode_struct
  FROM  sample_data
)

SELECT
  stats.quantiles[OFFSET(50)] AS median_value,
  stats.mode_struct[OFFSET(0)].value AS mode_value,
  (SELECT AVG(sample_data.example_value) FROM sample_data) AS mean_value
FROM stats
;
```
| median_value | mode_value | mean_value            |
|----------------------|------------|-----------|
|         114 |        148 | 113.1000000000000000  |

For now - there is not much to add for the SQL implementation of these median and mode values, apart from the fact that they are a little bit annoying…

Let’s also use this same syntax to calculate our median and mode values for our `health.user_logs` dataset for the weight measurements only - what happens when we compare it with the average value?

```sql
WITH stats AS (
  SELECT
    APPROX_QUANTILES(measure_value, 100) AS quantiles,
    APPROX_TOP_COUNT(measure_value, 1) AS mode_struct
  FROM `health.user_logs`
  WHERE measure = 'weight'
)

SELECT
  stats.quantiles[OFFSET(50)] AS median_value,
  stats.mode_struct[OFFSET(0)].value AS mode_value,
  (SELECT AVG(measure_value) FROM `health.user_logs` WHERE measure = 'weight') AS mean_value
FROM stats;
```

| median_value  | mode_value  | mean_value       |
|--------------:|------------:|-----------------:|
| 75.976721975  | 68.49244787 | 28786.846657296  |

What do you notice about these values? Anything fishy???

Ok - now that we’ve covered these 3 main central location summary statistics, let’s move onto a few others!

# Spread of the Data

The previous summary statistics all show different measures regarding the central tendancy of a set of observations.

Although these are interesting, we also like to see summary statistics about the spread of the data.

Just like we can spread peanut butter from edge to edge of a piece of toast - we usually think of spread in terms of how is our data distributed?

This will make more sense once we cover a few more statistics concepts!

## Min, Max & Range

The minimum and maximum values of observations in a dataset help us identify the boundaries of where our data exists.

We use the range calculation to show the total possible limits of our data by simply calculating the difference between the max and min values from our data.

Luckily the implementations in SQL are straightforward unlike the mode and median calculations! Also the `MAX` and `MIN` functions are portable across different flavours of SQL.

There is no built-in implementation of range, but we can simply subtract the `MIN` from the `MAX` value as shown below with the weight values:

```sql
SELECT
  MIN(measure_value) AS minimum_value,
  MAX(measure_value) AS maximum_value,
  MAX(measure_value) - MIN(measure_value) AS range_value
FROM `health.user_logs`
WHERE measure = 'weight';
```

| minimum_value | maximum_value | range_value |
|--------------:|--------------:|------------:|
|             0.0 |      39642120.0 |    39642120.0 |

If you feel the need for speed - this following query below is actually slightly more efficient because it doesn’t duplicate the `MIN` and `MAX` calculations on the entire dataset but rather just takes the calculated values for the range metric following the CTE in the following query"

Remember what we mentioned about SQL query performance and the number of rows in our datasets? This is one simple example of this exact case where we reduced the number of rows down from the total size of the `health.user_logs` down to just 2 numbers!

```sql
WITH min_max_values AS (
  SELECT
    MIN(measure_value) AS minimum_value,
    MAX(measure_value) AS maximum_value
  FROM health.user_logs
  WHERE measure = 'weight'
)
SELECT
  minimum_value,
  maximum_value,
  maximum_value - minimum_value AS range_value
FROM min_max_values;
```

# Variance & Standard Deviation

The minimum and maximum values and the range are important to know for sure, but the real statistics stuff starts in when we start thinking about variance and standard deviation.

These two important values are used to describe the “spread” of the data about the mean value. Also, the variance is simply the square of the standard deviation. So once we know how to calculate one of these - we know both of these!

OK - the simplest way to learn more about these metrics is to develop an algorithm to calculate them!

## Variance Algorithm

Before we dive straight into the algorithms itself - it’s important to understand the mathematical formula for the variance value.

We often use $\sigma$ as the mathematical symbol for standard deviation and $\sigma^2$ for variance. Do you remember what the $\mu$ symbol stands for?

$$\sigma^2 = \frac{\sum^n_{i=1}(x_i-\mu)^2}{n-1}$$

To explain this formula in simple terms - the variance is the sum of the (difference between each X value and the mean) squared, divided by N - 1, 1 less than the total number of values.

Why do we need to square this value? See if you can figure it out on your own!

Let’s return to our original sample data example to explain our variance algorithm:

Consider the following data set with 10 numbers:

$${82,51,144,84,120,148,148,108,160,86}$$

First we calculate the mean:

$$\mu = 82+51+144+84+120+148+148+108+160+8610=113.1$$

Next, for each value in our dataset, we need to calculate the square of the difference between the value and the mean.

Let’s take the first value of 82 from our dataset:

$$(82−113.1)∗∗2=967.21$$

Now imagine we do this for all of our values to obtain the following numerator of our variance formula divided by 1 less than the total number of records we have in our dataset:

$$\sigma^2 = \frac{\sum^n_{i=1} (x_i - \mu)^2}{n-1}$$

$$\frac{(82−113.1)^2+(51−113.1)^2+...+(86−113.1)^2}{10−1}=1340.99$$

Finally let’s take a look at the single standard deviation $\sigma$ value by taking the square root of our previous calculated value.

$$\sigma = 36.62$$

This is what it looks like in SQL to add onto our previous example:

```sql
with sample_data AS (
  SELECT *
  FROM
  UNNEST([82, 51, 144, 84, 120, 148, 148, 108, 160, 86]) as example_value
),

stats AS (
  SELECT
    APPROX_QUANTILES(example_value, 100) AS quantiles,
    APPROX_TOP_COUNT(example_value, 1) AS mode_struct
  FROM  sample_data
)

SELECT
  ROUND((SELECT VARIANCE(example_value) FROM sample_data), 2) AS variance_value,
  ROUND((SELECT STDDEV(example_value) FROM sample_data), 2) AS standard_dev_value,
  ROUND((SELECT AVG(example_value) FROM sample_data), 2) AS mean_value,
  (SELECT DISTINCT PERCENTILE_CONT(example_value, 0.5) OVER () FROM sample_data) AS median_value,
  stats.mode_struct[OFFSET(0)].value AS mode_value
FROM stats;

```

| variance_value | standard_dev_value | mean_value | median_value | mode_value |
|--------------- |------------------- |----------- |------------- |----------- |
| 1340.99        | 36.62              | 113.10     | 114          | 148        |

So now we have the raw values - but there is still a question isn’t there…what do they **ACTUALLY** mean????

## Interpreting Spread

We’ve talked about the variance and standard deviation being a measure of spread - but what do we mean by this?

One of the best examples to explain this is to take a look at the humble bell curve, or normal distribution.

Now there is a huge caveat to this explanation in that - data is not ALWAYS normally distributed. In fact, lots of messy datasets are totally not normally distributed - there are certain statistical tests to detect this which I’ll share some links in the appendix to cover further.

### Spread For Normal Distribution

<img width="485" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/5413f203-4f33-44e9-8977-6d5cc6bc9879">

 I am almost certain you’ve encountered normal distributions in your life at some point in time, even if you didn’t know what they were!

For example, you could have encountered it in high school exams when you got your results. When you were told your exam score and a percentile value, this is exactly where the distribution comes into play!

However the percentile relates to how you performed compared to others who took the test. So you might be asking - how do we calculate this percentile? Well, it depends on a few factors, but in technical terms, we use the available data (the exam scores) to generate a normal distribution by using:

1. The average or the mean score of all exam takers
2. The spread or the standard deviation/variance of the score values

So let’s say for a specific test - I had scored 95, the average score was 75 and the standard deviation was 10 - on the back of my results, there would be a chart that looked like this with my percentile score inside the red box:

<img width="475" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/bc9bb54f-183b-428d-9ff1-cbce863d8adc">

To interpret my score: I had beaten ~97% of other test takers based off the mean and standard deviation statistics.

But let’s say that for a different example - the average score was slightly lower at 65 and the standard deviation was also higher at 15:

<img width="471" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/7598b4a2-9439-4864-9850-dcbf61132c91">

In this example test - I would have beaten ~99% of other test takers with my score of 95.

If you take a look at the X axis for both of the above charts - what do you notice about the “spread” of the values around that mean value?

This increase in standard deviation value sees a subsequent “spread” in the possible x-values - and this is exactly what we’d expect when we apply the same standard deviation metrics to our data - even though they might not be normal distributions!

Also a sidenote: did you know that we also refer to the mean as the “first moment” and the variance as the “second moment” in probability and statistics?

### The Empirical Rule or Confidence Intervals

One other way to interpret the standard deviation is the `Empirical Rule`

Essentially if our data is approximately normally distributed (it looks like a bell curve) - we can make rough generalisations about how much percentage of the total lies between different ranges related to our standard deviation values.

These boundaries are also known as confidence intervals or confidence bands - ranges where we are fairly sure that the data will lie within!

| Percentage of Values | Range of Values |
|---------------------|--------------- |
|                 68%  |     $\mu \pm \sigma$      |
|                 95%  |     $\mu \pm 2*\sigma$    |
|               99.7%  |     $\mu \pm 3*\sigma$    |

Quite commonly we will refer to these ranges as “one standard deviation about the mean” contains 68% of values etc.

<img width="375" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/aea155f2-21d4-4072-9d1d-5939d486ffe5">

# Calculating All The Summary Statistics

Ok - let’s try calculating all of the statistics we’ve covered so far on our real dataset to see what we can find out about our values.

To make things a bit cleaner - let’s deep dive into the weight measure values first before we get carried away!

Let’s also round everything to 2 decimal places to keep the printing nice and clean.

```sql
WITH stats AS (
  SELECT
    ROUND(MIN(measure_value), 2) AS minimum_value,
    ROUND(MAX(measure_value), 2) AS maximum_value,
    ROUND(AVG(measure_value), 2) AS mean_value,
    ROUND(STDDEV(measure_value), 2) AS standard_dev_value,
    ROUND(VARIANCE(measure_value), 2) AS variance_value,
    APPROX_QUANTILES(measure_value, 100)[OFFSET(50)] AS median_value
  FROM `health.user_logs`
  WHERE measure = 'weight'
),

mode AS (
  SELECT value AS mode_value
  FROM (
    SELECT 
      measure_value AS value, 
      COUNT(*) AS count
    FROM `health.user_logs`
    WHERE measure = 'weight'
    GROUP BY measure_value
    ORDER BY count DESC
    LIMIT 1
  )
)

SELECT
  'weight' AS measure,
  minimum_value,
  maximum_value,
  mean_value,
  ROUND(median_value, 2) AS median_value,
  ROUND((SELECT mode_value FROM mode), 2) AS mode_value,
  standard_dev_value,
  variance_value
FROM stats;
```

| measure | minimum_value | maximum_value | mean_value | median_value | mode_value | standard_deviation | variance_value        |
|---------|--------------:|--------------:|-----------:|-------------:|-----------:|-------------------:|----------------------:|
| weight  |          0.00 |   39642120.00 |   28786.85 |        75.98 |      68.49 |         1062759.55 | 1129457862383.41      |
