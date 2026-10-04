# Pizza Runner

## Cleaning the Data

```
UPDATE runner_orders
SET cancellation = NULL
WHERE cancellation = 'null';

UPDATE runner_orders
SET cancellation = NULL
WHERE cancellation = '';

UPDATE runner_orders
SET distance = NULL
WHERE distance = 'null';

UPDATE runner_orders
SET duration = NULL
WHERE duration = 'null';

ALTER TABLE runner_orders
RENAME COLUMN distance to distance_km;

ALTER TABLE runner_orders
ALTER COLUMN distance_km TYPE DOUBLE PRECISION
USING NULLIF(
  TRIM(REPLACE(distance_km, 'km', '')),
  ''
)::DOUBLE PRECISION;

ALTER TABLE runner_orders
RENAME COLUMN duration to duration_mins;

UPDATE runner_orders
SET duration_mins = CAST(TRIM(
  regexp_replace(duration_mins, '[\s]*(minute|min)[s]?', '', 'i')
) AS INT);

UPDATE customer_orders
SET exclusions = ''
WHERE exclusions = 'null';

UPDATE customer_orders
SET extras = ''
WHERE extras = 'null';
```

## A: Pizza Metrics

1. How many pizzas were ordered?

```
SELECT COUNT(*) AS pizza_orders_count FROM customer_orders;
```

| pizza_orders_count |
|:------------------:|
| 14                 |

2. How many unique customer orders were made?

```
SELECT 
COUNT(DISTINCT order_id) AS pizza_orders_count
FROM customer_orders;
```

| unique_orders |
|:-------------:|
| 10            |

3. How many successful orders were delivered by each runner?

```
SELECT
runner_id,
COUNT(order_id) AS successful_orders
FROM runner_orders
WHERE pickup_time IS NOT NULL
GROUP BY runner_id;
```

| runner_id | successful_orders |
|-----------|:-----------------:|
| 3         | 1                 |
| 2         | 3                 |
| 1         | 4                 |

4. How many of each type of pizza was delivered?

```
SELECT n.pizza_name, COUNT(ro.order_id) AS orders
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
JOIN pizza_names n ON co.pizza_id = n.pizza_id
GROUP BY n.pizza_name;
```

| pizza_name | orders |
|------------|:------:|
| Meatlovers | 10     |
| Vegetarian | 4      |

5. How many Vegetarian and Meatlovers were ordered by each customer?

```
SELECT 
  co.customer_id,
  n.pizza_name,
  COUNT(ro.order_id) AS orders
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
JOIN pizza_names n ON co.pizza_id = n.pizza_id
GROUP BY co.customer_id, n.pizza_name
ORDER BY co.customer_id;
```

| customer_id | pizza_name | orders |
|:-----------:|:----------:|:------:|
| 101         | Meatlovers | 2      |
| 101         | Vegetarian | 1      |
| 102         | Meatlovers | 2      |
| 102         | Vegetarian | 1      |
| 103         | Meatlovers | 3      |
| 103         | Vegetarian | 1      |
| 104         | Meatlovers | 3      |
| 105         | Vegetarian | 1      |

6. What was the maximum number of pizzas delivered in a single order?

```
WITH pizza_count AS (
  SELECT order_id, COUNT(order_id) AS number_of_pizzas
  FROM customer_orders
  GROUP BY order_id
  HAVING COUNT(order_id) > 1
)

SELECT MAX(number_of_pizzas) AS max_pizza_count
FROM pizza_count;
```

| max_pizza_count |
|:---------------:|
| 3               |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```
SELECT
  co.customer_id,
  SUM(
    CASE
      WHEN co.exclusions <> '' OR co.extras <> '' THEN 1
      ELSE 0
    END
  ) AS at_least_one_change,
  SUM(
    CASE
      WHEN co.exclusions = '' AND co.extras = '' THEN 1
      ELSE 0
    END
  ) AS no_change
FROM customer_orders co
JOIN runner_orders r ON co.order_id = r.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY co.customer_id
ORDER BY co.customer_id;
```

| customer_id | at_least_one_change | no_change |
|:-----------:|:-------------------:|:---------:|
| 101         | 0                   | 2         |
| 102         | 0                   | 2         |
| 103         | 3                   | 0         |
| 104         | 2                   | 1         |
| 105         | 1                   | 0         |

8. How many pizzas were delivered that had both exclusions and extras?

```
SELECT COUNT(co.order_id) AS pizza_count_with_both
FROM customer_orders co
JOIN runner_orders r ON co.order_id = r.order_id
WHERE r.pickup_time IS NOT NULL 
  AND co.exclusions <> '' AND co.extras <> '';
```

| pizza_count_with_both |
|:---------------------:|
| 1                     |

9. What was the total volume of pizzas ordered for each hour of the day?

```
SELECT
date_part('hour', order_time::timestamp) AS hour_of_day,
COUNT(order_id) AS pizzas_ordered
FROM customer_orders
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

| hour_of_day | pizzas_ordered |
|:-----------:|:--------------:|
| 11          | 1              |
| 13          | 3              |
| 18          | 3              |
| 19          | 1              |
| 21          | 3              |
| 23          | 3              |

10. What was the volume of orders for each day of the week?

```
SELECT
TO_CHAR(order_time::timestamp, 'Day') AS day_of_week,
COUNT(order_id) AS pizzas_ordered
FROM customer_orders
GROUP BY day_of_week
ORDER BY pizzas_ordered DESC;
```

| day_of_week | pizzas_ordered |
|:-----------:|:--------------:|
| Saturday    | 5              |
| Wednesday   | 5              |
| Thursday    | 3              |
| Friday      | 1              |

## B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```
SELECT
TO_CHAR(registration_date::timestamp, 'W') AS registration_week,
COUNT(runner_id) AS runners_signed_up
FROM runners
GROUP BY registration_week
ORDER BY registration_week;
```

| registration_week | runners_signed_up |
|:-----------------:|:-----------------:|
| 1                 | 2                 |
| 2                 | 1                 |
| 3                 | 1                 |

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```
SELECT
ro.runner_id,
ROUND(AVG(EXTRACT(EPOCH FROM (ro.pickup_time::timestamp - co.order_time::timestamp)) / 60)) AS time_to_pickup
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.cancellation IS NULL
GROUP BY ro.runner_id;
```

| runner_id | time_to_pickup |
|:---------:|:--------------:|
| 1         | 18             |
| 2         | 24             |
| 3         | 10             |

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

```
SELECT
co.order_id,
COUNT(co.order_id) AS number_of_pizzas,
ROUND(AVG(EXTRACT(EPOCH FROM (ro.pickup_time::timestamp - co.order_time::timestamp)) / 60)) AS time_to_prepare
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.cancellation IS NULL
GROUP BY co.order_id
ORDER BY time_to_prepare DESC;
```

| order_id | number_of_pizzas | time_to_prepare |
|:--------:|:----------------:|:---------------:|
| 4        | 3                | 29              |
| 3        | 2                | 21              |
| 8        | 1                | 20              |
| 10       | 2                | 16              |
| 7        | 1                | 10              |
| 5        | 1                | 10              |

Yes, there is a relationship. The orders with the most pizzas usually take the longest.

4. What was the average distance travelled for each customer?

```
SELECT
co.customer_id,
ROUND(AVG(ro.distance_km)) AS avg_distance
FROM runner_orders ro
JOIN customer_orders co ON ro.order_id = co.order_id
WHERE ro.cancellation IS NULL
GROUP BY co.customer_id;
```

| customer_id | avg_distance |
|:-----------:|:------------:|
| 101         | 20           |
| 102         | 17           |
| 105         | 25           |
| 104         | 10           |
| 103         | 23           |

5. What was the difference between the longest and shortest delivery times for all orders?

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

7. What is the successful delivery percentage for each runner?
