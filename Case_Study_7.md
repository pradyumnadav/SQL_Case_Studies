# Case Study 7: Balanced Tree Clothing Co.
Here is the official introduction and problem statement for this case study:
>Introduction
>Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!
>
>Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.
>
>Available Data:
For this case study there is a total of 4 datasets for this case study - however you will only need to utilise 2 main tables to solve all of the regular questions, and the additional 2 tables are used only for the bonus challenge question!
>

# High Level Sales Analysis

## 1. What was the total quantity sold for all products?
```sql
SELECT SUM(qty) AS quantity_sold
FROM balanced_tree.sales
```
<img width="277" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/615e31f9-91d2-4d0a-a483-41a73e69275f">

## 2. What is the total generated revenue for all products before discounts?
```sql
SELECT SUM((qty * price)) AS total_revenue_before_discount
FROM balanced_tree.sales

```
<img width="359" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/0a7432ab-29b2-470a-9848-198f13a616fd">

## 3. What was the total discount amount for all products?
```sql
SELECT SUM(discount) AS total_discount
FROM balanced_tree.sales
```

<img width="280" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/304df223-2c25-4fe6-9b0d-faa5addab1f3">

# Transaction Analysis

## 1. How many unique transactions were there?

```sql
SELECT COUNT(DISTINCT txn_id) AS count_transactions
FROM balanced_tree.sales
```

<img width="251" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/f5685435-bda8-4715-8f35-251e617f993b">


## 2. What is the average unique products purchased in each transaction?
- Find the number of products purchased in each transaction i.e quantity. Then find the overall average

```sql

WITH CTE AS(
SELECT txn_id, SUM(qty) AS num_prods
FROM balanced_tree.sales
GROUP BY txn_id
)
SELECT ROUND(AVG(num_prods),1) AS unique_prods_purchased 
FROM CTE
```

<img width="295" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/a201d1c2-22f6-405d-a984-9bafd670a82b">


## 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

- Discount is given in percentage. Therefore revenue = quantity * price * (1 - discount / 100)
- The PERCENTILE_CONT() function can be used to find the 25th, 50th and 75th percentiles of revenues.

```sql
WITH transaction_revenues AS(
  SELECT txn_id, SUM(qty * price * (1 - discount/100)) AS revenue
  FROM balanced_tree.sales
  GROUP BY txn_id
)

SELECT PERCENTILE_CONT(revenue, 0.25) OVER() AS percentile_25 , PERCENTILE_CONT(revenue, 0.5) OVER() AS percentile_50, PERCENTILE_CONT(revenue, 0.75) OVER() AS percentile_75
FROM transaction_revenues
LIMIT 1
```
<img width="517" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/826fe6d1-41ff-48e3-833c-c611c80cffb3">

## 4. What is the average discount value per transaction?

```sql
SELECT ROUND(AVG(discount_value),2) AS avg_discount_value
FROM (SELECT txn_id, SUM((qty * price * (discount/100))) AS discount_value
FROM balanced_tree.sales
GROUP BY txn_id)
```

<img width="305" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/a037a007-7376-4636-a1ed-7a58e5d82ff9">

## 5. What is the percentage split of all transactions for members vs non-members?
```sql
SELECT ROUND(COUNT(DISTINCT CASE WHEN member = TRUE THEN txn_id END)/COUNT(DISTINCT txn_id) * 100, 1) AS pct_member, ROUND(COUNT(DISTINCT CASE WHEN member = FALSE THEN txn_id END)/COUNT(DISTINCT txn_id) * 100,1) AS pct_non_member
FROM balanced_tree.sales
```
<img width="435" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/34acb80b-b794-4461-8f5a-4f4737b5af2b">

## 6. What is the average revenue for member transactions and non-member transactions?
```sql
WITH total_revenues AS(
SELECT member, txn_id, SUM(qty * price * (1 - discount/100)) AS revenue
FROM balanced_tree.sales
GROUP BY member, txn_id
)
SELECT member, ROUND(AVG(revenue),2) AS avg_revenue
FROM total_revenues
GROUP BY member
```
<img width="436" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/caea2b77-22bb-4761-8d86-fb1ca34ed5ca">

# Product Analysis

## 1. What are the top 3 products by total revenue before discount?
```sql
SELECT P.product_name, SUM(S.qty * S.price) AS revenue_prior_discount
FROM balanced_tree.sales S JOIN balanced_tree.product_details P ON S.prod_id  = P.product_id
GROUP BY P.product_name
ORDER BY revenue_prior_discount DESC
LIMIT 3
```

<img width="446" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/a385fc84-e600-48d2-a8b6-3671cfa7f3ea">

## 2. What is the total quantity, revenue for each segment?

```sql
SELECT P.segment_id, P.segment_name, SUM(S.qty) AS quantity, ROUND(SUM(S.qty * S.price * (1 - S.discount/100)),1) AS revenue
FROM balanced_tree.sales S JOIN balanced_tree.product_details P ON S.prod_id  = P.product_id
GROUP BY P.segment_id, P.segment_name
ORDER BY P.segment_id
```

<img width="743" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/25df823c-2f2c-4ba5-9bc0-d16566e77fb3">


## 3. What is the top selling product for each segment?

```sql

WITH segment_product_revenues AS(
SELECT P.segment_name,P.product_name, SUM(S.qty * S.price * (1 - S.discount/100)) AS revenue, RANK() OVER(PARTITION BY P.segment_name ORDER BY SUM(S.qty) DESC) AS revenue_rank, SUM(S.qty) AS quantity
FROM balanced_tree.sales S JOIN balanced_tree.product_details P ON S.prod_id  = P.product_id
GROUP BY P.segment_name, P.product_name
ORDER BY P.segment_name
)
SELECT segment_name, product_name, ROUND(revenue,0) AS revenue, quantity AS quantity_sold
FROM segment_product_revenues
WHERE revenue_rank = 1
ORDER BY revenue DESC
```

<img width="754" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/12d9755c-710e-410d-98f4-3c95c9b6b56f">

## 4. What is the total quantity, revenue and discount for each category?
```sql
SELECT P.category_name,SUM(S.qty) AS quantity, ROUND(SUM(S.qty * S.price * (1 - S.discount/100)),1) AS revenue, ROUND(SUM((S.qty * S.price) * S.discount/100),1) AS discount
FROM balanced_tree.sales S JOIN balanced_tree.product_details P ON S.prod_id  = P.product_id
GROUP BY P.category_name
```

<img width="720" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/659b7d10-cd8c-40d4-8189-0f1611356f5b">


## 5. What is the top selling product, by revenue, for each category?
```sql
WITH revenue_per_category AS(
SELECT P.category_name, P.product_name, SUM(S.qty * S.price) AS revenue, RANK() OVER(PARTITION BY category_name ORDER BY SUM(S.qty*S.price) DESC) AS revenue_rank
FROM balanced_tree.sales S JOIN balanced_tree.product_details P ON S.prod_id = P.product_id
GROUP BY P.category_name, P.product_name
)
SELECT category_name, product_name AS top_selling_product, revenue AS revenue
FROM revenue_per_category
WHERE revenue_rank = 1
ORDER BY category_name

```

<img width="680" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/c9648bd0-6dba-4cc6-a7f3-cc134d540da9">


## 6. What is the percentage split of revenue by product for each segment?
```sql
WITH segment_revenues AS(
SELECT P.segment_name, P.product_name, SUM(S.qty * S.price * (1 - S.discount/100)) AS revenue, SUM(SUM(S.qty * S.price * (1 - S.discount/100))) OVER(PARTITION BY P.segment_name) AS total_revenue
 FROM balanced_tree.product_details P JOIN balanced_tree.sales S ON P.product_id = S.prod_id 
 GROUP BY P.segment_name, P.product_name
)
SELECT segment_name, product_name, ROUND((revenue/total_revenue) * 100,1) AS percentage_revenue
FROM segment_revenues
ORDER BY segment_name, percentage_revenue DESC
```
<img width="671" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/a14a5152-7fa3-4f7c-9c77-6fc7f1f7876d">
<img width="650" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/59d8f6e4-1ac2-403c-8754-c837356c49f4">


## 7. What is the percentage split of revenue by segment for each category?
```sql
WITH segment_revenues AS(
SELECT P.category_name , P.segment_name, SUM(S.qty * S.price) AS revenue, SUM(SUM(S.qty * S.price)) OVER(PARTITION BY P.category_name) AS total_revenue
 FROM balanced_tree.product_details P JOIN balanced_tree.sales S ON P.product_id = S.prod_id 
 GROUP BY P.category_name , P.segment_name
)
SELECT category_name, segment_name, ROUND((revenue/total_revenue) * 100,1) AS percentage_revenue
FROM segment_revenues
ORDER BY segment_name, percentage_revenue DESC
```

<img width="712" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/eddb4d09-e962-4456-b600-26a16939c9af">


## 8. What is the percentage split of total revenue by category?
```sql
WITH category_revenues AS(
SELECT P.category_name, SUM(S.qty * S.price * (1-S.discount/100)) AS revenue
FROM balanced_tree.product_details P JOIN balanced_tree.sales S ON P.product_id = S.prod_id 
GROUP BY P.category_name
),
total_revenues AS(
  SELECT SUM(qty * price * (1 - discount/100)) AS total_revenue
  FROM balanced_tree.sales
)
SELECT category_name, ROUND((revenue/total_revenue) * 100,1) AS pct_revenue_contributed
FROM category_revenues, total_revenues
ORDER BY pct_revenue_contributed DESC
```

<img width="433" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/0fb1d54f-25aa-4325-8ba3-d164429695b7">














