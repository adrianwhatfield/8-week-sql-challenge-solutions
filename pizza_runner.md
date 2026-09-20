# Pizza Runner

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
AND pickup_time NOT LIKE 'null'
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
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?
