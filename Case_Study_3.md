# 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM foodie_fi.subscriptions
```

# 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?
```sql
SELECT EXTRACT(MONTH FROM S.start_date) AS subscription_month, COUNT(DISTINCT S.customer_id) AS num_subscribers
FROM foodie_fi.subscriptions S JOIN foodie_fi.plans P ON S.plan_id = P.plan_id
WHERE P.plan_id = 0
GROUP BY subscription_month
ORDER BY num_subscribers DESC
```

<img width="436" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/77ae6078-4161-43fd-84a9-d4a10585f409">

# 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?
```sql
SELECT P.plan_name, COUNT(*) AS count_events
FROM foodie_fi.subscriptions S JOIN foodie_fi.plans P ON S.plan_id = P.plan_id
WHERE EXTRACT(YEAR FROM S.start_date) >= 2021
GROUP BY P.plan_name
ORDER BY P.plan_name
```

<img width="498" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/88da7e45-8657-45d3-992e-42a8f8ac7c0c">

#4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT SUM(CASE WHEN S.plan_id = 4 THEN 1 ELSE 0 END) AS churn_count, ROUND((SUM(CASE WHEN S.plan_id = 4 THEN 1 ELSE 0 END)/ COUNT(DISTINCT S.customer_id)) * 100, 1) AS churn_percentage
FROM foodie_fi.plans P JOIN foodie_fi.subscriptions S ON
P.plan_id = S.plan_id
```

<img width="343" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/49832cce-8ea8-4e5d-b790-7007e4c7d143">

#5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH customer_total_counts AS(
  SELECT COUNT(DISTINCT customer_id) AS total_customers
  FROM foodie_fi.subscriptions
),
churn_customer_counts AS(
SELECT COUNT(DISTINCT S.customer_id) AS churned_after_Trial
FROM foodie_fi.subscriptions S JOIN foodie_fi.subscriptions T ON S.customer_id = T.customer_id AND 
S.plan_id = 0 AND T.plan_id = 4
WHERE EXTRACT(DAY FROM (T.start_date - S.start_date)) = 7)
SELECT churned_after_Trial, ROUND((churned_after_Trial/total_customers) * 100,1) AS churn_after_trial_pct
FROM customer_total_counts, churn_customer_counts;

```

<img width="464" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/136bc9f6-e8f7-401e-bba9-3aace7d7d488">

#6. What is the number and percentage of customer plans after their initial free trial?
```sql
WITH post_trial_customer_count AS(
SELECT T.plan_id, P.plan_name, COUNT(DISTINCT T.customer_id) AS customer_count
FROM foodie_fi.subscriptions S JOIN foodie_fi.subscriptions T ON S.customer_id = T.customer_id
JOIN foodie_fi.plans P ON T.plan_id = P.plan_id
AND S.plan_id = 0 AND T.plan_id IN (1,2,3,4) AND EXTRACT(DAY FROM (T.start_date - S.start_date)) = 7
GROUP BY T.plan_id, P.plan_name
ORDER BY T.plan_id ASC
),
total_customer_count AS(
  SELECT COUNT(DISTINCT customer_id) AS total_customer_count
  FROM foodie_fi.subscriptions
)
SELECT plan_id, plan_name,customer_count,(customer_count/ CAST(total_customer_count AS NUMERIC)) * 100 AS percentage, 
FROM post_trial_customer_count P, total_customer_count T
```
<img width="682" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/cda680ee-4675-400e-80f0-3636a7df1005">

# 7. How many customers have upgraded to an annual plan in 2020?
```sql
WITH CTE AS(
SELECT customer_id, plan_id AS initial_plan, LEAD(plan_id, 1) OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS next_plan
FROM foodie_fi.subscriptions
WHERE EXTRACT(YEAR FROM start_date) = 2020
)
SELECT COUNT(DISTINCT customer_id) AS count_Annualupgrade
FROM CTE
WHERE initial_plan != 3 and next_plan = 3
```
<img width="352" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/0ad0467a-224a-47b6-b494-5eda2406cb7b">

# 8. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH CTE AS(
  SELECT S.customer_id, FIRST_VALUE(S.start_date) OVER(PARTITION BY S.customer_id ORDER BY S.start_date ASC) AS initial_plan_date, T.start_date AS pro_plan_date
  FROM foodie_fi.subscriptions S JOIN foodie_fi.subscriptions T ON S.customer_id = T.customer_id AND S.plan_id = 0 AND T.plan_id = 3
)
SELECT ROUND(AVG(EXTRACT(DAY FROM (pro_plan_date - initial_plan_date))), 0) AS avg_days_upgrade_to_pro
FROM CTE
```
<img width="274" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/03a77f84-cae9-4bbb-9337-f5560fee5d70">

# 9. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH CTE AS(
SELECT customer_id, plan_id AS initial_plan, LEAD(plan_id, 1) OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS next_plan
FROM foodie_fi.subscriptions
)
SELECT COUNT(DISTINCT customer_id) AS count_downgrade
FROM CTE
WHERE initial_plan = 2 AND next_plan = 1
```
<img width="256" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/bfc92b73-28b0-458a-ae0c-7aa8397ecfa8">


















