# Marketing Analysis Case Study

In this second section of the technical components for this Serious SQL course we turn our attention to business problem solving within the marketing analytics space through a guided case study.

The aim of this case study is to further your learning by providing you with realistic business problems to solve utilizing only SQL.

These examples should both challenge you and expand your data skills by demonstrating how we can deploy simple algorithmic thinking, reverse engineering and most importantly - thinking in SQL - to solve problems!

# Why Case Studies?

Honestly, when I first designed this section of the course, I thought that introducing each SQL concept in a sequential manner was the best course of action, and case studies were just an afterthought!

However - when I thought about it a bit deeper and reflected on how I learnt many of the SQL skills and concepts on my own data journey - a light bulb went off in my mind.

I realised that my skills improved almost exponentially as I was exposed to different scenarios that required unique applications of various techniques that I had never seen before.

This sparked my desire to create as-realistic as possible business problems to help you upskill quickly in the same way that I did!

To make these examples as close to real life as possible, new techniques will sprinkled throughout this section in a totally non-linear path.

In the real world - you won’t always have the luxury to learn everything sequentially like we’re taught in school, so this bouncing around from concept to concept will help you familiarize yourself with the current challenges in the data world!

The focus of these case studies is to help improve your business awareness and continue honing your SQL skills by reverse engineering expected outputs from raw data inputs.

Actually, this exact method of working backwards from an expected output from certain inputs is pretty much the same way we would tackle similar problems in the workplace. This is not just when using SQL for data analytics, but this extends to a wider range of programming problems where we may know what we need as outputs, and we simply need to figure out how to get from A to B!

# Learning Outcomes

The following SQL skills and concepts will be covered in this section of the Serious SQL course:

1. Learning how to interpret ERDs for data literacy and business context (entity-relationship diagrams)
- Identify key columns of interest and how they are linked to other tables via foreign keys
- Use ERDs to analyze the data types for different columns in database tables
- Understand data context for various tables that cause inherent natural relationships between fields
2. Introduction to all types of table joining operations
- Simple joins: left, inner
- More involved joins: cross, anti, left-semi, full outer
- Combination set operations: union, union all, intersect, except
3. Practical application of table joins
- Joining multiple tables to combine disparate datasets into a single data structure
- Joining interim SQL outputs for more advanced group-by, split, merge data hacking strategies
- Performing table joins that use 2 or more table references in the ON condition
- Using anti joins to exclude records in a sequential fashion
4. Filtering, window functions and aggregates for analytics and ranking
- Using ROW_NUMBER to effecively rank order records with equal ties
- Using WHERE filters to extract ranked records
- Using multiple aggregate functions with different target partitions and ordering expressions for efficient data analysis
- Using aggregate group by clauses to generate unique customer level insights
5. Case statements for data transformation and manipulation
- Pivoting datasets from long to wide using MAX and CASE WHEN
- Manipulating actual data values using conditional logic for business translation purposes
6. SQL scripting basics
- Designing SQL workflows which can be easily understood and implemented
- Managing multiple dependencies for downstream table joining operations by using temporary tables to store interim datasets
7. Manipulating text data
- Converting text columns to title case
- Combining multiple text and numeric data type columns into a single text expression
