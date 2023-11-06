# Case Study 4: 
Here is the official introduction and problem statement for this case study:
>Introduction
>There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.
>
>Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides >to launch a new initiative - Data Bank!
>
>Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure >distributed data storage platform!
>
>Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a >few interesting caveats that go with this business model, and this is where the Data Bank team need your help!
>
>The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage >their customers will need.
>
>This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast >and plan for their future developments!
>
>Available Data
>The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get >you familiar with their tables.

Here is the ERD:

<img width="712" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/93aefcd5-0658-4478-847c-5ef452a64fed">

<b> Here are my solutions to the queries: </b>

# A. Customer Nodes Exploration

## 1. How many unique nodes are there on the Data Bank system?

Perform a count distinct of the number of nodes.

```sql
SELECT COUNT(DISTINCT node_id) AS count_nodes
FROM data_bank.plans
```
<img width="223" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/11fefd5a-8ac7-4498-91af-4bd89599c8c1">

## 2. What is the number of nodes per region?
Group by region id and perform a count distinct of the number of nodes.

```sql
SELECT region_id, COUNT(DISTINCT node_id) AS nodes_per_region
FROM data_bank.plans
GROUP BY region_id
ORDER BY region_id
```
<img width="479" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/cc2c4ca0-3698-4ba1-ba48-07f9b48c3344">

## 3. How many customers are allocated to each region?

Group by region_id and count the number of unique customers.

```sql
SELECT region_id, COUNT(DISTINCT customer_id) AS customer_count
FROM data_bank.plans
GROUP BY region_id
ORDER BY region_id
```

<img width="346" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/5c045639-d9d1-4704-8f31-699f4b2edc6a">


## 4.How many days on average are customers reallocated to a different node?
- Step 1- In the node_days CTE: For each customer, find the node id and the number of days they were placed in that node, by calculating the difference between end date and start date (then convert the difference to days using EXTRACT function).
- Step 2- Use the node_days CTE to sum up the number of days a particular customer was placed in a particular node, through a group by of customer_id and node_id.
- Step 3- Find the overall average for the number of days customers are placed in a certain node. The number of days a customer is one node is esentially the same as the number of days after which a customer is reallocated to a different node.

```sql
WITH days_in_node AS(
SELECT customer_id, node_id, EXTRACT(DAY FROM (end_date - start_date)) AS days
FROM data_bank.plans
WHERE end_date != '9999-12-31'
),
total_node_days AS (
  SELECT customer_id, node_id, SUM(days) AS sum_days_in_node
  FROM days_in_node
  GROUP BY customer_id, node_id
)
SELECT ROUND(AVG(sum_days_in_node),1) AS avg_days_in_node
FROM total_node_days
```

<img width="409" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/e10809ae-eff7-4571-94e3-bc44fc2d72da">


# B. Customer Transactions
## 1. What is the unique count and total amount for each transaction type?

Group by transaction type and find the count and total amount for transactions.

```sql
SELECT txn_type, COUNT(DISTINCT customer_id) AS count_transactions, SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type
ORDER BY txn_type
```

<img width="565" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/804c3b97-d516-4f6d-9562-36532d754d80">

## 2. What is the average total historical deposit counts and amounts for all customers?
- First: For each customer, find the number of deposits and average deposit amount, by performing a group by operation and filtering only for deposits.
- Use the above data to find the overall average of number of deposits and deposit amount, for all customers.

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
- Step 1: In a CTE, for each month, find the number of deposits, purchases and withdrawls made by each customer. Conditional aggregation through CASE statements is utilised to find the number of deposits, purchases and withdrawls easily.
- Step 2: Now, utilise the data in the CTE to find the number of customers who made more than 1 deposit and either 1 withdrawl OR one purchase per month.

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











