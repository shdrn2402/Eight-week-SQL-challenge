![Project Logo](../images/case2_logo.png)

### Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Data Cleaning & Data Transformation](#data-cleaning--data-transformation)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)
  - [E. Bonus Questions](#e-bonus-questions)
  
### Introduction

> Did you know that over 115 million kilograms of pizza are consumed worldwide every day? (Well, at least according to Wikipedia!)
> 
> While scrolling through Instagram, Danny stumbled upon an intriguing trend: “80s Retro Styling and Pizza Is The Future!” Inspired by this idea, he realized that pizza alone wouldn’t be enough to attract seed funding for his ambitious vision of a Pizza Empire. That’s when Danny had his next big breakthrough – he decided to Uberize the experience. And so, Pizza Runner was born!
> 
> Danny began by recruiting a team of runners to deliver freshly made pizzas directly from Pizza Runner Headquarters (aka Danny’s house). To bring his idea to life, he maxed out his credit card to hire freelance developers to create a mobile app for taking customer orders.

### Entity Relationship Diagram
<details>
  <summary><em>show database schema</em></summary>

```SQL
CREATE SCHEMA pizza_runner;
SET search_path = pizza_runner;

DROP TABLE IF EXISTS runners;
CREATE TABLE runners (
  "runner_id" INTEGER,
  "registration_date" DATE
);
INSERT INTO runners
  ("runner_id", "registration_date")
VALUES
  (1, '2021-01-01'),
  (2, '2021-01-03'),
  (3, '2021-01-08'),
  (4, '2021-01-15');


DROP TABLE IF EXISTS customer_orders;
CREATE TABLE customer_orders (
  "order_id" INTEGER,
  "customer_id" INTEGER,
  "pizza_id" INTEGER,
  "exclusions" VARCHAR(4),
  "extras" VARCHAR(4),
  "order_time" TIMESTAMP
);

INSERT INTO customer_orders
  ("order_id", "customer_id", "pizza_id", "exclusions", "extras", "order_time")
VALUES
  ('1', '101', '1', '', '', '2020-01-01 18:05:02'),
  ('2', '101', '1', '', '', '2020-01-01 19:00:52'),
  ('3', '102', '1', '', '', '2020-01-02 23:51:23'),
  ('3', '102', '2', '', NULL, '2020-01-02 23:51:23'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '1', '4', '', '2020-01-04 13:23:46'),
  ('4', '103', '2', '4', '', '2020-01-04 13:23:46'),
  ('5', '104', '1', 'null', '1', '2020-01-08 21:00:29'),
  ('6', '101', '2', 'null', 'null', '2020-01-08 21:03:13'),
  ('7', '105', '2', 'null', '1', '2020-01-08 21:20:29'),
  ('8', '102', '1', 'null', 'null', '2020-01-09 23:54:33'),
  ('9', '103', '1', '4', '1, 5', '2020-01-10 11:22:59'),
  ('10', '104', '1', 'null', 'null', '2020-01-11 18:34:49'),
  ('10', '104', '1', '2, 6', '1, 4', '2020-01-11 18:34:49');


DROP TABLE IF EXISTS runner_orders;
CREATE TABLE runner_orders (
  "order_id" INTEGER,
  "runner_id" INTEGER,
  "pickup_time" VARCHAR(19),
  "distance" VARCHAR(7),
  "duration" VARCHAR(10),
  "cancellation" VARCHAR(23)
);

INSERT INTO runner_orders
  ("order_id", "runner_id", "pickup_time", "distance", "duration", "cancellation")
VALUES
  ('1', '1', '2020-01-01 18:15:34', '20km', '32 minutes', ''),
  ('2', '1', '2020-01-01 19:10:54', '20km', '27 minutes', ''),
  ('3', '1', '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
  ('4', '2', '2020-01-04 13:53:03', '23.4', '40', NULL),
  ('5', '3', '2020-01-08 21:10:57', '10', '15', NULL),
  ('6', '3', 'null', 'null', 'null', 'Restaurant Cancellation'),
  ('7', '2', '2020-01-08 21:30:45', '25km', '25mins', 'null'),
  ('8', '2', '2020-01-10 00:15:02', '23.4 km', '15 minute', 'null'),
  ('9', '2', 'null', 'null', 'null', 'Customer Cancellation'),
  ('10', '1', '2020-01-11 18:50:20', '10km', '10minutes', 'null');


DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" TEXT
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');


DROP TABLE IF EXISTS pizza_recipes;
CREATE TABLE pizza_recipes (
  "pizza_id" INTEGER,
  "toppings" TEXT
);
INSERT INTO pizza_recipes
  ("pizza_id", "toppings")
VALUES
  (1, '1, 2, 3, 4, 5, 6, 8, 10'),
  (2, '4, 6, 7, 9, 11, 12');


DROP TABLE IF EXISTS pizza_toppings;
CREATE TABLE pizza_toppings (
  "topping_id" INTEGER,
  "topping_name" TEXT
);
INSERT INTO pizza_toppings
  ("topping_id", "topping_name")
VALUES
  (1, 'Bacon'),
  (2, 'BBQ Sauce'),
  (3, 'Beef'),
  (4, 'Cheese'),
  (5, 'Chicken'),
  (6, 'Mushrooms'),
  (7, 'Onions'),
  (8, 'Pepperoni'),
  (9, 'Peppers'),
  (10, 'Salami'),
  (11, 'Tomatoes'),
  (12, 'Tomato Sauce');
```
</details>

![Project Logo](../images/case2_diagram.png)

### Data Cleaning & Data Transformation

#### A. Cleaning Missing Data in `customer_orders` Table

<details>
  <summary><em>show original table</em></summary>

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            | NaN    | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

</details>

*description:*

> As seen in the [database schema](#entity-relationship-diagram), the **`exclusions`** and **`extras`** columns in the **`customer_orders`** table contain missing values represented in various forms:
> - empty strings `('')`
> - the string `'null'`
> - the data type `NULL`
>
> These values indicate that no actions were taken by the customer: no ingredients were excluded, and no extras were added. To simplify data analysis and ensure consistency, all missing values will be standardized to the `NULL` data type. This approach will:
> - enable proper handling of missing data in SQL (e.g., using IS NULL)
> - reduce errors in data analysis and processing
> 
> Standardizing these values to `NULL` will improve the accuracy of future queries and ensure a clearer logic for data analysis.

*query:*
```SQL
UPDATE customer_orders
SET 
  exclusions = CASE 
    WHEN exclusions IS NULL OR exclusions = 'null' OR exclusions = '' THEN NULL
    ELSE exclusions
    END,
  extras = CASE 
    WHEN extras IS NULL OR extras = 'null' OR extras = '' THEN NULL
    ELSE extras
    END
;
```
<details>
  <summary><em>show table after data cleaning</em></summary>

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

</details>

#### B. Cleaning Missing Data in `runner_orders` Table

<details>
  <summary><em>show original table</em></summary>

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    | NaN                     |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         | NaN                     |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         | NaN                     |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

</details>

*description:*

> As seen in the [database schema](#entity-relationship-diagram), the `runner_orders` table contains inconsistencies in several columns, such as > `pickup_time`, `distance`, `duration`, and `cancellation`:
> 
> - **`pickup_time`**: Missing values are represented as `NULL`, the string `'null'`, or empty strings (`''`).
> - **`distance`**: Contains various formats for numeric values, such as `'20km'`, `'13.4km'`, or `'null'`.
> - **`duration`**: Similar to `distance`, it uses inconsistent formats, including `'32 minutes'`, `'20 mins'`, and `'null'`.
> - **`cancellation`**: Missing values are represented as `NULL`, the string `'null'`, or empty strings (`''`).
> 
> To standardize the data and prepare it for accurate analysis:
> 
> 1. **Missing Values**: All missing values in `pickup_time`, `distance`, `duration`, and `cancellation` will be converted to the `NULL` data type to enable consistent handling in SQL queries.
> 2. **Data Normalization**: 
>    - The `distance` and `duration` columns will have non-numeric characters removed to ensure only numeric values remain.
>    - The `pickup_time` column will be converted to the `TIMESTAMP` data type for accurate date and time analysis.
>    - The `duration` column will be converted to the `INTEGER` data type, representing the duration in minutes.
>    - The `distance` column will be converted to the `NUMERIC` data type to support precision in distance calculations.
> 
> These transformations will ensure data consistency, simplify future analysis, and reduce the risk of errors when running SQL queries.

*query:*
```SQL
UPDATE runner_orders
SET 
  distance = CASE 
    WHEN distance IS NULL OR distance = 'null' OR distance = '' THEN NULL
    ELSE REGEXP_REPLACE(distance, '[^0-9.]', '', 'g')
    END,
  duration = CASE 
    WHEN duration IS NULL OR duration = 'null' OR duration = '' THEN NULL
    ELSE REGEXP_REPLACE(duration, '[^0-9.]', '', 'g')
    END,
  pickup_time = CASE
    WHEN pickup_time IS NULL OR pickup_time = 'null' OR pickup_time = '' THEN NULL
    ELSE pickup_time
    END,
  cancellation = CASE 
    WHEN cancellation IS NULL OR cancellation = 'null' OR cancellation = '' THEN NULL
    ELSE cancellation
    END;

ALTER TABLE runner_orders
ALTER COLUMN distance TYPE NUMERIC USING distance::NUMERIC,
ALTER COLUMN duration TYPE INTEGER USING duration::INTEGER,
ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::TIMESTAMP;
```

<details>
  <summary><em>show table after data cleaning</em></summary>

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

</details>

### Case Study Questions & Solutions
#### A. Pizza Metrics
**1. How many pizzas were ordered?**

*query:*

```SQL
SELECT COUNT(*) AS pizzas_ordered
FROM customer_orders;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total number of pizzas ordered across all entries in the `customer_orders` table.

- The `COUNT(*)` function is used to count all rows in the `customer_orders` table, with each row representing an individual pizza order.
- The result is labeled as `pizzas_ordered` for clarity.

This query provides a simple and accurate total count of pizzas ordered, regardless of any additional details such as exclusions or extras.

</details>

*answer*

| pizzas_ordered |
| -------------- |
| 14             |

**2. How many unique customer orders were made?**

*query:*

```SQL
SELECT COUNT(DISTINCT(order_id)) as unique_pizza_orders
FROM customer_orders;
```
<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total number of unique customer orders made in the customer_orders table.

- The DISTINCT(order_id) function ensures that only unique order_id values are considered in the count, effectively ignoring duplicate entries

</details>

*answer*

| unique_pizza_orders |
| ------------------- |
| 10                  |

**3. How many successful orders were delivered by each runner?**

*query:*
```SQL
SELECT runner_id, COUNT(*) AS orders_delivered
FROM runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY orders_delivered DESC;
```
<details>
  <summary><em>show description</em></summary>

The SQL query retrieves the number of successful orders delivered by each runner.

- It selects the `runner_id` and calculates the count of successful deliveries (`orders_delivered`) using the `COUNT(*)` function.
- The `WHERE cancellation IS NULL` clause filters out any canceled orders, ensuring only completed deliveries are included in the results.
- The `GROUP BY runner_id` groups the data by each `runner_id` to calculate the total successful orders for each runner.
- The `ORDER BY orders_delivered DESC` clause sorts the results in descending order of the number of successful deliveries, with the most active runners appearing first.

</details>

| runner_id | orders_delivered |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

**4. How many of each type of pizza was delivered?**
  
*query:*
```SQL
SELECT pizza_name, COUNT(*) AS orders_delivered
FROM customer_orders C
JOIN runner_orders R ON C.order_id=R.order_id
JOIN pizza_names PN ON C.pizza_id=PN.pizza_id
WHERE cancellation IS NULL
GROUP BY pizza_name;
```
<details>
  <summary><em>show description</em></summary>

The SQL query retrieves the number of each type of pizza that was successfully delivered.

- It selects the `pizza_name` and calculates the count of successful deliveries (`orders_delivered`) using the `COUNT(*)` function.
- The `customer_orders` table (`C`) is joined with the `runner_orders` table (`R`) on the `order_id` column to link pizza orders with their delivery information.
- The `customer_orders` table (`C`) is also joined with the `pizza_names` table (`PN`) on the `pizza_id` column to map the pizza types to their names.
- The `WHERE cancellation IS NULL` clause filters out any canceled deliveries, ensuring only completed deliveries are included in the results.
- The `GROUP BY pizza_name` groups the results by each pizza type to calculate the total successful deliveries for each pizza.

</details>

*answer*

| pizza_name     | orders_delivered |
| -------------- | ---------------- |
| Meatlovers     | 9                |
| Vegetarian     | 4                |

**5. How many Vegetarian and Meatlovers were ordered by each customer?**
  
*query:*
```SQL
SELECT C.customer_id, PN.pizza_name, COUNT(*) AS orders_amount
FROM customer_orders C
JOIN pizza_names PN ON C.pizza_id=PN.pizza_id
GROUP BY C.customer_id, PN.pizza_name
ORDER BY C.customer_id
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates how many `Vegetarian` and `Meatlovers` pizzas were ordered by each customer.

- It retrieves data from the `customer_orders` table and joins it with the `pizza_names` table using the `pizza_id` column.
- The `GROUP BY` clause is used to group the results by `customer_id` and `pizza_name`, ensuring separate counts for each type of pizza for every customer.
- The `COUNT(*)` function calculates the total number of pizzas of each type ordered by each customer.
- The results are sorted by `customer_id` in ascending order for clarity.

This query correctly determines the number of orders for both `Vegetarian` and `Meatlovers` pizzas placed by each customer, providing a detailed breakdown.

</details>

*answer*

| customer_id | pizza_name | orders_amount |
| ----------- | ---------- | ------------- |
| 101         | Meatlovers | 2             |
| 101         | Vegetarian | 1             |
| 102         | Meatlovers | 2             |
| 102         | Vegetarian | 1             |
| 103         | Meatlovers | 3             |
| 103         | Vegetarian | 1             |
| 104         | Meatlovers | 3             |
| 105         | Vegetarian | 1             |

**6. What was the maximum number of pizzas delivered in a single order?**
  
*query:*

```SQL
SELECT MAX(order_count) AS max_pizzas_delivered
FROM (
  SELECT C.order_id, COUNT(*) AS order_count
  FROM customer_orders C
  JOIN runner_orders R ON C.order_id = R.order_id
  WHERE R.cancellation IS NULL
  GROUP BY C.order_id
) subquery;

```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the maximum number of pizzas delivered in a single order.

- A subquery is used to determine the total number of pizzas (`order_count`) associated with each `order_id` in the `customer_orders` table.
- The `JOIN` clause connects the `customer_orders` table with the `runner_orders` table on the `order_id` column to filter only valid, delivered orders. This is ensured by the condition `R.cancellation IS NULL`.
- The `COUNT(*)` function within the subquery counts the number of rows (pizzas) for each `order_id`.
- The `GROUP BY` clause groups the results by `order_id`, calculating the pizza count for each individual order.
- The outer query applies the `MAX()` function to identify the largest pizza count (`order_count`) among all grouped results.

This query efficiently identifies the single order with the highest number of pizzas delivered.

</details>

*answer*

| max_pizzas_delivered |
| -------------------- |
| 3                    |

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**
  
*query:*
```SQL
SELECT 
C.customer_id,
  SUM(CASE WHEN C.exclusions IS NOT NULL OR C.extras IS NOT NULL THEN 1 ELSE 0 END) AS pizzas_with_changes,
  SUM(CASE WHEN C.exclusions IS NULL AND C.extras IS NULL THEN 1 ELSE 0 END) AS pizzas_without_changes
FROM customer_orders C
JOIN runner_orders R ON C.order_id = R.order_id
WHERE R.cancellation IS NULL
GROUP BY C.customer_id
ORDER BY C.customer_id;
```
<details>
  <summary><em>show description</em></summary>

The SQL query calculates, for each customer, the number of delivered pizzas with at least one change (exclusions or extras) and those with no changes, accounting for data that has already been cleaned.

- The `JOIN` clause connects the `customer_orders` table (`C`) with the `runner_orders` table (`R`) using the `order_id` column to include only valid deliveries, determined by the condition `R.cancellation IS NULL`.
- Two `SUM(CASE ... END)` expressions are used:
  - The first expression checks for pizzas with changes:
    - A pizza is considered "with changes" if either the `exclusions` or `extras` column is not `NULL`.
    - A value of `1` is added to the sum for such pizzas; otherwise, `0` is added.
  - The second expression checks for pizzas without changes:
    - A pizza is considered "without changes" if both the `exclusions` and `extras` columns are `NULL`.
    - A value of `1` is added to the sum for such pizzas; otherwise, `0` is added.
- The results are grouped by `customer_id` to calculate totals for each customer.
- The `ORDER BY C.customer_id` clause sorts the results in ascending order of `customer_id`.

This query leverages the cleaned data to accurately determine the count of pizzas with and without changes for each customer.

</details>

*answer*

| customer_id | pizzas_with_changes | pizzas_without_changes |
| ----------- | ------------------- | ---------------------- |
| 101         | 0                   | 2                      |
| 102         | 0                   | 3                      |
| 103         | 3                   | 0                      |
| 104         | 2                   | 1                      |
| 105         | 1                   | 0                      |

**8. How many pizzas were delivered that had both exclusions and extras?**
  
*query:*
```SQL
SELECT 
  COUNT(*) AS pizzas_with_both_changes
FROM customer_orders C
JOIN runner_orders R ON C.order_id = R.order_id
WHERE R.cancellation IS NULL
  AND C.exclusions IS NOT NULL
  AND C.extras IS NOT NULL
```
<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total number of pizzas that were delivered with both exclusions and extras.

- The `JOIN` clause connects the `customer_orders` table (`C`) with the `runner_orders` table (`R`) using the `order_id` column to include only valid deliveries.
- The `WHERE` clause includes three conditions:
  - `R.cancellation IS NULL` ensures only successfully delivered pizzas are considered.
  - `C.exclusions IS NOT NULL` checks that exclusions were specified for the pizza.
  - `C.extras IS NOT NULL` checks that extras were specified for the pizza.
- The `COUNT(*)` function counts the total number of rows that satisfy all the conditions.

This query provides the count of pizzas that had both exclusions and extras and were successfully delivered.

</details>

| pizzas_with_both_changes |
| ------------------------ |
| 1                        |

**9. What was the total volume of pizzas ordered for each hour of the day?**
  
*query:*
```SQL
SELECT 
  EXTRACT(HOUR FROM order_time) AS order_hour,
  COUNT(*) AS total_pizzas_ordered
FROM customer_orders
GROUP BY order_hour
ORDER BY order_hour;
```
<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total number of pizzas ordered for each hour of the day.

- The `EXTRACT(HOUR FROM order_time)` function extracts the hour from the `order_time` column, grouping orders by the hour they were placed.
- The `COUNT(*)` function counts the total number of pizzas ordered within each hour.
- The `GROUP BY order_hour` groups the results by each unique hour value to calculate the totals for each hour.
- The `ORDER BY order_hour` sorts the results in ascending order of the hour for clear representation.

This query provides the total number of pizzas ordered for each hour of the day based on the `order_time` column.

</details>

*answer*

| order_hour | total_pizzas_ordered |
| ---------- | -------------------- |
| 11         | 1                    |
| 13         | 3                    |
| 18         | 3                    |
| 19         | 1                    |
| 21         | 3                    |
| 23         | 3                    |

**10. What was the volume of orders for each day of the week?**
  
*query:*
```SQL
SELECT TO_CHAR(order_time, 'Day')AS order_day, COUNT(*) AS total_orders
FROM customer_orders
GROUP BY order_day
ORDER BY total_orders DESC;
```
<details>
  <summary><em>show description</em></summary>

The SQL query retrieves the total number of pizza orders (`total_orders`) for each day of the week (`order_day`) and sorts the results in descending order of the total orders.

- The `TO_CHAR(order_time, 'Day')` function extracts the day of the week from the `order_time` column, formatting it with the first letter capitalized and adding spaces for alignment.
- The `COUNT(*)` function calculates the total number of orders for each day of the week.
- The `GROUP BY order_day` clause groups the results by the day of the week (`order_day`), ensuring that the count of orders is aggregated for each unique day.
- The `ORDER BY total_orders DESC` clause sorts the results by the `total_orders` column in descending order, showing the days with the highest number of orders first.

This query provides insights into which days of the week had the most pizza orders while maintaining a readable day format.

</details>

*answer*

| order_day | total_orders |
| --------- | ------------ |
| Friday    | 1            |
| Thursday  | 3            |
| Saturday  | 5            |
| Wednesday | 5            |

#### B. Runner and Customer Experience
**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

*query:*

```SQL
SELECT 
  DATE_TRUNC('week', registration_date) AS week_start,
  COUNT(runner_id) AS runners_signed_up
FROM runners
GROUP BY week_start
ORDER BY week_start;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the number of runners who signed up during each one-week period starting from `2021-01-01`.

- The `DATE_TRUNC('week', registration_date)` function truncates the `registration_date` to the beginning of the week, aligning all dates within the same week to the same starting date.
- The `COUNT(*)` function counts the number of runners who signed up within each week.
- The `GROUP BY` clause groups the results by the truncated `registration_date`, effectively grouping by week.
- The `ORDER BY` clause ensures the results are sorted by the starting date of each week in ascending order.

This query helps identify how many runners signed up during each week, providing insight into recruitment trends over time.

</details>

| week_start             | runners_signed_up |
| ---------------------- | ----------------- |
| 2020-12-28 00:00:00+00 | 2                 |
| 2021-01-04 00:00:00+00 | 1                 |
| 2021-01-11 00:00:00+00 | 1                 |

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

*query:*

```SQL
SELECT 
    runner_id,
    ROUND(AVG(EXTRACT(EPOCH FROM (pickup_time - order_time)) / 60)::NUMERIC, 2) AS avg_pickup_time_minutes
FROM runner_orders
JOIN customer_orders USING(order_id)
WHERE pickup_time IS NOT NULL
GROUP BY runner_id
ORDER BY avg_pickup_time_minutes;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the average time (in minutes) it took for each runner to arrive at the Pizza Runner HQ to pick up orders, rounded to two decimal places.

- **`EXTRACT(EPOCH FROM (pickup_time - order_time))`**: This expression calculates the difference between the `pickup_time` and `order_time` in seconds by extracting the epoch time (total seconds).
- **`/ 60`**: Converts the time difference from seconds to minutes.
- **`AVG(...)`**: Computes the average time in minutes for each runner.
- **`ROUND(..., 2)`**: Rounds the resulting average to two decimal places for clarity and precision.
- The query joins the `runner_orders` table with the `customer_orders` table using the `order_id` field to match orders with their pickup times.
- The `WHERE` clause filters out rows where `pickup_time` is NULL.
- **`GROUP BY runner_id`**: Groups the results by `runner_id` to calculate the average pickup time for each runner.
- **`ORDER BY avg_pickup_time_minutes`**: Sorts the results in ascending order based on the average pickup time in minutes.

This query ensures accurate computation of the average pickup times for runners, providing insight into the efficiency of each runner.

</details>

*answer*

| runner_id | avg_pickup_time_minutes |
| --------- | ----------------------- |
| 3         | 10.47                   |
| 1         | 15.68                   |
| 2         | 23.72                   |

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

*query:*

```SQL
WITH pizza_count_per_order AS (
    SELECT 
        order_id,
        COUNT(pizza_id) AS pizzas_ordered
    FROM customer_orders
    GROUP BY order_id
)
SELECT 
    pco.pizzas_ordered,
    ROUND(AVG(EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60)::NUMERIC, 2) AS avg_preparation_time_minutes
FROM runner_orders ro
JOIN customer_orders co USING(order_id)
JOIN pizza_count_per_order pco ON co.order_id = pco.order_id
WHERE ro.pickup_time IS NOT NULL
GROUP BY pco.pizzas_ordered
ORDER BY pco.pizzas_ordered;
```

<details>
  <summary><em>show description</em></summary>

The SQL query determines if there is a relationship between the number of pizzas in an order (`pizzas_ordered`) and the average preparation time (`avg_preparation_time_minutes`) required for the order.

- A Common Table Expression (CTE) named `pizza_count_per_order` is used to calculate the total number of pizzas (`pizzas_ordered`) for each order (`order_id`).
  - The `COUNT(pizza_id)` function counts the number of pizzas in each order, and the results are grouped by `order_id`.
- The main query joins the `runner_orders` table with the `customer_orders` table using the `order_id` column. It also joins with the CTE `pizza_count_per_order` to include the number of pizzas for each order.
- The `WHERE` clause ensures that only orders with a valid `pickup_time` (non-NULL) are considered.
- The `AVG(EXTRACT(EPOCH FROM (ro.pickup_time - co.order_time)) / 60)` function calculates the average preparation time in minutes for each group of orders with the same number of pizzas.
- The `ROUND` function rounds the calculated average preparation time to two decimal places for clarity.
- The query groups the results by `pizzas_ordered` to calculate the average preparation time for each group.
- The results are ordered by `pizzas_ordered` in ascending order to display the relationship clearly.

This query provides insight into whether the number of pizzas in an order affects the preparation time.

</details>

*answer*

| pizzas_ordered | avg_preparation_time_minutes |
| -------------- | ---------------------------- |
| 1              | 12.36                        |
| 2              | 18.38                        |
| 3              | 29.28                        |

**4. What was the average distance travelled for each customer?**

*query:*

```SQL
SELECT 
    customer_id,
    ROUND(AVG(distance), 2) AS avg_distance_km
FROM runner_orders
JOIN customer_orders USING(order_id)
WHERE distance IS NOT NULL
GROUP BY customer_id
ORDER BY avg_distance_km;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the average distance in kilometers (`avg_distance_km`) traveled for each customer (`customer_id`) and orders the results by customer ID.

- The `JOIN` clause combines the `runner_orders` table with the `customer_orders` table using the `order_id` column to link corresponding orders.
- The `WHERE distance IS NOT NULL` clause ensures that only rows with valid distance values are included in the calculation.
- The `AVG(distance)` function calculates the average distance traveled for each customer.
- The `ROUND(..., 2)` function rounds the average distance to two decimal places for better readability.
- The `GROUP BY customer_id` clause groups the results by customer ID, ensuring that the average is calculated individually for each customer.
- The `ORDER BY customer_id` clause sorts the results by customer ID in ascending order for organized output.

</details>

*answer*

| customer_id | avg_distance_km |
| ----------- | --------------- |
| 104         | 10.00           |
| 102         | 16.73           |
| 101         | 20.00           |
| 103         | 23.40           |
| 105         | 25.00           |

**5. What was the difference between the longest and shortest delivery times for all orders?**

*query:*

```SQL
SELECT 
    MAX(duration) - MIN(duration) AS max_delivery_time_difference
FROM runner_orders;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the difference between the longest (`MAX(duration)`) and shortest (`MIN(duration)`) delivery times for all orders in the `runner_orders` table.

- The `MAX(duration)` function retrieves the longest delivery time from the `duration` column.
- The `MIN(duration)` function retrieves the shortest delivery time from the same column.
- The query subtracts the shortest delivery time from the longest to calculate the difference.
- The result is returned as a single value in the column `max_delivery_time_difference`.

This query provides a straightforward measure of the range of delivery times, helping to understand the variability in order deliveries.

</details>

*answer*

| max_delivery_time_difference |
| ---------------------------- |
| 30                           |

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

*query:*

```SQL
SELECT 
  runner_id, 
  distance, 
  duration, 
  ROUND((distance / duration) * 60, 2) AS speed_km_per_hr
FROM runner_orders
WHERE duration IS NOT NULL AND distance IS NOT NULL
ORDER BY speed_km_per_hr DESC;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the average speed (`speed_km_per_hr`) for each delivery made by runners, using the distance traveled and the delivery duration.

- **`ROUND((distance / duration) * 60, 2) AS speed_km_per_hr`**:
  - This formula calculates the speed in kilometers per hour by dividing the `distance` by `duration` (in minutes), multiplying the result by 60 to convert to hours, and rounding to two decimal places for clarity.
- **`WHERE duration IS NOT NULL AND distance IS NOT NULL`**:
  - Ensures only records with valid `distance` and `duration` values are included in the calculation.
- **`ORDER BY speed_km_per_hr DESC`**:
  - Sorts the results in descending order of speed, highlighting the fastest deliveries first.

This query provides insights into the speed of each delivery for all runners and helps identify trends or anomalies in the data.

Upon reviewing the results, the following trends and observations can be noted:

- **Anomalous Speed**:
   - The highest speed recorded (`93.60 km/h`) appears unrealistic for a pizza delivery scenario. This suggests potential data issues, such as incorrect distance or duration values for this delivery.

- **General Trend**:
   - As the delivery distance increases beyond a certain threshold (e.g., 20-25 km), the calculated average speed tends to decrease. This could be due to longer travel times for greater distances or potential traffic delays.

- **Task Interpretation**:
   - The question asks for the "average speed for each runner for each delivery."
   Since each delivery involves a single distance and duration, the "average speed" for a single delivery is equivalent to the calculated speed for that delivery. 
   Therefore, our query calculates the speed for each delivery individually, which aligns with the task.

</details>

*answer*

| runner_id | distance | duration | speed_km_per_hr |
| --------- | -------- | -------- | --------------- |
| 2         | 23.4     | 15       | 93.60           |
| 2         | 25       | 25       | 60.00           |
| 1         | 10       | 10       | 60.00           |
| 1         | 20       | 27       | 44.44           |
| 1         | 13.4     | 20       | 40.20           |
| 3         | 10       | 15       | 40.00           |
| 1         | 20       | 32       | 37.50           |
| 2         | 23.4     | 40       | 35.10           |

**7. What is the successful delivery percentage for each runner?**

*query:*

```SQL
SELECT 
    runner_id,
    ROUND(
        (COUNT(*) FILTER (WHERE cancellation IS NULL) / COUNT(*)::NUMERIC) * 100, 2
    ) AS successful_delivery_percentage
FROM runner_orders
GROUP BY runner_id
ORDER BY runner_id;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the percentage of successful deliveries for each runner.

- **`COUNT(*)`**: Counts the total number of orders for each runner.
- **`COUNT(*) FILTER (WHERE cancellation IS NULL)`**: Counts only the successful deliveries where `cancellation` is `NULL`.
- **Division and Multiplication**: The ratio of successful deliveries to total deliveries is multiplied by 100 to get a percentage.
- **`ROUND(..., 2)`**: Rounds the result to two decimal places for better readability.
- **`GROUP BY runner_id`**: Groups the results by each runner (`runner_id`) to calculate the success rate individually.
- **`ORDER BY runner_id`**: Ensures the output is sorted by runner ID in ascending order.

This query provides an overview of how many deliveries each runner completed successfully as a percentage of their total assigned deliveries.

</details>

*answer*

| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 1         | 100.00                         |
| 2         | 75.00                          |
| 3         | 50.00                          |

#### C. Ingredient Optimisation
**1. What are the standard ingredients for each pizza?**

*query:*

```SQL
SELECT
  subquery.pizza_name,
  string_agg(PT.topping_name, ', ') AS toppings
FROM
  (SELECT
     PN.pizza_name,
     string_to_table(toppings, ', ')::INT topping_id
   FROM pizza_recipes
   JOIN pizza_names PN USING(pizza_id)
  ) subquery
JOIN pizza_toppings PT USING (topping_id)
GROUP BY pizza_name;
```

<details>
  <summary><em>show description</em></summary>

The SQL query retrieves the standard ingredients for each type of pizza by combining information from the `pizza_recipes`, `pizza_names`, and `pizza_toppings` tables.

- **Subquery**:
  - Selects the `pizza_name` from `pizza_names` and splits the `toppings` column in `pizza_recipes` into individual `topping_id` values using the `string_to_table` function.
  - The result creates rows for each topping associated with a specific pizza.
  
- **Join**:
  - Joins the subquery with the `pizza_toppings` table using the `topping_id` to retrieve the topping names corresponding to each `topping_id`.

- **Grouping and Aggregation**:
  - Groups the results by `pizza_name`.
  - Uses `string_agg` to combine all the topping names into a single comma-separated string for each pizza.

This query effectively lists the standard ingredients for each pizza type in a readable format, ensuring clarity and consistency.

</details>

*answer*

| pizza_name | toppings                                                              |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

**2. What was the most commonly added extra?**

*query:*

```SQL
WITH toppings_counts AS (
  SELECT
    PT.topping_name,
    COUNT(*) AS was_added_times
  FROM
    (SELECT
      string_to_table(extras, ', ')::INT AS topping_id
    FROM customer_orders
    ) subquery
  JOIN pizza_toppings PT USING(topping_id)
  GROUP BY PT.topping_name
)

SELECT
  topping_name,
  was_added_times
FROM toppings_counts
WHERE was_added_times = (SELECT MAX(was_added_times) FROM toppings_counts);
```

<details>
  <summary><em>show description</em></summary>

The SQL query identifies the most frequently added topping as an extra by leveraging a Common Table Expression (CTE) to calculate counts for each topping and then filtering for the maximum count.

- **CTE (`toppings_counts`)**:
  - Extracts toppings from the `extras` column using `string_to_table` and converts them to integers (`topping_id`).
  - Joins the resulting `topping_id` values with the `pizza_toppings` table to retrieve topping names.
  - Groups the results by `topping_name` and calculates the count of occurrences (`was_added_times`) for each topping.

- **Main Query**:
  - Selects rows from the `toppings_counts` CTE where the count (`was_added_times`) matches the maximum value.
  - Uses a subquery with `MAX(was_added_times)` to determine the highest count among all toppings.
  - Ensures all toppings tied for the maximum count are included.

This query provides the name(s) of the topping(s) most frequently added as extras, including ties if multiple toppings share the maximum count.

</details>

*answer*

| topping_name | was_added_times |
| ------------ | --------------- |
| Bacon        | 4               |

**3. What was the most common exclusion?**

*query:*

```SQL
WITH exclusions_counts AS (
    SELECT
        PT.topping_name,
        COUNT(*) AS was_excluded_times
    FROM
        (SELECT
            string_to_table(exclusions, ', ')::INT AS topping_id
         FROM customer_orders
        ) subquery
    JOIN pizza_toppings PT USING(topping_id)
    GROUP BY PT.topping_name
)
SELECT
    topping_name,
    was_excluded_times
FROM exclusions_counts
WHERE was_excluded_times = (SELECT MAX(was_excluded_times) FROM exclusions_counts);
```

<details>
  <summary><em>show description</em></summary>

The SQL query determines the most frequently excluded topping by counting exclusions for each topping and identifying the one(s) with the highest count.

- **CTE (`exclusions_counts`)**:
  - Extracts toppings from the `exclusions` column using `string_to_table` and converts them to integers (`topping_id`).
  - Joins the extracted `topping_id` values with the `pizza_toppings` table to retrieve topping names.
  - Groups the results by `topping_name` and calculates the number of times each topping was excluded (`was_excluded_times`).

- **Main Query**:
  - Selects rows from the `exclusions_counts` CTE where the exclusion count (`was_excluded_times`) matches the maximum value.
  - Uses a subquery with `MAX(was_excluded_times)` to find the highest exclusion count.
  - Includes all toppings tied for the maximum count, if applicable.

This query provides the name(s) of the topping(s) that were excluded the most frequently, ensuring ties are captured when multiple toppings share the maximum exclusion count.

</details>

*answer*

| topping_name | was_excluded_times |
| ------------ | ------------------ |
| Cheese       | 4                  |

**4. Generate an order item for each record in the customers_orders table in the format of one of the following:**
  - Meat Lovers
  - Meat Lovers - Exclude Beef
  - Meat Lovers - Extra Bacon
  - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

*query:*

```SQL
SELECT
	ROW_NUMBER() OVER(ORDER BY order_time) AS id,
    order_id,
    pizza_name ||
    CASE
        WHEN exclusions IS NOT NULL THEN ' - Exclude ' || (
            SELECT string_agg(topping_name, ', ')
            FROM pizza_toppings
            WHERE topping_id = ANY(string_to_array(exclusions, ',')::INT[])
        )
        ELSE ''
    END ||
    CASE
        WHEN extras IS NOT NULL THEN ' - Extra ' || (
            SELECT string_agg(topping_name, ', ')
            FROM pizza_toppings
            WHERE topping_id = ANY(string_to_array(extras, ',')::INT[])
        )
        ELSE ''
    END AS order_items
FROM customer_orders
JOIN pizza_names USING (pizza_id)
;
```

<details>
  <summary><em>show description</em></summary>

The SQL query generates a unique order item (`order_items`) for each record in the `customer_orders` table, formatted to include the pizza name and any exclusions or extras applied to the order. Additionally, the query assigns a unique row identifier (`id`) to each result, providing sequential numbering for the output.

- `ROW_NUMBER() OVER()` assigns a unique sequential number (`id`) to each row in the output, independent of the original `order_id`. This ensures that every record has a unique identifier.
- `pizza_name` is concatenated with:
  - **Exclusions**: If the `exclusions` column is not `NULL`, a subquery retrieves the ingredient names associated with the exclusions (from the `pizza_toppings` table) and concatenates them into a comma-separated list.
  - **Extras**: If the `extras` column is not `NULL`, a similar subquery retrieves the ingredient names for extras and concatenates them into a list.
- `CASE` statements ensure that exclusions and extras are only included in the formatted output if they exist.
- `||` is used to concatenate strings, adding text such as " - Exclude " and " - Extra " to describe exclusions and extras.
- The `JOIN` with `pizza_names` links each `pizza_id` to its corresponding pizza name.

This query outputs a formatted `order_items` string for each record in the `customer_orders` table while ensuring every row in the result has a unique identifier (`id`).

</details>

*answer*

| id  | order_id | order_items                                                     |
| --- | -------- | --------------------------------------------------------------- |
| 1   | 1        | Meatlovers                                                      |
| 2   | 2        | Meatlovers                                                      |
| 3   | 3        | Meatlovers                                                      |
| 4   | 3        | Vegetarian                                                      |
| 5   | 4        | Vegetarian - Exclude Cheese                                     |
| 6   | 4        | Meatlovers - Exclude Cheese                                     |
| 7   | 4        | Meatlovers - Exclude Cheese                                     |
| 8   | 5        | Meatlovers - Extra Bacon                                        |
| 9   | 6        | Vegetarian                                                      |
| 10  | 7        | Vegetarian - Extra Bacon                                        |
| 11  | 8        | Meatlovers                                                      |
| 12  | 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 13  | 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |
| 14  | 10       | Meatlovers                                                      |


**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients**
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

*query:*

```SQL
SELECT
    ROW_NUMBER() OVER(ORDER BY order_time) AS id,
    order_id,
    pizza_name || ': ' ||
    (
      SELECT
      string_agg(
        CASE
          WHEN topping_id = ANY(string_to_array(extras, ',')::INT[])
          THEN '2x' || topping_name
          ELSE topping_name
        END, ', ' ORDER BY topping_name
        )
      FROM pizza_toppings
      WHERE topping_id = ANY(
        array(
          SELECT unnest(string_to_array(pr.toppings, ',')::INT[]) 
          EXCEPT 
          SELECT unnest(string_to_array(exclusions, ',')::INT[])
          )
          ) OR topping_id = ANY(string_to_array(extras, ',')::INT[])
    ) AS order_items
FROM customer_orders
JOIN pizza_names USING (pizza_id)
JOIN pizza_recipes pr USING (pizza_id)
ORDER BY id;
```

<details>
  <summary><em>show description</em></summary>

The SQL query generates an ordered list of ingredients for each pizza order, handling standard toppings, exclusions, and extras:

- `ROW_NUMBER() OVER(ORDER BY order_time):` Assigns a unique ID to each row based on the order time for proper ordering.
- `pizza_name || ': ' || ...:` Concatenates the pizza name with the formatted list of toppings.
- `string_agg:` Aggregates ingredients into a comma-separated list:
  - `CASE statement:` Adds "2x" before the ingredient name for extras.
  - `ORDER BY topping_name:` Ensures alphabetical ordering of ingredients.
- `WHERE clause:` Combines logic for standard toppings, exclusions, and extras:
  - `EXCEPT:` Removes excluded ingredients from the standard toppings.
  - `OR:` Includes extras in the final list.
- `ORDER BY id:` Sorts the final result based on the unique ID assigned earlier.

This query provides a detailed and well-structured list of ingredients for each order, incorporating exclusions and extras accurately.
</details>

*answer*

| id  | order_id | order_items                                                                         |
| --- | -------- | ----------------------------------------------------------------------------------- |
| 1   | 1        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 2   | 2        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 3   | 3        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 4   | 3        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 5   | 4        | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                      |
| 6   | 4        | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 7   | 4        | Meatlovers: BBQ Sauce, Bacon, Beef, Chicken, Mushrooms, Pepperoni, Salami           |
| 8   | 5        | Meatlovers: BBQ Sauce, 2xBacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 9   | 6        | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes              |
| 10  | 7        | Vegetarian: 2xBacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes     |
| 11  | 8        | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |
| 12  | 9        | Meatlovers: BBQ Sauce, 2xBacon, Beef, 2xChicken, Mushrooms, Pepperoni, Salami       |
| 13  | 10       | Meatlovers: 2xBacon, Beef, 2xCheese, Chicken, Pepperoni, Salami                     |
| 14  | 10       | Meatlovers: BBQ Sauce, Bacon, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami   |

**6.What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

*query:*

```SQL
WITH delivered AS (
  SELECT 
    CO.order_id, 
    CO.pizza_id, 
    CO.exclusions, 
    CO.extras, 
    PR.toppings
  FROM runner_orders RO
  JOIN customer_orders CO USING(order_id)
  JOIN pizza_recipes PR USING(pizza_id)
  WHERE RO.cancellation IS NULL
),
final_toppings_calculated AS (
  SELECT 
    order_id,
    pizza_id,
    exclusions,
    extras,
    toppings,
    ARRAY(
      SELECT UNNEST(string_to_array(toppings, ',')::INT[])
      EXCEPT SELECT UNNEST(string_to_array(exclusions, ',')::INT[])
      UNION ALL SELECT UNNEST(string_to_array(extras, ',')::INT[])
      ) AS final_toppings
  FROM delivered
),
ingredient_totals AS (
  SELECT 
    UNNEST(final_toppings) AS topping_id
  FROM final_toppings_calculated
)

SELECT 
  pt.topping_name,
  COUNT(*) AS total_count
FROM ingredient_totals it
JOIN pizza_toppings pt ON it.topping_id = pt.topping_id
GROUP BY pt.topping_name
ORDER BY total_count DESC, pt.topping_name;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total quantity of each ingredient used in all delivered pizzas, sorted by most frequent first.

- **Delivered Pizzas**:
  - Filters only delivered pizzas (`cancellation IS NULL`) and extracts the relevant details (`order_id`, `pizza_id`, `exclusions`, `extras`) along with the standard recipe (`toppings`) from related tables.

- **Final Toppings Calculation**:
  - Computes the final list of toppings for each order:
    - Removes excluded toppings using `EXCEPT`.
    - Adds extra toppings using `UNION ALL`.
    - Ensures the resulting list is sorted to maintain consistency.

- **Ingredient Aggregation**:
  - Converts the list of toppings into individual ingredient IDs using `UNNEST`.
  - Joins with the `pizza_toppings` table to map `topping_id` to `topping_name`.
  - Groups the results by `topping_name` to calculate the total count of each ingredient.
  - Sorts the output by the total count of ingredients in descending order and alphabetically by name.

This query provides a precise count of each topping used in all delivered pizzas, fully accounting for any exclusions and additions.

</details>

*answer*

| topping_name | total_count |
| ------------ | ----------- |
| Bacon        | 12          |
| Mushrooms    | 11          |
| Cheese       | 10          |
| Beef         | 9           |
| Chicken      | 9           |
| Pepperoni    | 9           |
| Salami       | 9           |
| BBQ Sauce    | 8           |
| Onions       | 3           |
| Peppers      | 3           |
| Tomato Sauce | 3           |
| Tomatoes     | 3           |

#### D. Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

*query:*

```SQL
SELECT
  PN.pizza_name,
  CASE
  WHEN pizza_id=1
    THEN total_sold*12
    ELSE total_sold*10
  END AS total_earned_usd
FROM
	(SELECT
    CO.pizza_id,
    COUNT(*) AS total_sold
	FROM runner_orders RO
	JOIN customer_orders CO USING(order_id)
	WHERE cancellation IS NULL
	GROUP BY pizza_id) SQ
JOIN pizza_names PN USING(pizza_id)
ORDER BY total_earned_usd DESC;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total revenue generated from each type of pizza.

- **Inner Query (`SQ`)**:
  - Counts the total number of each type of pizza sold (`total_sold`) by grouping rows based on `pizza_id`.
  - Excludes orders with cancellations by filtering with `WHERE cancellation IS NULL`.
  - Joins `runner_orders` and `customer_orders` using the common column `order_id`.

- **Outer Query**:
  - Joins the result of the inner query with the `pizza_names` table to retrieve the names of the pizzas.
  - Uses a `CASE` statement to calculate the revenue (`total_earned_usd`) based on the price of each type of pizza:
    - Meat Lovers (`pizza_id = 1`) are priced at $12.
    - Vegetarian (`pizza_id = 2`) are priced at $10.

- **Sorting**:
  - The result is ordered in descending order by `total_earned_usd`, showing the pizza type with the highest revenue first.

This query provides insights into which type of pizza generates the most revenue.

</details>

*answer*

| pizza_name | total_earned_usd |
| ---------- | ---------------- |
| Meatlovers | 108              |
| Vegetarian | 30               |

**2.**

*query:*

```SQL

```

<details>
  <summary><em>show description</em></summary>



</details>

*answer*

**3.**

*query:*

```SQL

```

<details>
  <summary><em>show description</em></summary>



</details>

*answer*


#### E. Bonus Questions
  ...