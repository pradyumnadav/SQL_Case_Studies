# Data Cleaning:
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
```sql
SELECT DISTINCT FORMAT_DATE('%A', week_date) AS day_of_week
FROM data_mart.clean_weekly_sales
ORDER BY day_of_week;
```

<img width="301" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/b83a7eff-425a-477e-93b7-94327aec68d4">


## 2. What range of week numbers are missing from the dataset?
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
```sql
SELECT month_number, ROUND(SUM(CASE WHEN platform = 'Shopify' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS shopify_sales_pct,
ROUND(SUM(CASE WHEN platform = 'Retail' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS retail_sales_pct
FROM data_mart.clean_weekly_sales
GROUP BY month_number
ORDER BY month_number ASC
```
<img width="518" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/8a58439c-48aa-40f8-a9e3-f3497f32333a">

## 7. What is the percentage of sales by demographic for each year in the dataset?
```sql
SELECT month_number, ROUND(SUM(CASE WHEN demographic = 'Couples' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS couples_sales_pct, ROUND(SUM(CASE WHEN demographic = 'Families' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS families_sales_pct, ROUND(SUM(CASE WHEN demographic = 'Unknown' THEN sales ELSE 0 END)/SUM(sales) * 100,1) AS unknown_sales_pct
FROM data_mart.clean_weekly_sales
GROUP BY month_number
ORDER BY month_number ASC
```
<img width="594" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/5686a2b1-ee01-45c5-8923-0347be47b843">

## 8. Which age_band and demographic values contribute the most to Retail sales?
```sql
SELECT age_band, demographic, SUM(sales) AS retail_sales, ROUND((SUM(sales)/ SUM(SUM(sales)) OVER () * 100),1) AS percent_contribution
FROM data_mart.clean_weekly_sales
GROUP BY age_band, demographic
ORDER BY percent_contribution DESC
```

<img width="822" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/794b62bf-0805-4cbd-b092-301f97cbedde">











