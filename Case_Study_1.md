# 1. What is the total amount spent by each customer? 
```sql
SELECT S.customer_id, SUM(M.price) AS Amount_Spent
FROM dannys_diner.sales S JOIN dannys_diner.menu M
ON s.product_id = M.product_id
GROUP BY S.customer_id
ORDER BY Amount_Spent
```
<img width="429" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/c978dc8f-9d63-441f-b858-9c1248984a65">


#2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS Days_Visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY Days_Visited DESC;
```
<img width="436" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/beee134f-3cf1-42f0-828c-4164dd1d656b">

#3. What was the first item ordered by each customer?
WITH CTE AS(
  SELECT S.customer_id, M.product_name, DENSE_RANK() OVER(PARTITION BY S.customer_id ORDER BY S.order_date) AS dense_rank
  FROM dannys_diner.sales S JOIN dannys_diner.menu M ON s.product_id = M.product_id

)
SELECT customer_id, product_name
FROM CTE
WHERE dense_rank = 1
GROUP BY customer_id, product_name

<img width="531" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/270013f2-4147-4432-99fc-5058b56f30f7">

#4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT M.product_name ,COUNT(S.order_date) AS Purchase_Count
FROM dannys_diner.sales S JOIN dannys_diner.menu M
ON S.product_id = M.product_id
GROUP BY M.product_name
ORDER BY Purchase_Count DESC
LIMIT 1
```
<img width="427" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/0ce64511-e47a-4e6b-ab48-28611697ae7b">

#5. Which item was the most popular for each customer?

```sql
WITH Customer_PopularProducts AS(
SELECT S.customer_id ,M.product_name AS most_popular_product, COUNT(S.order_date) AS Purchase_Count, RANK() OVER(PARTITION BY S.customer_id ORDER BY COUNT(S.order_date) DESC) AS purchase_count_rank
FROM dannys_diner.sales S JOIN dannys_diner.menu M
ON S.product_id = M.product_id
GROUP BY S.customer_id, M.product_name
ORDER BY S.customer_id, Purchase_Count DESC
)
SELECT customer_id, most_popular_product, Purchase_count 
FROM Customer_PopularProducts
WHERE purchase_count_rank = 1
ORDER BY customer_id
```
<img width="614" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/1f9a4dc1-05b7-451d-89e6-0879dec7fb89">


#6. Which item was purchased first by the customer after they became a member?

```sql
WITH CTE AS(
  SELECT S.customer_id, M.product_name, MS.join_date, RANK() OVER(PARTITION BY S.customer_id ORDER BY S.order_date) AS rank
  FROM dannys_diner.sales S JOIN dannys_diner.menu M
  ON S.product_id = M.product_id JOIN dannys_diner.members MS ON
  S.customer_id = MS.customer_id WHERE S.order_date > MS.join_date
)
SELECT customer_id, join_date, product_name AS FirstProductOrderedAfterJoining
FROM CTE
WHERE rank = 1
ORDER BY customer_id
```

<img width="694" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/2bf36c58-e4be-4d38-9530-03a59be083aa">

#7. Which item was purchased just before the customer became a member?
```sql
WITH CTE AS(
  SELECT S.customer_id, M.product_name,MS.join_date, RANK() OVER(PARTITION BY S.customer_id ORDER BY S.order_date) AS rank 
  FROM dannys_diner.sales S JOIN dannys_diner.menu M
  ON S.product_id = M.product_id JOIN dannys_diner.members MS ON
  S.customer_id = MS.customer_id WHERE S.order_date < MS.join_date
)
SELECT customer_id, join_date, product_name AS LastOrderBeforeMembership
FROM CTE
WHERE rank = 1
ORDER BY customer_id
```

<img width="647" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/47cbd269-4978-461d-bbd1-0774fd3f5110">

#8. What is the total items and amount spent for each member before they became a member?

```sql
WITH CTE AS(
  SELECT S.customer_id, COUNT(S.order_date) AS Total_Items,
  SUM(M.price) AS Amount_Spent
  FROM dannys_diner.sales S JOIN dannys_diner.menu M
  ON S.product_id = M.product_id JOIN dannys_diner.members MS ON
  S.customer_id = MS.customer_id 
  WHERE S.order_date < MS.join_date
  GROUP BY S.customer_id
)
SELECT * FROM CTE
```

<img width="529" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/8cbe626d-d762-4b1e-a915-ae7f460c48eb">

#9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH CTE AS(
  SELECT S.customer_id, M.product_name, SUM(M.price) AS sum
FROM dannys_diner.sales S JOIN dannys_diner.menu M ON S.product_id = M.product_id
GROUP BY S.customer_id, M.product_name
),
CalculateTotalPoints AS(
  
  SELECT customer_id, product_name, CASE WHEN product_name ='sushi' THEN sum * 20 ELSE sum*10 END AS Total_Points
FROM CTE
)
SELECT customer_id, SUM(Total_Points) AS Points_Scored
FROM CalculateTotalPoints
GROUP BY customer_id
ORDER BY Points_Scored DESC
```
<img width="437" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/31da905d-9674-4936-9e15-2b792153eb12">









