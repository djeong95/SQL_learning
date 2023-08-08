# Review ERD

<img width="543" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/9bdf39ff-956e-4a5d-b0e2-98c82dca582e">

## Summary

1. Identify top 2 categories for each customer based off their past rental history
2. For each customer recommend up to 3 popular unwatched films for each category
3. Generate 1st category insights that includes:
    - How many total films have they watched in their top category?
    - How many more films has the customer watched compared to the average DVD Rental Co customer?
    - How does the customer rank in terms of the top X% compared to all other customers in this film category?
4. Generate 2nd insights that includes:
    - How many total films has the customer watched in this category?
    - What proportion of each customer’s total films watched does this count make?
5. Identify each customer’s favorite actor and film count, then recommend up to 3 more unwatched films starring the same actor

# Approaching the Data Problem
To frame our solution and strategy for the case study - we need to mentally walk through exactly what data points we need for our final outputs, then systematically work backwards.

We will follow this reverse-engineering strategy for the entire case study - it is especially potent when we need to think about all of our data processing steps required to generate our required output in practice!

To reduce any ambiguity in this case study - let’s first inspect the exact SQL outputs we need to match our business requirements.

## Defining The Target State

One very important concept when working with data is “reverse engineering” an end results or working backwards from the result or target state.

But what does this look like in practice? Where do we even start with our current business problem?

A counter-intuitive approach is to look at the final end user/business output that we are required to generate.

In our marketing analytics problem - the actual end result will be a completed email template for each customer.

Since we are not dealing with the downstream business aspects of the marketing campaign such as campaign reporting or attribution of sales - we are only interested in helping the marketing team populate the data elements of their emails - and hence it will be our final endpoint to start reverse engineering our solution.

## Final SQL Output

**Category Data Points**

1. Top ranking category name: `cat_1`
2. Top ranking category customer insight: `insight_cat_1`
3. Top ranking category film recommendations: `cat_1_reco_1`, `cat_1_reco_2`, `cat_1_reco_3`
4. 2nd ranking category name: `cat_2`
5. 2nd ranking category customer insight: `insight_cat_2`
6. 2nd ranking category film recommendations: `cat_2_reco_1`, `cat_2_reco_2`, `cat_2_reco_3`

**Actor Data Points**

7. Top actor name: `actor`
8. Top actor insight: `insight_actor`
9. Actor film recommendations: `actor_reco_1`, `actor_reco_2`, `actor_reco_3`

The final output that we need to generate will contain the above columns required by the marketing team to execute their marketing campaign.

The table should contain a single row for each `customer_id` and contains the names of the top 2 categories, the top actor’s first and full name as well all of the recommendations and insights previously generated as stated above - you will need to scroll right to see the complete table output as it is quite wide!

Certainly! Here's the provided data in a GitHub Markdown table format:

| customer_id | cat_1   | cat_1_reco_1 | cat_1_reco_2 | cat_1_reco_3 | cat_2   | cat_2_reco_1 | cat_2_reco_2 | cat_2_reco_3 | actor        | actor_reco_1  | actor_reco_2   | actor_reco_3    | insight_cat_1                                                                                    | insight_cat_2                                                                         | insight_actor                                                                                    |
|-------------|---------|--------------|--------------|--------------|---------|--------------|--------------|--------------|--------------|---------------|----------------|------------------|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| 1           | Classics | Timberland Sky | Gilmore Boiled | Voyage Legally | Comedy  | Zorro Ark   | Cat Coneheads | Operation Operation | Val Bolger  | Primary Glass | Alaska Phantom | Metropolis Coma | You’ve watched 6 Classics films, that’s 4 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!   | You’ve watched 5 Comedy films making up 16% of your entire viewing history!     | You’ve watched 6 films featuring Val Bolger! Here are some other films Val stars in that might interest you! |
| 2           | Sports  | Gleaming Jawbreaker | Talented Homicide | Roses Treasure | Classics | Frost Head  | Gilmore Boiled | Voyage Legally     | Gina Degeneres | Goodfellas Salute | Wife Turn     | Dogma Family     | You’ve watched 5 Sports films, that’s 3 more than the DVD Rental Co average and puts you in the top 7% of Sports gurus!    | You’ve watched 4 Classics films making up 15% of your entire viewing history!    | You’ve watched 5 films featuring Gina Degeneres! Here are some other films Gina stars in that might interest you! |
| 3           | Action  | Rugrats Shakespeare | Suspects Quills | Handicap Boondock | Animation | Juggler Hardly | Dogma Family   | Storm Happiness    | Jayne Nolte   | English Bulworth | Sweethearts Suspects | Dancing Fever | You’ve watched 4 Action films, that’s 2 more than the DVD Rental Co average and puts you in the top 14% of Action gurus! | You’ve watched 3 Animation films making up 12% of your entire viewing history! | You’ve watched 4 films featuring Jayne Nolte! Here are some other films Jayne stars in that might interest you! |
| 4           | Horror  | Pulp Beverly       | Family Sweet   | Swarm Gold    | Travel  | Bucket Brotherhood | Muscle Bright  | Horror Reign       | Kirk Jovovich | Network Peak     | Rush Goodfellas    | Forrester Comancheros | You’ve watched 3 Horror films, that’s 2 more than the DVD Rental Co average and puts you in the top 22% of Horror gurus!   | You’ve watched 2 Travel films making up 9% of your entire viewing history!       | You’ve watched 4 films featuring Kirk Jovovich! Here are some other films Kirk stars in that might interest you! |
| 5           | Classics | Timberland Sky | Frost Head     | Gilmore Boiled | Animation | Juggler Hardly | Dogma Family   | Storm Happiness    | John Suvari  | Gleaming Jawbreaker | Goldmine Tycoon   | Color Philadelphia | You’ve watched 7 Classics films, that’s 5 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!   | You’ve watched 6 Animation films making up 16% of your entire viewing history!    | You’ve watched 4 films featuring John Suvari! Here are some other films John stars in that might interest you! |


The above Markdown table presents the provided data on customer preferences and movie recommendations. If you have more data or any other questions, feel free to ask!

# Customer Insights

If we isolate each component of the final SQL output table we need to generate there are a few clear sections which we can use to focus our problem solving on.

The first things that pop out are the long text fields for the insights for both categories and top actor sections of the email.

**Top Category**

| customer_id | category_name | insight                                                                                                          |
|-------------|---------------|------------------------------------------------------------------------------------------------------------------|
| 1           | Classics      | You’ve watched 6 Classics films, that’s 4 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!       |
| 2           | Sports        | You’ve watched 5 Sports films, that’s 3 more than the DVD Rental Co average and puts you in the top 7% of Sports gurus!         |
| 3           | Action        | You’ve watched 4 Action films, that’s 2 more than the DVD Rental Co average and puts you in the top 14% of Action gurus!        |
| 4           | Horror        | You’ve watched 3 Horror films, that’s 2 more than the DVD Rental Co average and puts you in the top 22% of Horror gurus!        |
| 5           | Classics      | You’ve watched 7 Classics films, that’s 5 more than the DVD Rental Co average and puts you in the top 1% of Classics gurus!       |

**Second Category**

| customer_id | category_name | insight                                                                                                          |
|-------------|---------------|------------------------------------------------------------------------------------------------------------------|
| 1           | Comedy        | You’ve watched 5 Comedy films making up 16% of your entire viewing history!                                       |
| 2           | Classics      | You’ve watched 4 Classics films making up 15% of your entire viewing history!                                     |
| 3           | Animation     | You’ve watched 3 Animation films making up 12% of your entire viewing history!                                    |
| 4           | Travel        | You’ve watched 2 Travel films making up 9% of your entire viewing history!                                        |
| 5           | Animation     | You’ve watched 6 Animation films making up 16% of your entire viewing history!                                    |

**Actor Insight**

| customer_id | actor_name      | insight                                                                                                       |
|-------------|-----------------|---------------------------------------------------------------------------------------------------------------|
| 1           | Val Bolger      | You’ve watched 6 films featuring Val Bolger! Here are some other films Val stars in that might interest you! |
| 2           | Gina Degeneres  | You’ve watched 5 films featuring Gina Degeneres! Here are some other films Gina stars in that might interest you! |
| 3           | Jayne Nolte     | You’ve watched 4 films featuring Jayne Nolte! Here are some other films Jayne stars in that might interest you! |
| 4           | Kirk Jovovich   | You’ve watched 4 films featuring Kirk Jovovich! Here are some other films Kirk stars in that might interest you! |
| 5           | John Suvari     | You’ve watched 4 films featuring John Suvari! Here are some other films John stars in that might interest you! |


The text outputs match exactly the template we’ve received for the email - but if we investigate further, there are clear metrics that we need to calculate before we combine them to form the text fields.

If we pull on this thread a little further - we should be required to generate a prior table output which contains all of these in each row so we can concatenate them efficiently.

Let’s first investigate the category components of the insights.

# Top Categories Information

To generate each customer insight we need the following inputs:

- `category_name`: The name of the top or second ranking category
- `rental_count`: How many total films have they watched in this category?
- `average_comparison`: How many more films has the customer watched compared to the average DVD Rental Co customer?
- `percentile`: How does the customer rank in terms of the top X% compared to all other customers in this film category?
- `category_percentage`: What proportion of each customer’s total films watched does this count make?


The top category insight will use these inputs in the following text output:

> You’ve watched {`rental_count`} {`category_name`} films, that’s {`average_comparison`} more than the Dvd Rental Co average and puts you in the top {`percentile`}% of {`category_name`} gurus!


The second category insight text output uses the fields in a similar way:

> You’ve watched {`rental_count`} {`category_name`} films making up {`category_percentage`}% of your entire viewing history!

An example output from this table looks like the following - where each record is a customer and category combination for the top 2 categories.

Although we do not need all of these metrics for both the top and the second category insight - it seems like it would be an efficient process to generate all the required data in one step and deal with the filtering in a later step.

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                  | 2          | 16                  |
| 2           | 1                | Sports        | 5            | 3                  | 7          | 19                  |
| 2           | 2                | Classics      | 4            | 2                  | 11         | 15                  |
| 3           | 1                | Action        | 4            | 2                  | 14         | 15                  |
| 3           | 2                | Sci-Fi        | 3            | 1                  | 15         | 12                  |
| 4           | 1                | Horror        | 3            | 2                  | 22         | 14                  |
| 4           | 2                | Travel        | 2            | 1                  | 57         | 9                   |

## Top Actor Information

We can now move our focus onto the top actors for each customer. We also need the `rental_count` as well as the `first_name` and full `actor_name` for our corresponding actor insight.

The text output will use the fields as follows:

> You’ve watched <`rental_count`> films featuring <`actor_name`>! Here are some other films <`first_name`> stars in that might interest you!

| customer_id | actor_id | first_name | actor_name      | rental_count |
|-------------|----------|------------|-----------------|--------------|
| 1           | 37       | Val        | Val Bolger      | 6            |
| 2           | 107      | Gina       | Gina Degeneres  | 5            |
| 3           | 150      | Jayne      | Jayne Nolte     | 4            |
| 4           | 43       | Kirk       | Kirk Jovovich   | 4            |
| 5           | 192      | John       | John Suvari     | 4            |


# Film Recommendations

The next component of the final SQL output table are the individual film recommendations for each category and top actor.

Since our final output seems to have a single row per customer - it makes sense for one of our interim outputs to also be in the same wide format.

## Category Recommendations

The table below is what we could generate in order to populate the final SQL table output where the `customer_id` is followed by 6 columns with the first 3 related to the top category and the last 3 for the second category.

We also need to remember that the customer must not have seen any of these films before and that the film recommendations are ranked by total popularity across all customers.

| customer_id | cat_1_reco_1       | cat_1_reco_2   | cat_1_reco_3   | cat_2_reco_1 | cat_2_reco_2 | cat_2_reco_3   |
|-------------|--------------------|----------------|----------------|--------------|--------------|----------------|
| 1           | Timberland Sky     | Gilmore Boiled | Voyage Legally | Zorro Ark    | Cat Coneheads| Operation Operation |
| 2           | Gleaming Jawbreaker| Talented Homicide | Roses Treasure | Frost Head   | Gilmore Boiled | Voyage Legally   |
| 3           | Rugrats Shakespeare| Suspects Quills | Handicap Boondock | Juggler Hardly | Dogma Family | Storm Happiness |
| 4           | Pulp Beverly       | Family Sweet | Swarm Gold   | Bucket Brotherhood | Muscle Bright | Horror Reign |
| 5           | Timberland Sky     | Frost Head   | Gilmore Boiled | Juggler Hardly | Dogma Family | Storm Happiness |


## Actor Recommendations
Just like the category recommendations we will be required to generate the top 3 films by their favourite actor.

An extra reminder that we need to also remove films which customers have already seen before - or if they’ve already been recommended in the category recommendations.

| customer_id | cat_1_reco_1       | cat_1_reco_2   | cat_1_reco_3   | cat_2_reco_1 | cat_2_reco_2 | cat_2_reco_3   |
|-------------|--------------------|----------------|----------------|--------------|--------------|----------------|
| 1           | Timberland Sky     | Gilmore Boiled | Voyage Legally | Zorro Ark    | Cat Coneheads| Operation Operation |
| 2           | Gleaming Jawbreaker| Talented Homicide | Roses Treasure | Frost Head   | Gilmore Boiled | Voyage Legally   |
| 3           | Rugrats Shakespeare| Suspects Quills | Handicap Boondock | Juggler Hardly | Dogma Family | Storm Happiness |
| 4           | Pulp Beverly       | Family Sweet | Swarm Gold   | Bucket Brotherhood | Muscle Bright | Horror Reign |
| 5           | Timberland Sky     | Frost Head   | Gilmore Boiled | Juggler Hardly | Dogma Family | Storm Happiness |

# Identifying Key Columns

So the next question is naturally - how are we going to generate all of these SQL table output from our raw data?

One step is to identify just exactly where our data comes from so we can trace back our data to help us combine multiple tables for our analysis.

Let’s quickly review the ERD but also highlight some of the columns which we might be interested in from the final SQL output table.

<img width="536" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/10510c28-4805-4e44-b52b-a513c16445f3">

# Next Steps?
We clearly have our work cut out for us if we need to combine the key columns that we’ve identified then perform some intense data manipulation to transform the data into something which we can use to generate these outputs for the marketing team!

In the next tutorial we will take a slight detour from our current case study and focus purely on table joins - something which we will seriously need a lot of in this marketing analytics case study!

I will introduce some simple datasets which we will perform every type of SQL join possible. This should help us build up our foundation before we start diving into our case study further!

