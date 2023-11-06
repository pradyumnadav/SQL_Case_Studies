# Case Study 1: Danny's Diner
Here is the official introduction and problem statement for this case study:
> Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.
>
>Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
>
>Problem Statement:
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.
>
>He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.
>
>Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!
>
>Danny has shared with you 3 key datasets for this case study:
>
> sales, menu & members
> You can inspect the entity relationship diagram and example data below.

Here is the ERD Diagram:

<img width="671" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/42af6e38-b3bc-4c67-80a5-13a37cd497d4">


<b> This case study contains 10 questions, to be answered through SQL queries. Here are the questions and my solutions:


# 1. What is the total amount spent by each customer? 

- The Sales table contains the customer id, order date and products id for the products bought. The Menu table contains the product id and their prices. Therefore, Join the Sales table with the Menu table to group by each customer and find the total amount spent.
```sql
SELECT S.customer_id, SUM(M.price) AS Amount_Spent
FROM dannys_diner.sales S JOIN dannys_diner.menu M
ON s.product_id = M.product_id
GROUP BY S.customer_id
ORDER BY Amount_Spent
```
<img width="429" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/c978dc8f-9d63-441f-b858-9c1248984a65">


#2. How many days has each customer visited the restaurant?

A distinct count of the order date, grouped by each customer id, will return the amount of days the customer has visited the restaurant.

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS Days_Visited
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY Days_Visited DESC;
```
<img width="436" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/beee134f-3cf1-42f0-828c-4164dd1d656b">

#3. What was the first item ordered by each customer?

A dense rank, when partitioned by the Customer id and ordered by the order date in ascending order, will assign a rank to each order placed by the customer, with rank 1 being the first ever order placed by the customer. If a customer has ordered multiple items in their first ever orders, dense rank assigns 1 to each item, so that possibility has also been accounted for.
- A CTE is used to join the sales and menu tables and assign the ranks to each customer's orders based on the order date. The CTE is then filtered out by rank = 1, to show only the items ordered in the customer's first ever order.

```sql
WITH CTE AS(
  SELECT S.customer_id, M.product_name, DENSE_RANK() OVER(PARTITION BY S.customer_id ORDER BY S.order_date) AS dense_rank
  FROM dannys_diner.sales S JOIN dannys_diner.menu M ON s.product_id = M.product_id

)
SELECT customer_id, product_name
FROM CTE
WHERE dense_rank = 1
GROUP BY customer_id, product_name
```

<img width="531" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/270013f2-4147-4432-99fc-5058b56f30f7">

#4. What is the most purchased item on the menu and how many times was it purchased by all customers?

Group by the product name and count the number of times each product was ordered. Order the query by purchase count in <b> Descending order and show only the first row.

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

In a CTE, Group by each customer id and Product to find a count of the number of times that customer ordered a particular product. Also assign a rank to the purchase counts using the RANK() function. Then filter the CTE to find only purchases with Rank 1 for each customer.

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

All 3 tables in the ERD were required to answer this question. After the join, we get one unified table which contains the information about each customer's orders and their membership details. Use the membership join date and order date, to find only the orders placed by each customer <b> after </b> they became a member. Use the RANK() window function to assign a rank to each customer's order, in a CTE. Filter the CTE to only display the first ranked orders, which are the first orders placed by the customer after becoming a member.

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

This question is a slight twist on the previous question. So the orders of each customer and their membership join date are used to find all orders placed by the customer <b> before they became a member. Then find the item purchased just before becoming a member using the RANK function.

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

Again, the membership join date can be used to find the orders placed by customers before they became a member. Then find the number of items purchased and total amount spent.

```sql
SELECT S.customer_id, COUNT(S.order_date) AS Total_Items, SUM(M.price) AS Amount_Spent
FROM dannys_diner.sales S JOIN dannys_diner.menu M ON S.product_id = M.product_id JOIN dannys_diner.members MS ON 
S.customer_id = MS.customer_id 
WHERE S.order_date < MS.join_date
GROUP BY S.customer_id
```

<img width="529" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/8cbe626d-d762-4b1e-a915-ae7f460c48eb">

#9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

This question was solved using two CTEs. 
- First: Join the sales and menu tables to find the customer id and the amount of money they spent on each product, through a group by done on customer id and product name.
- Second: Assign 10 points to each 1$ spent on other products and 20 points to each 1$ spent on Sushi, using the Case statement. Name the amount of points scored by each customer, per product as "Points". 
- Then use the second CTE's results to sum up the total amount of points scored by each customer.


```sql
WITH CTE AS(
  SELECT S.customer_id, M.product_name, SUM(M.price) AS sum
FROM dannys_diner.sales S JOIN dannys_diner.menu M ON S.product_id = M.product_id
GROUP BY S.customer_id, M.product_name
),
CalculateTotalPoints AS(
SELECT customer_id, product_name, CASE WHEN product_name ='sushi' THEN sum * 20 ELSE sum*10 END AS Points
FROM CTE
)
SELECT customer_id, SUM(Total_Points) AS Points_Scored
FROM CalculateTotalPoints
GROUP BY customer_id
ORDER BY Points_Scored DESC
```
<img width="437" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/31da905d-9674-4936-9e15-2b792153eb12">









