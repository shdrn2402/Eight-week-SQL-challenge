![Project Logo](../images/case2_logo.png)

## Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Data Cleaning & Data Transformation](#data-cleaning--data-transformation)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)
  - [E. Bonus Questions](#e-bonus-questions)
  
## Introduction

> Did you know that over 115 million kilograms of pizza are consumed worldwide every day? (Well, at least according to Wikipedia!)
> 
> While scrolling through Instagram, Danny stumbled upon an intriguing trend: “80s Retro Styling and Pizza Is The Future!” Inspired by this idea, he realized that pizza alone wouldn’t be enough to attract seed funding for his ambitious vision of a Pizza Empire. That’s when Danny had his next big breakthrough – he decided to Uberize the experience. And so, Pizza Runner was born!
> 
> Danny began by recruiting a team of runners to deliver freshly made pizzas directly from Pizza Runner Headquarters (aka Danny’s house). To bring his idea to life, he maxed out his credit card to hire freelance developers to create a mobile app for taking customer orders.

## Entity Relationship Diagram
<details>
  <summary><em><strong>show database schema</strong></em></summary>

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

## Data Cleaning & Data Transformation

### A. Cleaning Missing Data in `customer_orders` Table

<details>
  <summary><em><strong>show original table</strong></em></summary>

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

***description:***

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

***query:***
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
  <summary><em><strong>show table after data cleaning</strong></em></summary>

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

### B. Cleaning Missing Data in `runner_orders` Table

<details>
  <summary><em><strong>show original table</strong></em></summary>

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

***description:***

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

***query:***
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
  <summary><em><strong>show table after data cleaning</strong></em></summary>

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

## Case Study Questions & Solutions
### A. Pizza Metrics
#### 1. How many pizzas were ordered?

***query:***

```SQL
SELECT COUNT(*) AS pizzas_ordered
FROM customer_orders;
```

<details>
  <summary><em><strong>show description</strong></em></summary>

The SQL query calculates the total number of pizzas ordered across all entries in the `customer_orders` table.

- The `COUNT(*)` function is used to count all rows in the `customer_orders` table, with each row representing an individual pizza order.
- The result is labeled as `pizzas_ordered` for clarity.

This query provides a simple and accurate total count of pizzas ordered, regardless of any additional details such as exclusions or extras.

</details>

<details>
<summary><em><strong>show answer</strong></em></summary>

| pizzas_ordered |
| -------------- |
| 14             |

</details>

#### 2. How many unique customer orders were made?

***query:***

```SQL
SELECT COUNT(DISTINCT(order_id)) as unique_pizza_orders
FROM customer_orders;
```
<details>
  <summary><em><strong>show description</strong></em></summary>

The SQL query calculates the total number of unique customer orders made in the customer_orders table.

- The DISTINCT(order_id) function ensures that only unique order_id values are considered in the count, effectively ignoring duplicate entries

</details>

<details>
<summary><em><strong>show answer</strong></em></summary>

| unique_pizza_orders |
| ------------------- |
| 10                  |

</details>

#### 3. How many successful orders were delivered by each runner?

***query:***
```SQL
SELECT runner_id, COUNT(*) AS orders_delivered
FROM runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY orders_delivered DESC;
```
<details>
  <summary><em><strong>show description</strong></em></summary>

The SQL query retrieves the number of successful orders delivered by each runner.

- It selects the `runner_id` and calculates the count of successful deliveries (`orders_delivered`) using the `COUNT(*)` function.
- The `WHERE cancellation IS NULL` clause filters out any canceled orders, ensuring only completed deliveries are included in the results.
- The `GROUP BY runner_id` groups the data by each `runner_id` to calculate the total successful orders for each runner.
- The `ORDER BY orders_delivered DESC` clause sorts the results in descending order of the number of successful deliveries, with the most active runners appearing first.

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

| runner_id | orders_delivered |
| --------- | ---------------- |
| 1         | 4                |
| 2         | 3                |
| 3         | 1                |

</details>

#### 4. How many of each type of pizza was delivered?
  
***query:***
```SQL
SELECT pizza_name, COUNT(*) AS orders_delivered
FROM customer_orders C
JOIN runner_orders R ON C.order_id=R.order_id
JOIN pizza_names PN ON C.pizza_id=PN.pizza_id
WHERE cancellation IS NULL
GROUP BY pizza_name;
```
<details>
  <summary><em><strong>show description</strong></em></summary>

The SQL query retrieves the number of each type of pizza that was successfully delivered.

- It selects the `pizza_name` and calculates the count of successful deliveries (`orders_delivered`) using the `COUNT(*)` function.
- The `customer_orders` table (`C`) is joined with the `runner_orders` table (`R`) on the `order_id` column to link pizza orders with their delivery information.
- The `customer_orders` table (`C`) is also joined with the `pizza_names` table (`PN`) on the `pizza_id` column to map the pizza types to their names.
- The `WHERE cancellation IS NULL` clause filters out any canceled deliveries, ensuring only completed deliveries are included in the results.
- The `GROUP BY pizza_name` groups the results by each pizza type to calculate the total successful deliveries for each pizza.

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

| pizza_name     | orders_delivered |
| -------------- | ---------------- |
| Meatlovers     | 9                |
| Vegetarian     | 4                |


</details>

#### 5. How many Vegetarian and Meatlovers were ordered by each customer?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>

#### 6. What was the maximum number of pizzas delivered in a single order?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>

#### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>

#### 8. How many pizzas were delivered that had both exclusions and extras?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>

#### 9. What was the total volume of pizzas ordered for each hour of the day?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>

#### 10. What was the volume of orders for each day of the week?
  
***query:***
```SQL

```
<details>
  <summary><em><strong>show description</strong></em></summary>

</details>


<details>
<summary><em><strong>show answer</strong></em></summary>

</details>


### B. Runner and Customer Experience
  ...
### C. Ingredient Optimisation
  ...
### D. Pricing and Ratings
  ...
### E. Bonus Questions
  ...