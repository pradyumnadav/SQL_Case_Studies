# Case Study 5: Data Mart

Here is the official introduction and problem statement for this case study:
>Introduction
>Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh >produce - Danny is asking for your support to analyse his sales performance.
>
>In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in >every single step from the farm all the way to the customer.
>
>Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.
>
>The key business question he wants you to help him answer are the following:
>
>What was the quantifiable impact of the changes introduced in June 2020?
>Which platform, region, segment and customer types were the most impacted by this change?
>What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?
>Available Data
>For this case study there is only a single table: data_mart.weekly_sales
>
>The Entity Relationship Diagram is shown below with the data types made clear, please note that there is only this one table - hence >why it looks a little bit lonely!

Here is the ERD:

<img width="359" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/2969bcf6-00bc-4678-8880-39d70966b693">


# Data Cleaning:

Create a new table called clean_weekly_sales which cleans the original data and creates new information like the Age band and demographics.

```sql
CREATE OR REPLACE TABLE data_mart.clean_weekly_sales AS
SELECT 
  week_date, region, platform, customer_type, transactions, sales,
  EXTRACT(WEEK FROM week_date) AS week_number, 
  EXTRACT(MONTH FROM week_date) AS month_number, 
  EXTRACT(YEAR FROM week_date) AS calendar_year, 
  CASE 
    WHEN segment LIKE '%_1' THEN 'Young Adults' 
    WHEN segment LIKE '%_2' THEN 'Middle Aged' 
    WHEN segment LIKE '%_3' OR segment LIKE '%_4' THEN 'Retirees' 
    WHEN segment = 'null' THEN 'Unknown' 
  END AS age_band, 
  CASE 
    WHEN segment LIKE 'C%' THEN 'Couples' 
    WHEN segment LIKE 'F%' THEN 'Families' 
    WHEN segment = 'null' THEN 'Unknown' 
  END AS demographic, 
  CASE 
    WHEN segment = 'null' THEN 'Unknown' 
    ELSE segment 
  END AS segment, 
  ROUND((sales / transactions), 2) AS avg_transaction
FROM 
  data_mart.weekly_sales;
```

# Data Analysis:
## 1. What day of the week is used for each week_date value?
FORMAT_DATE function can help find the day of week.

```sql
SELECT DISTINCT FORMAT_DATE('%A', week_date) AS day_of_week
FROM data_mart.clean_weekly_sales
ORDER BY day_of_week;
```

<img width="301" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/b83a7eff-425a-477e-93b7-94327aec68d4">


## 2. What range of week numbers are missing from the dataset?
- In a CTE, use the generate array function to generate a series between 1 to 52, to indicate weeks.
- Use the unnest function in Big Query (https://count.co/sql-resources/bigquery-standard-sql/unnest) to convert the array containing week numbers (1 to 52) into a table. The original data is used in a left join with the unnested week numbers data.
- Compare the week numbers in the original data to the weeks 1 to 52 and display the weeks that do not match.

```sql
WITH weeks_of_year AS (
  SELECT GENERATE_ARRAY(1, 52) AS weeks
)
SELECT DISTINCT week
FROM weeks_of_year W, UNNEST(W.weeks) AS week
LEFT JOIN data_mart.clean_weekly_sales S ON week = S.week_number
WHERE S.week_number IS NULL
```

<img width="357" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/f266f81b-a949-476a-9d0c-16fb99f9c3df">

## 3. How many total transactions were there for each year in the dataset?
Group by the calendar year and count the total number of transactions in each year.

```sql
SELECT calendar_year,COUNT(*) AS number_transactions
FROM data_mart.clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year
```
<img width="420" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/0aef43f5-a0ca-4bf7-9f84-1a516a7855c0">


## 4. What is the total sales for each region for each month?


```sql
SELECT region, month_number, SUM(sales) AS total_sales
FROM data_mart.clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, total_sales DESC
```

<img width="550" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/9f76c625-aa3f-4d9d-8651-4b340c3a8fc3">

<img width="586" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/68547543-45ce-46e0-90c6-a76a08e7ea53">

## 5. What is the total count of transactions for each platform
```sql
SELECT platform, COUNT(*) AS count_transactions
FROM data_mart.weekly_sales
GROUP BY platform
ORDER BY count_transactions DESC
```
<img width="468" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/285ccff5-84fe-4c3d-9f4b-9c289ec232fd">

## 6. What is the percentage of sales for Retail vs Shopify for each month?
The conditional aggregation comes in handy again and simplifies the query substantially. 
- Use CASE statements to sum up the sales when the condition platform = 'shopify' is true- this gives the sum of sales through the shopify platform. Divide this by overall total sales to calculate the contribution made by shopify platform towards total sales, in percentage.
- Repeat the same calculation as above but for platform = retail.

```sql
SELECT month_number, ROUND(SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS shopify_sales_pct,
ROUND(SUM(CASE WHEN platform = 'Retail' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS retail_sales_pct
FROM data_mart.clean_weekly_sales
GROUP BY month_number
ORDER BY month_number ASC
```
<img width="518" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/8a58439c-48aa-40f8-a9e3-f3497f32333a">

The retail platform contributes to oer 97% of the sales in every month.

## 7. What is the percentage of sales by demographic for each year in the dataset?
Use conditional aggregation logic again, but this time calculate the contributions made towards sales by demographic factors.

```sql
SELECT month_number, ROUND(SUM(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS couples_sales_pct, ROUND(SUM(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS families_sales_pct, ROUND(SUM(CASE WHEN demographic = 'Unknown' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS unknown_sales_pct
FROM data_mart.clean_weekly_sales
GROUP BY month_number
ORDER BY month_number ASC
```
<img width="594" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/5686a2b1-ee01-45c5-8923-0347be47b843">

## 8. Which age_band and demographic values contribute the most to Retail sales?
- Group by age_band and demographic to calculate the amount of sales contributed.
- Perform a SUM(sales) OVER() operation: This is done with a blank OVER() i.e without any partition, meaning that the sum of sales is calculated for the entire result set in the current window. This value is essentially the overall total sum of sales. See explanation here: https://www.vertica.com/docs/9.3.x/HTML/Content/Authoring/AnalyzingData/Optimizations/AvoidingSingle-NodeExecutionByAvoidingEmptyOVERClauses.htm
- Divide the individual sum of sales for each demographic-age_band with the overall total sum of sales, to get the percent contribution.

```sql
SELECT age_band, demographic, SUM(sales) AS retail_sales, ROUND((SUM(sales)/ SUM(SUM(sales)) OVER () * 100),1) AS percent_contribution
FROM data_mart.clean_weekly_sales
GROUP BY age_band, demographic
ORDER BY percent_contribution DESC
```

<img width="822" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/794b62bf-0805-4cbd-b092-301f97cbedde">











