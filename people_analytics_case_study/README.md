# 1. People Analytics Case Study

![image](https://github.com/djeong95/SQL_learning/assets/102641321/17458dd4-793f-45fd-b9cf-dd33cd8454cb)

People Analytics or HR Analytics is an increasingly popular focus area for data professionals. Many business and people decisions which were traditionally based off senior management gut feels and intuition are starting to become more data-driven, despite what the above Dilbert comic depicts!

There is a lot of available high quality data in this field, and often the people insights that are discovered can lead to very important and impactful business decisions - often coming with full endorsement and support from senior executives such as the CEO or Chief of Staff.

In our second complete SQL case study - we will assist HR Analytica to construct datasets to answer basic reporting questions and also feed their bespoke People Analytics dashboards.

# 2. Final Outputs
Let’s first inspect what our final visualized data outputs will look like in dashboard format.

We will be generating all the data points required to feed the following 2 analytics outputs below:

## 2.1. People Analytics Dashboard

![image](https://github.com/djeong95/SQL_learning/assets/102641321/951bfca6-82e6-4897-9b46-a03dfb547296)

## 2.2. Employee Deep Dive

![image](https://github.com/djeong95/SQL_learning/assets/102641321/bab6233f-95f9-486f-bcf5-6e8ee1f8c9ea)

# 3. Key Technical Requirements

HR Analytica team requires 2 separate analytical views to be created using a single SQL script for two separate data assets that can be used for reporting purposes.

A current snapshot of the information is required to power HR Analytica’s People Analytics dashboard and Employee Deep Dive shown above.

The following data requirements is as follows:

## 3.1. Dashboard Data Components


**Company Level Insights**

- Total number of employees
- Average company tenure in years
- Gender ratios
- Average payrise percentage and amount

**Department Level Insights**

- Number of employees in each department
- Current department manager tenure in years
- Gender ratios
- Average payrise percentage and amount

**Title Level Insights**

- Number of employees with each title
- Minimum, average, standard deviation of salaries
- Average total company tenure
- Gender ratios
- Average payrise percentage and amount

A historic data asset is also required by HR Analytica so their People Analytics team can perform deep dives into a specific employee’s history. This analysis is used for decision making when it comes to pay rises and promotions.

## 3.2. Deep Dive Data Components

**Individual Employee Deep Dive**

- See all the various employment history ordered by effective date including salary, department, manager and title changes
- Calculate previous historic payrise percentages and value changes
- Calculate the previous position and department history in months with start and end dates
- Compare an employee’s current salary, total company tenure, department, position and gender to the average benchmarks for their current position

# 4. Our Approach

There is clearly a lot to unpack from the business requirements so let’s first break up each component into digestable components so we can focus our learning.

1. Take inventory of all the tables in the employees schema with special attention on relationships between tables
1. Explore the date fields which require updating and use Data Manipulation Language (DML)
1. Understand the difference between temporary tables, permanent tables, views and materialized views as reusable data assets
1. Lightly touch upon data design with a particular focus on slow changing dimensions (SCD)
1. Identify all relevant columns required to implement the current employee view
1. Use LEAD and LAG window functions to calculate comparisons for payrise changes
1. Write queries to support HR Analytica’s data team for current company, department and title level reporting scenarios
1. Adjust current employee view to incorporate historic data
1. Create company, tenure and position benchmarks table to join onto for further comparisons
1. Refactor SQL views with optimized design for efficient computation

We’ve got a lot to do - so let’s dive right into it!

