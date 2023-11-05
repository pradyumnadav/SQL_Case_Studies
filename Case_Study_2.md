# Data Cleaning Queries:
## 1. Set exclusions and extras = 'null' when they are blank :

```sql
UPDATE pizza_runner.customer_orders
SET exclusions = 'null'
WHERE exclusions = ''
```

```sql
UPDATE pizza_runner.customer_orders
SET extras = 'null'
WHERE extras = '' 
```
## 2. Set cancellation to be 'delivered' when it is either null or blank- meaning the orders were delivered
```sql
UPDATE pizza_runner.runner_orders
SET cancellation = 'delivered'
WHERE cancellation IS NULL or cancellation = ''
```

## 3. Clean up the distance column:
```sql
ALTER TABLE pizza_runner.runner_orders
ADD COLUMN delivery_distance NUMERIC(4,2)

UPDATE pizza_runner.runner_orders
SET delivery_distance =
    CASE
        WHEN distance LIKE '%km' THEN
            CAST(LEFT(distance,2) AS NUMERIC)
        WHEN distance = 'null' THEN
            0
        WHEN distance LIKE '__' THEN
            CAST(LEFT(distance,2) AS NUMERIC)
        WHEN distance LIKE '__._' THEN
            CAST(LEFT(distance,4) AS NUMERIC)
        ELSE
            delivery_distance  -- Leave it unchanged for other cases
    END
WHERE distance LIKE '%km' OR distance = 'null' OR distance NOT LIKE '%km';
```


## 4. Clean up duration column:
```sql
ALTER TABLE pizza_runner.runner_orders
ADD COLUMN duration_minutes NUMERIC(4,2);


UPDATE pizza_runner.runner_orders
SET duration_minutes = 
    CASE
      WHEN duration != 'null' THEN CAST(LEFT(duration,2) AS NUMERIC)
    ELSE 
      0
    END
WHERE duration != 'null' OR duration = 'null';
```

## 5. Clean up pickup_time column:
```sql

ALTER TABLE pizza_runner.runner_orders
ADD COLUMN pickup_dateTime TIMESTAMP;

UPDATE pizza_runner.runner_orders
SET pickup_time = '1970-01-01 00:00:00'
WHERE pickup_time = 'null'



UPDATE pizza_runner.runner_orders
SET pickup_dateTime = CAST(pickup_time AS TIMESTAMP)
WHERE pickup_time != 'null'
```



# Pizza Metrics- Data Analysis

## 1.How many pizzas were ordered?
```sql
SELECT COUNT(order_id) AS num_orders
FROM pizza_runner.customer_orders
```

<img width="279" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/a951826d-1462-4d02-b707-a391bc1b4a1b">

## 2. How many unique customer orders were made?

```sql
SELECT COUNT(DISTINCT customer_id) AS unique_cust_orders
FROM pizza_runner.customer_orders
```

<img width="352" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/39d6b8e5-5f1d-4c6a-9c67-3b6554aecfcc">


## 3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id, COUNT(order_id) AS orders_delivered
FROM pizza_runner.runner_orders
WHERE cancellation = 'delivered'
GROUP BY runner_id
ORDER BY runner_id
```

<img width="433" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/06d95d56-f54e-4dfe-9b12-c5c1c8d1f361">


## 4. How many of each type of pizza was delivered?
```sql
SELECT C.pizza_id, COUNT(R.order_id) AS pizzas_delivered
FROM pizza_runner.runner_orders R JOIN pizza_runner.customer_orders C 
ON R.order_id = C.order_id
WHERE R.cancellation = 'delivered'
GROUP BY C.pizza_id
```
<img width="428" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/53c9e14d-3442-47a4-853e-2c27be933685">

## 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT C.customer_id, P.pizza_name, COUNT(C.order_id) AS Pizzas_Ordered
FROM pizza_runner.customer_orders C JOIN pizza_runner.pizza_names P
ON C.pizza_id = P.pizza_id
GROUP BY C.customer_id, P.pizza_name
ORDER BY C.customer_id, Pizzas_Ordered DESC
```

<img width="537" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/8412da0f-2aff-402e-a4d5-9f464fdd192a">


## 6. What was the maximum number of pizzas delivered in a single order?
```sql
WITH CTE AS (
  SELECT order_id, COUNT(pizza_id) as num_pizzas
  FROM pizza_runner.customer_orders
  GROUP BY order_id
)  
SELECT order_id, MAX(num_pizzas) AS max_pizza
FROM CTE
GROUP BY order_id
ORDER BY max_pizza DESC
LIMIT 1
```
<img width="431" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/f76932e5-7592-4fb5-bfd1-a5e5bff46c37">

## 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT C.customer_id,
COUNT(CASE WHEN C.exclusions != 'null' OR C.extras != 'null' THEN 1 END) AS changed_orders,
COUNT(CASE WHEN C.exclusions = 'null' AND C.extras = 'null' THEN 1 END) AS unchanged_orders
FROM pizza_runner.customer_orders C JOIN pizza_runner.runner_orders R 
ON C.order_id = R.order_id
WHERE (R.cancellation = 'delivered')
GROUP BY C.customer_id
ORDER BY C.customer_id
```
<img width="501" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/753f949b-c44e-4473-bf35-c0409689e8d4">




## 8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT COUNT(CASE WHEN C.exclusions != 'null' AND C.extras != 'null' THEN 1 END) AS ExclusionsAndExtras_orders
FROM pizza_runner.customer_orders C JOIN pizza_runner.runner_orders R 
ON C.order_id = R.order_id
WHERE R.cancellation = 'delivered'
```

<img width="285" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/d7b0c021-496a-469a-aa91-3e941f753e76">

## 9. What was the volume of pizzas for each hour of the day?
```sql
SELECT EXTRACT(hour FROM order_time) AS hour_of_order, COUNT(order_id) AS count_orders
FROM pizza_runner.customer_orders
GROUP BY hour_of_order
ORDER BY hour_of_order
```
<img width="398" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/3c06cf85-71a2-4c38-8813-e0f97923a07b">

## 10. What was the volume of pizzas for each week?
```sql
SELECT FORMAT_DATETIME('%A', order_time) AS day_of_week, COUNT(order_id)
FROM pizza_runner.customer_orders
GROUP BY day_of_week
ORDER BY day_of_week
```
<img width="445" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/ba75f57e-73d0-4cbd-83d1-2390a6172ed9">

# 2. Runner and Customer Experience: Data Analysis

## 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT CAST((EXTRACT(DAY FROM registration_date) / 7) + 1 AS INT64) AS week_registration, COUNT(runner_id) AS registered_runners
FROM pizza_runner.runners
GROUP BY week_registration
ORDER BY registered_runners DESC
```
The query extracted day and converted it to week by dividing by 7 and adding 1

<img width="435" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/cc4a8915-c129-43ab-a1de-d2a4bbe97ce2">

## 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH CTE AS(
SELECT O.runner_id, EXTRACT(MINUTE FROM (O.pickup_dateTime - C.order_time)) AS runner_time
FROM pizza_runner.runners R JOIN pizza_runner.runner_orders O ON R.runner_id = O.runner_id JOIN pizza_runner.customer_orders C ON O.order_id = C.order_id
)
SELECT runner_id, AVG(runner_time) AS avg_runner_time_mins
FROM CTE
GROUP BY runner_id
ORDER BY avg_runner_time_mins
```
<img width="432" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/3dfb9027-de01-4cb3-a01a-d7752cde0d23">

## 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH CTE AS (SELECT C.order_id, COUNT(C.pizza_id) AS num_pizzas, EXTRACT(MINUTE FROM (O.pickup_dateTime - C.order_time)) AS prep_time
FROM pizza_runner.customer_orders C JOIN pizza_runner.runner_orders O ON C.order_id = O.order_id
WHERE O.cancellation = 'delivered'
GROUP BY C.order_id, O.pickup_dateTime, C.order_time
)
SELECT num_pizzas, AVG(prep_time) AS avg_prep_time
FROM CTE
GROUP BY num_pizzas
ORDER BY num_pizzas
```
<img width="373" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/e0f090a5-2f79-47b4-b2df-90c53bb6a155">

## 4. What was the average distance travelled for each customer?
```sql
SELECT C.customer_id, ROUND(AVG(O.delivery_distance),1) AS avg_distance_km
FROM pizza_runner.customer_orders C JOIN pizza_runner.runner_orders O ON C.order_id = O.order_id
GROUP BY C.customer_id
ORDER BY C.customer_id
```

<img width="422" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/e07b574f-e249-4fd8-b479-1afa47e82d39">


## 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT (MAX(duration_minutes) - MIN(duration_minutes)) AS diff_delivery
FROM pizza_runner.runner_orders 
```


<img width="258" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/55c9cee2-d71b-4404-9abb-754fa4cd11eb">

## 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT DISTINCT order_id,runner_id, ROUND(delivery_distance / (duration_minutes/60), 2) AS average_speed
FROM pizza_runner.runner_orders 
WHERE cancellation = 'delivered'
ORDER BY order_id
```

<img width="507" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/5e5fbb3b-51eb-4cdc-97a5-30386a431c8f">

## 7. What is the successful delivery percentage for each runner?
```sql
SELECT runner_id, (SUM(CASE WHEN cancellation = 'delivered' THEN 1 ELSE 0 END)/COUNT(order_id) * 100) AS successful_deliv_pct
FROM pizza_runner.runner_orders 
GROUP BY runner_id
ORDER BY runner_id
```


<img width="474" alt="image" src="https://github.com/pradyumnadav/SQL_Case_Studies/assets/132384475/322d2f07-18c8-41ac-97a8-050b0bd4b735">










