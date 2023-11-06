# Case Study 2: Foodie-Fi

<b> Here is the official introduction and problem statement for this case study:
>Introduction
>Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!
>
>Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

>Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.
>
>Available Data
>Danny has shared the data design for Foodie-Fi and also short descriptions on each of the database tables - our case study focuses on only 2 tables but there will be a challenge to create a new table for the Foodie-Fi team.
>
>All datasets exist within the foodie_fi database schema - be sure to include this reference within your SQL scripts as you start exploring the data and answering the case study questions.

Here is the ERD:
<img width="710" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/f58be2fb-b455-4b6c-aeea-816b6a609834">

Here are the questions and my solutions:




## 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM foodie_fi.subscriptions
```
<img width="271" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/200ed8cd-87fd-44f7-a7cd-8dd86b85f315">

Foodie-fi has had 1000 customers so far.


## 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?

The EXTRACT() function can be used to find the month from the subscription start_date column. The month can be used to find the number of subscribers who signed up to trial plans.

```sql
SELECT EXTRACT(MONTH FROM S.start_date) AS subscription_month, COUNT(DISTINCT S.customer_id) AS num_subscribers
FROM foodie_fi.subscriptions S JOIN foodie_fi.plans P ON S.plan_id = P.plan_id
WHERE P.plan_id = 0
GROUP BY subscription_month
ORDER BY num_subscribers DESC
```

<img width="436" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/77ae6078-4161-43fd-84a9-d4a10585f409">

## 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?

As per my understanding, the question wants us to show the breakdown of the number of customers who signed up to each plan after 2020, with the plans being- trial, basic monthly, pro monthly, pro annual and churn (the customer has left the company).

- Filter the data based on start date after 2020. Then for each plan name, find the number of events.


```sql
SELECT P.plan_name, COUNT(*) AS count_events
FROM foodie_fi.subscriptions S JOIN foodie_fi.plans P ON S.plan_id = P.plan_id
WHERE EXTRACT(YEAR FROM S.start_date) >= 2021
GROUP BY P.plan_name
ORDER BY P.plan_name
```

<img width="498" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/88da7e45-8657-45d3-992e-42a8f8ac7c0c">

There were no customers signing up to the trial plan after 2020.

## 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
- Use conditional aggregation through CASE statements to find the number of customers who churned, by assigning 1 when plan_id = 4 (which is churn), then sum up all the ones. This is the number of customers who churned- churn_count.
- Use churn_count and divide by the distinct count of customers to get the percentage of customers who churned.


```sql
SELECT SUM(CASE WHEN S.plan_id = 4 THEN 1 ELSE 0 END) AS churn_count, ROUND((SUM(CASE WHEN S.plan_id = 4 THEN 1 ELSE 0 END)/ COUNT(DISTINCT S.customer_id)) * 100, 1) AS churn_percentage
FROM foodie_fi.plans P JOIN foodie_fi.subscriptions S ON
P.plan_id = S.plan_id
```

<img width="343" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/49832cce-8ea8-4e5d-b790-7007e4c7d143">

30.7% of Foodie fi customers churned.

## 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
This is an interesting question. Let's breakdown my solution in steps:
- First, use a CTE named customer_total_counts to find the number of customers i.e total_customers.
- Second, use the churn_customer_counts CTE to:
  - Make a self join of the subscriptions table with customer id, with additional conditions such that the tables are joined <b> only when the customer is initially on a trial plan (plan id = 0), and then churns (plan id = 4) within 7 days </b> . Therefore, this table will only contain those customers who churned straight after the expiry of the trial period, with the Foodie fi trial period lasting for 7 days.
  - Count the number of customers. This is the churned_after_Trial column and it contains the number of customers who churned straight after the free trial.
-  Use the data from the two CTEs and divide the number of customers who churned after tirial by the total customers, to get percentage of customers who churned after the trial period.

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

9.2% of the customers churned after their trial period expired.

## 6. What is the number and percentage of customer plans after their initial free trial?
This questions asks us to break down the plans that customers are signing up for, after their trial periods expire. 
- Create a post_trial_customer_count CTE to create a table such that we have information about customers who signed up for a free trial initially and then moved to a different plan or churned after the free trial expired (in 7 days). Self join is utilised again to create this data. Once the data is prepared through self-joins, count the number of customers in each plan. <b> This customer_count column will indicate what plans customers are adopting after their trial periods are ending. </b>
- Create total_customer_count CTE to find the total number of customers.
- Use the two data from the two CTEs to find the percentage breakdown of customers in each plan after their trial periods end.


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

54% of the customers are adopting a basic monthly plan after their trial ends. Only 3% customers are adopting a pro annual plan, Foodie fi should take measures to drive more customers into this plan.

## 7. How many customers have upgraded to an annual plan in 2020?

The questions asks us to find the number of customers who upgraded to an annual plan in 2020. 
- For each customer, find their initial plan. The LEAD() window function is used, after partioning the data by customer id and ordering by order date, to find the next plan id that each customer signed up for. Therefore, for each customer, we have data about their current plan id and next plan id. Filter this data by the year of 2020.
- Use the CTE to count the number of customers who were initially on another plan (initial_plan != 3) and then moved to the annual plan (next_plan = 3). The solution is perhaps overkill, but I wanted to account for the fact that customers can technically sign up to the annual plan directly. The question asked us to calculate the number of customers who <b> UPGRADED </b> to an annual plan!

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

195 customers upgraded to an annual plan in 2020.

## 8. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

Again, the self join technique is utilised to find customers who were initially on a trial plan (S.plan_id = 0) and then moved to the annual plan (T.plan_id = 3). The FIRST_VALUE() window function is used to find the date of joining of the customer and is saved as initial_plan_date. Then the difference between initial plan date and pro plan dates are calculated in days and an overall average of is taken to find the average amount of days it takes customers to upgrade to a pro plan.

```sql
WITH CTE AS(
  SELECT S.customer_id, FIRST_VALUE(S.start_date) OVER(PARTITION BY S.customer_id ORDER BY S.start_date ASC) AS initial_plan_date, T.start_date AS pro_plan_date
  FROM foodie_fi.subscriptions S JOIN foodie_fi.subscriptions T ON S.customer_id = T.customer_id AND S.plan_id = 0 AND T.plan_id = 3
)
SELECT ROUND(AVG(EXTRACT(DAY FROM (pro_plan_date - initial_plan_date))), 0) AS avg_days_upgrade_to_pro
FROM CTE
```
<img width="274" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/03a77f84-cae9-4bbb-9337-f5560fee5d70">

On average, it takes Foodie Fi customers 105 days to upgrade to a pro plan, calculated from the date they join Foodie Fi on a trial.

## 9. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
LEAD() function is utilised again to find the next plan that each customer went on to subscribe to, having partitioned the data by customer id and ordered by start date, this is saved as next_plan.

Then, it is simple to count the number of customers who were initially on a pro monthly plan (initial_plan = 2) and then downgraded to a basic monthly plan (next_plan = 1)

```sql
WITH CTE AS(
SELECT customer_id, plan_id AS initial_plan, LEAD(plan_id, 1) OVER(PARTITION BY customer_id ORDER BY start_date ASC) AS next_plan
FROM foodie_fi.subscriptions
WHERE EXTRACT(YEAR FROM start_date) = 2020
)
SELECT COUNT(DISTINCT customer_id) AS count_downgrade
FROM CTE
WHERE initial_plan = 2 AND next_plan = 1
```
<img width="256" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/bfc92b73-28b0-458a-ae0c-7aa8397ecfa8">

There were no customers who made a downgrade from pro monthly to basic monthly plan in 2020.
















