# Case Study 6: Clique Bait
Here is the official introduction and problem statement for this case study:
>Introduction
>Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!
>
>In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.
>
>Available Data:
>For this case study there is a total of 5 datasets which you will need to combine to solve all of the questions.

Here are the questions and my solutions:

# Digital Analysis
## 1. How many users are there?
```sql
SELECT COUNT(DISTINCT user_id) AS number_of_users
FROM clique_bait.users
```
<img width="309" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/bdb43ac5-e50a-43e3-bfef-53de60965d44">

## 2. How many cookies does each user have on average?
```sql
SELECT user_id, COUNT(DISTINCT cookie_id) AS number_cookies
FROM clique_bait.users
GROUP BY user_id
ORDER BY user_id
```
<img width="446" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/b8da85a7-68dc-48fd-a816-85c0adf0759e">

## 3. What is the unique number of visits by all users per month?
From event_time column, extract the month and then count the number of visits each month.

```sql
SELECT EXTRACT(MONTH FROM E.event_time) AS month_of_visit, COUNT(DISTINCT E.visit_id) AS number_of_visits
FROM clique_bait.users U JOIN clique_bait.events E ON U.cookie_id = E.cookie_id
GROUP BY month_of_visit
ORDER BY month_of_visit ASC
```

<img width="450" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/97ccb977-8cfe-4ffa-99c2-acc99615e07a">


## 4. What is the number of events for each event type?

```sql
SELECT event_type, COUNT(*) AS number_events
FROM clique_bait.events
GROUP BY event_type
ORDER BY event_type
```

<img width="428" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/b96a1bd9-e955-4e50-80fc-2b3a0a6c5efb">


## 5. What is the percentage of visits which have a purchase event?
- Event type = 3 are purchase events, use conditional aggregation to count the number of events that involve a purchase. Divide this value by the total number of events to find the percentage of visits which have a purchase event.

```sql
SELECT ROUND(SUM(CASE WHEN EI.event_type = 3 THEN 1 ELSE 0 END)/COUNT(DISTINCT E.visit_id) * 100,2) AS percent_purchase_events
FROM clique_bait.events E JOIN clique_bait.event_identifier EI ON E.event_type = EI.event_type
```
<img width="456" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/7c9fd39e-e6af-4e89-b2ce-c86a57c9889f">

## 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
-- event_type = 3, page_id = 12



## 7. What are the top 3 pages by number of views?
- When event_type = 1 then its a page view. Group by page name and find the number of such events for each page.

```sql
SELECT P.page_name, COUNT(DISTINCT E.visit_id) AS number_of_views
FROM clique_bait.events E JOIN clique_bait.page_hierarchy P ON E.page_id = P.page_id
WHERE E.event_type = 1 
GROUP BY P.page_name
ORDER BY number_of_views DESC
LIMIT 3
```
<img width="429" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/e2b1d341-a6a0-4072-b87b-9042a32919ae">

## 8. What is the number of views and cart adds for each product category?
- When event_type = 1 then its a page view.
- When event_type = 2 then it is a cart add
- Group by product category and find the number of events which have page views and cart adds.

```sql
SELECT P.product_category, SUM(CASE WHEN E.event_type = 1 THEN 1 ELSE 0 END) AS page_views, SUM(CASE WHEN E.event_type = 2 THEN 1 ELSE 0 END) AS cart_add
FROM clique_bait.events E JOIN clique_bait.page_hierarchy P ON E.page_id = P.page_id
WHERE P.product_category IS NOT NULL
GROUP BY P.product_category
ORDER BY P.product_category
```

<img width="591" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/23a27080-dc1b-4525-ae15-c1ef34afa619">










