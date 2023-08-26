# Data Overview

It would probably be a smart idea to take a look at the data we have to play with before diving straight into solution mode for our business requirements.

Luckily for us - the Analytics team at DVD Rental Co have created an entity-relationship diagram (also known as an “ERD”) to highlight which tables in the dvd_rentals schema we should focus on.

To find out how to generate these awesome looking diagrams - checkout the appendix for more details!

# Data Exploration

<img width="586" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/89d16c29-2e74-437f-8120-ed49a53488ce">

I’ve labeled each of our spotlight tables from 1 through to 7 so we can keep track of where we are as we explore the available data.

Entity-Relationship Diagrams are super popular in the workplace and you will most definitely come across them many many times!

There are a few things to note about ERDs:

- all column names as well as the data type are shown for each table
- table indexes and foreign keys are usually highlighted to show the linkages between tables

Let’s walk through these `dvd_rentals` tables together and communicate our findings in some simple terms. This style of talking about datasets and their relationships is really useful when you need to actually tackle problems in the workplace - take note of some of the wording and phrases and try to re-use them where relevant when discussing datasets and ERDs in your daily practice.

## Table #1 - rental

This dataset consists of rental data points at a customer level. There is a unique sequential `rental_id` for each record in the table which corresponds to an individual `customer_id` purchasing or renting a specific item with an `inventory_id`. There is also information about the `rental_date` and `return_date` as well as which staff member served the customers.

The `last_update` field is an internal database timestamp for when the datapoints were inserted into the table.

In the ERD - we can see that there is a linkage between this rental table with the `inventory` table via the `inventory_id` field.

Show all rentals made by `customer_id = 130`:

```sql
SELECT *
FROM dvd_rentals.rental
WHERE customer_id = 130;
```
| rental_id | rental_date         | inventory_id | customer_id | return_date         | staff_id | last_update           |
|----------|---------------------|--------------|-------------|---------------------|---------|-----------------------|
|         1 | 2005-05-24 22:53:30 |          367 |         130 | 2005-05-26 22:04:30 |       1 | 2006-02-15 21:30:53   |
|       746 | 2005-05-29 09:25:10 |         4272 |         130 | 2005-06-02 04:20:10 |       2 | 2006-02-15 21:30:53   |
|      1630 | 2005-06-16 07:55:01 |         2413 |         130 | 2005-06-19 06:38:01 |       1 | 2006-02-15 21:30:53   |
|      1864 | 2005-06-17 01:39:47 |         1815 |         130 | 2005-06-24 19:39:47 |       2 | 2006-02-15 21:30:53   |
|      2163 | 2005-06-17 23:46:16 |         2600 |         130 | 2005-06-22 22:48:16 |       2 | 2006-02-15 21:30:53   |
|      2292 | 2005-06-18 07:37:48 |          173 |         130 | 2005-06-20 02:45:48 |       2 | 2006-02-15 21:30:53   |
|      2535 | 2005-06-19 01:39:04 |          901 |         130 | 2005-06-28 01:33:04 |       2 | 2006-02-15 21:30:53   |
|      2982 | 2005-06-20 08:38:29 |         2574 |         130 | 2005-06-28 13:21:29 |       1 | 2006-02-15 21:30:53   |
|      4339 | 2005-07-07 18:41:42 |         3215 |         130 | 2005-07-08 13:00:42 |       1 | 2006-02-15 21:30:53   |
|      4485 | 2005-07-08 01:07:54 |         2614 |         130 | 2005-07-16 03:19:54 |       2 | 2006-02-15 21:30:53   |
|      6353 | 2005-07-11 20:48:56 |          699 |         130 | 2005-07-21 00:11:56 |       1 | 2006-02-15 21:30:53   |
|      7181 | 2005-07-27 08:14:34 |         2788 |         130 | 2005-07-28 03:09:34 |       1 | 2006-02-15 21:30:53   |
|      7728 | 2005-07-28 04:56:33 |          492 |         130 | 2005-07-31 07:54:33 |       1 | 2006-02-15 21:30:53   |
|      9452 | 2005-07-30 22:19:16 |         3178 |         130 | 2005-08-04 19:26:16 |       1 | 2006-02-15 21:30:53   |
|      9637 | 2005-07-31 05:18:54 |         3013 |         130 | 2005-08-03 01:23:54 |       2 | 2006-02-15 21:30:53   |
|      9724 | 2005-07-31 08:33:08 |          518 |         130 | 2005-08-08 04:50:08 |       1 | 2006-02-15 21:30:53   |
|     10568 | 2005-08-01 13:17:28 |         3009 |         130 | 2005-08-08 17:04:28 |       1 | 2006-02-15 21:30:53   |
|     10645 | 2005-08-01 15:52:01 |         3864 |         130 | 2005-08-09 18:58:01 |       1 | 2006-02-15 21:30:53   |
|     11811 | 2005-08-17 11:59:18 |          195 |         130 | 2005-08-18 09:13:18 |       2 | 2006-02-15 21:30:53   |
|     12094 | 2005-08-17 22:31:04 |         1050 |         130 | 2005-08-23 22:45:04 |       1 | 2006-02-15 21:30:53   |
|     12777 | 2005-08-18 23:39:22 |         3467 |         130 | 2005-08-27 20:28:22 |       1 | 2006-02-15 21:30:53   |
|     14111 | 2005-08-21 00:59:01 |         1294 |         130 | 2005-08-22 20:43:01 |       2 | 2006-02-15 21:30:53   |
|     15574 | 2005-08-23 05:29:32 |         3253 |         130 | 2005-09-01 01:25:32 |       2 | 2006-02-15 21:30:53   |
|     15777 | 2005-08-23 13:29:08 |         4241 |         130 | 2005-08-27 18:50:08 |       2 | 2006-02-15 21:30:53   |


## Table #2 - inventory

<img width="463" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/a150c6eb-4b6e-4630-b1cf-fe850172717d">

This `inventory` dataset consists of the relationship between specific items available for rental at each store - note that there may be multiple inventory items for a specific film at a unique store.

In other words - say a specific film like “Harry Potter & The Chamber of Secrets” might have 4 copies at store #1 and an additional 4 copies in store #2 - each record in this dataset will have a separate `inventory_id` whilst the `film_id` will all be the same and the `store_id` changes according to the store number.

This `inventory` table is linked to the previous `rental` table via the `inventory_id` which is an integer datatype.

The next table we will investigate is how we will link these inventory items to a specific film via the `film` table and the `film_id` column.

Note that this query shows the `film_id = 1` records only!

```sql
SELECT *
FROM dvd_rentals.inventory
WHERE film_id = 1;
```

| inventory_id | film_id | store_id | last_update         |
|-------------|--------|---------|---------------------|
|            1 |       1 |        1 | 2006-02-15 05:09:17 |
|            2 |       1 |        1 | 2006-02-15 05:09:17 |
|            3 |       1 |        1 | 2006-02-15 05:09:17 |
|            4 |       1 |        1 | 2006-02-15 05:09:17 |
|            5 |       1 |        2 | 2006-02-15 05:09:17 |
|            6 |       1 |        2 | 2006-02-15 05:09:17 |
|            7 |       1 |        2 | 2006-02-15 05:09:17 |
|            8 |       1 |        2 | 2006-02-15 05:09:17 |

## Table #3 - film

<img width="571" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/2ad3fe6d-94c9-40ad-96b0-1b0509b93460">


The third table in our ERD is the `film` table which helps us identify the title of films rented by DVD Rental Co customers. The film title, description as well as special features and other fields are available for further analysis.

One specific column which might be of interest is the `rental_rate` column which represents the cost of each rental - something which could be useful if we wanted to look at the sales performance of DVD Rental Co.

Note that there are a few complex data types in the last few columns of this table - you can practically ignore them for the time being but these more involved data structures do occur in real life too so it’s just something you need to be aware of.

We will use the `film_id` column to help us join onto table #4 `film_actor` to help us identify which actors appeared in each film.


```sql
SELECT *
FROM dvd_rentals.film
LIMIT 5;
```

| film_id | title             | description | release_year | language_id | original_language_id | rental_duration | rental_rate | length | replacement_cost | rating | last_update | special_features | fulltext |
|---------|-------------------|-----------------------------------------------------------------------------------------------------------------|--------------|-------------|----------------------|-----------------|-------------|--------|------------------|--------|----------------------|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1       | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies | 2006         | 1           | null                 | 6               | 0.99        | 86     | 20.99            | PG     | 2006-02-15 05:03:42 | {"Deleted Scenes","Behind the Scenes"} | 'academi':1 'battl':15 'canadian':20 'dinosaur':2 'drama':5 'epic':4 'feminist':8 'mad':11 'must':14 'rocki':21 'scientist':12 'teacher':17 |
| 2       | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China       | 2006         | 1           | null                 | 3               | 4.99        | 48     | 12.99            | G      | 2006-02-15 05:03:42 | {Trailers,"Deleted Scenes"}          | 'ace':1 'administr':9 'ancient':19 'astound':4 'car':17 'china':20 'databas':8 'epistl':5 'explor':12 'find':15 'goldfing':2 'must':14      |
| 3       | ADAPTATION HOLES | A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory                | 2006         | 1           | null                 | 7               | 2.99        | 50     | 18.99            | NC-17  | 2006-02-15 05:03:42 | {Trailers,"Deleted Scenes"}          | 'adapt':1 'astound':4 'baloon':19 'car':11 'factori':20 'hole':2 'lumberjack':8,16 'must':13 'reflect':5 'sink':14                                     |
| 4       | AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                   | 2006         | 1           | null                 | 5               | 2.99        | 117    | 26.99            | G      | 2006-02-15 05:03:42 | {Commentaries,"Behind the Scenes"}  | 'affair':1 'chase':14 'documentari':5 'fanci':4 'frisbe':8 'lumberjack':11 'monkey':16 'must':13 'prejudic':2 'shark':19 'tank':20           |
| 5       | AFRICAN EGG       | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | 2006         | 1           | null                 | 6               | 2.99        | 130    | 22.99            | G      | 2006-02-15 05:03:42 | {"Deleted Scenes"}                 | 'african':1 'chef':11 'dentist':14 'documentari':7 'egg':2 'fast':5 'fast-pac':4 'forens':19 'gulf':23 'mexico':25 'must':16 'pace':6 'pastri':10 'psychologist':20 'pursu':17                      |

## Table #4 - film_category

<img width="564" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/142f2f78-d57a-41a0-b242-801058a672bf">

The 4th table in the ERD is `film_category` which shows the relationship between `film_id` and `category_id`.

Multiple films will appear in each relevant `category_id` which will map one-to-one with our next table in our ERD, the `category` table.

```sql
SELECT *
FROM dvd_rentals.film_category
LIMIT 5;
```

| film_id | category_id | last_update         |
|---------|-------------|---------------------|
| 1       | 6           | 2006-02-15 05:07:09 |
| 2       | 11          | 2006-02-15 05:07:09 |
| 3       | 6           | 2006-02-15 05:07:09 |
| 4       | 11          | 2006-02-15 05:07:09 |
| 5       | 8           | 2006-02-15 05:07:09 |

## Table #5 - category

<img width="567" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/9731317c-b789-45e1-9eca-7497c2cbccb7">

The 5th table in the ERD is the `category` table which simply contains a one to one mapping between `category_id` and the name of each category.

```sql
SELECT *
FROM dvd_rentals.category
LIMIT 5;
```

| category_id | name       | last_update         |
|-------------|------------|---------------------|
| 1           | Action     | 2006-02-15 04:46:27 |
| 2           | Animation  | 2006-02-15 04:46:27 |
| 3           | Children   | 2006-02-15 04:46:27 |
| 4           | Classics   | 2006-02-15 04:46:27 |
| 5           | Comedy     | 2006-02-15 04:46:27 |

## Table #6 - film_actor

<img width="562" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/dd9ae2da-2761-4b4c-9de7-6a68f210b8e0">

The 6th table in our ERD is `film_actor` which shows actors who appeared in each film based off related `actor_id` and `film_id` values.

An actor can appear in multiple films and a film can feature multiple actors. This relationship between values is what we usually call a “many-to-many” relationship - this is really important to note especially when we are dealing with joins.

Show all the films that `actor_id = 1` appeared in:

```sql
SELECT *
FROM dvd_rentals.film_actor
WHERE actor_id = 1;
```

| actor_id | film_id | last_update         |
|----------|---------|---------------------|
| 1        | 1       | 2006-02-15 05:05:03 |
| 1        | 23      | 2006-02-15 05:05:03 |
| 1        | 25      | 2006-02-15 05:05:03 |
| 1        | 106     | 2006-02-15 05:05:03 |
| 1        | 140     | 2006-02-15 05:05:03 |
| 1        | 166     | 2006-02-15 05:05:03 |
| 1        | 277     | 2006-02-15 05:05:03 |
| 1        | 361     | 2006-02-15 05:05:03 |
| 1        | 438     | 2006-02-15 05:05:03 |
| 1        | 499     | 2006-02-15 05:05:03 |
| 1        | 506     | 2006-02-15 05:05:03 |
| 1        | 509     | 2006-02-15 05:05:03 |
| 1        | 605     | 2006-02-15 05:05:03 |
| 1        | 635     | 2006-02-15 05:05:03 |
| 1        | 749     | 2006-02-15 05:05:03 |
| 1        | 832     | 2006-02-15 05:05:03 |
| 1        | 939     | 2006-02-15 05:05:03 |
| 1        | 970     | 2006-02-15 05:05:03 |
| 1        | 980     | 2006-02-15 05:05:03 |

## Table #7 - actor


<img width="564" alt="image" src="https://github.com/djeong95/Yelp_review_datapipeline/assets/102641321/8a072e2a-d45f-439f-8ac0-145164a5baa9">

The `actor` table simply shows the first and last name for each actor based off their unique `actor_id` - we can trace which films a specific actor appears in by joining this table onto the previously discussed `film_actor` table on the `actor_id` column.

```sql
SELECT *
FROM dvd_rentals.actor
LIMIT 5;
```
| actor_id | first_name | last_name     | last_update         |
|----------|------------|---------------|---------------------|
| 1        | PENELOPE   | GUINESS       | 2006-02-15 04:34:33 |
| 2        | NICK       | WAHLBERG      | 2006-02-15 04:34:33 |
| 3        | ED         | CHASE         | 2006-02-15 04:34:33 |
| 4        | JENNIFER   | DAVIS         | 2006-02-15 04:34:33 |
| 5        | JOHNNY     | LOLLOBRIGIDA  | 2006-02-15 04:34:33 |

# Conclusion

We have now covered all the raw datasets required to solve this case study as well as the relationships between the various tables by interpreting the entire entity-relationship diagram (ERD).

In the next tutorial - we will start diving into problem solving mode!