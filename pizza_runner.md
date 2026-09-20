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
COUNT(order_id) AS successful_orders
FROM runner_orders
WHERE pickup_time IS NOT NULL
AND pickup_time NOT LIKE 'null';
```

| successful_orders |
|:-----------------:|
| 8                 |

4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?
