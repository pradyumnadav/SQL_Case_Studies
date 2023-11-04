# A. Customer Nodes Exploration

## 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT COUNT(DISTINCT node_id) AS count_nodes
FROM data_bank.plans
```
<img width="223" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/11fefd5a-8ac7-4498-91af-4bd89599c8c1">

## 2. What is the number of nodes per region?
```sql
SELECT region_id, COUNT(DISTINCT node_id) AS nodes_per_region
FROM data_bank.plans
GROUP BY region_id
ORDER BY region_id
```
<img width="479" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/cc2c4ca0-3698-4ba1-ba48-07f9b48c3344">

## 3. How many customers are allocated to each region?
```sql
SELECT region_id, COUNT(DISTINCT customer_id) AS customer_count
FROM data_bank.plans
GROUP BY region_id
ORDER BY region_id
```

<img width="346" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/5c045639-d9d1-4704-8f31-699f4b2edc6a">


## 4.How many days on average are customers reallocated to a different node?
```sql
WITH node_days AS(
SELECT customer_id, node_id, EXTRACT(DAY FROM (end_date - start_date)) AS days_in_node
FROM data_bank.plans
WHERE end_date != '9999-12-31'
),
total_node_days AS (
  SELECT customer_id, node_id, SUM(days_in_node)
  FROM node_days
  GROUP BY customer_id, node_id
)
SELECT ROUND(AVG(days_in_node),1) AS avg_days_in_node
FROM node_days
```

<img width="265" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/c86eca2c-f8a8-4750-85c5-828084460254">

# B. Customer Transactions
## 1. What is the unique count and total amount for each transaction type?
```sql
SELECT txn_type, COUNT(DISTINCT customer_id) AS count_transactions, SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type
ORDER BY txn_type
```

<img width="565" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/804c3b97-d516-4f6d-9562-36532d754d80">

## 2. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH each_customer_deposit AS(
SELECT customer_id, COUNT(*) AS deposit_counts, AVG(txn_amount) AS avg_deposit_amount_cust
FROM data_bank.customer_transactions
WHERE txn_type = 'deposit'
GROUP BY customer_id
)
SELECT ROUND(AVG(deposit_counts),0) AS avg_deposits, ROUND(AVG(avg_deposit_amount_cust),2) AS avg_deposit_amount
FROM each_customer_deposit
```
<img width="361" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/e415f7cc-d2c7-4999-a0ea-4d077e6553aa">

## 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH CTE AS(
SELECT EXTRACT(MONTH FROM txn_date) AS transaction_month, customer_id, SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_counts, SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_counts, SUM(CASE WHEN txn_type = 'withdrawl' THEN 1 ELSE 0 END) AS withdrawl_counts
FROM data_bank.customer_transactions
GROUP BY transaction_month, customer_id
ORDER BY customer_id
)
SELECT transaction_month, COUNT(customer_id) AS number_customers
FROM CTE
WHERE deposit_counts > 1 AND (purchase_counts = 1 OR withdrawl_counts = 1)
GROUP BY transaction_month
ORDER BY transaction_month
```

<img width="419" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/776f65e1-897b-499d-92cf-ee9dc5d40f2b">











