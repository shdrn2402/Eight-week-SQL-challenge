![Project Logo](project_images/logo.png)

# Contents:
- [Introduction](#introduction)
- [Problem Statement](#problem-statment) 
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
- [Bonus Questions & Solutions](#bonus-questions--solutions)
- [Key Insights](#key-insights)
  
## Introduction
Danny is a big fan of Japanese cuisine, and at the start of 2021, he took a bold step by opening a small, charming restaurant specializing in his three favorite dishes: sushi, curry, and ramen.

Now, Danny’s Diner needs your help to keep the business thriving. The restaurant has collected some basic data from its first months of operation but lacks the expertise to analyze and utilize this information effectively for better decision-making.

## Problem Statement
Danny wants to analyze his customer data to understand their visiting habits, spending patterns, and favorite menu items. These insights will help him improve customer experience and decide whether to expand the loyalty program.

He also needs simple, pre-generated datasets for his team to inspect without using SQL. Due to privacy concerns, Danny has provided a sample of customer data, which includes three key datasets:

- sales
- menu
- members
## Entity Relationship Diagram
<details>
  <summary><strong>show database schema</strong></summary>

```SQL
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```
</details>

![Project Logo](project_images/entity_relationship_diagram.png)

## Case Study Questions & Solutions
### 1. What is the total amount each customer spent at the restaurant?

***query:***
```SQL
SELECT s.customer_id, SUM(m.price) AS total_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_spent DESC;
```

***description:***

The SQL query retrieves the `customer_id` and calculates the `total amount spent` (total_spent) by each customer at the restaurant.

- It joins the sales table (s) and the menu table (m) using the `product_id` column, which serves as a common key between the two tables.
- The query then groups the results by `customer_id`, aggregating the `total price` (SUM(m.price)) for each customer.
- The SUM() function calculates the total amount spent by each customer across all their purchases.
- Finally, the results are sorted in descending order (DESC) based on the total_spent value, so customers who spent the most appear at the top.

***answer:***
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

### 2. How many days has each customer visited the restaurant?

***query:***
```SQL
SELECT customer_id, COUNT(DISTINCT order_date) AS visits_amount
FROM sales
GROUP BY customer_id
ORDER BY visits_amount ASC;
```

***description:***

The SQL query retrieves the `customer_id` and calculates the total number of unique visit dates (visits_amount) for each customer at the restaurant.

- It uses the `COUNT(DISTINCT order_date)` function to count the number of unique dates each customer visited the restaurant, ensuring duplicate dates are not counted.
- The query groups the results by customer_id, so the count of unique dates is calculated for each individual customer.
- The results are then sorted in ascending order (ASC) based on `visits_amount`, so customers with the fewest visits appear first.

***answer:***
| customer_id | visits_amount |
| ----------- | ------------- |
| C           | 2             |
| A           | 4             |
| B           | 6             |

3. **What was the first item from the menu purchased by each customer?**

***query:***
```SQL
WITH first_purchases AS (
  SELECT 
    customer_id,
    MIN(order_date) AS first_order_date
  FROM sales
  GROUP BY customer_id
)
SELECT 
  fp.customer_id,
  m.product_name
FROM first_purchases fp
JOIN sales s 
  ON fp.customer_id = s.customer_id 
  AND fp.first_order_date = s.order_date
JOIN menu m 
  ON s.product_id = m.product_id
ORDER BY fp.customer_id;
```

***description:***

The SQL query retrieves the `customer_id` and identifies the first menu item (product_name) purchased by each customer based on their earliest order date.

- It uses a Common Table Expression (CTE) named `first_purchases` to calculate the first order date (first_order_date) for each customer.
- Within the CTE, the `MIN(order_date)` function is applied to find the earliest purchase date for each customer_id, and the results are grouped by customer to ensure accuracy.
- The main query joins the result of the CTE (first_purchases) with the `sales` table on both `customer_id` and `order_date` to retrieve all rows corresponding to the first purchase date for each customer.
- It then joins the sales table with the menu table on `product_id` to extract the product name (product_name) of the items purchased on that date.
- The final result is sorted by `customer_id` to provide an organized output.

This query focuses on identifying items purchased on the first recorded date for each customer. It assumes that all purchases on the same date are part of a single order and doesn't account for the sequence of purchases within the day, as the dataset lacks time information.

While the `DENSE_RANK()` function could also be used to identify the first purchase, this approach was chosen for its simplicity and clarity, given the dataset's structure.

***answer:***
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

4. **What is the most purchased item on the menu and how many times was it purchased by all customers?**

***query:***
```SQL
WITH product_counts AS (
  SELECT 
    m.product_name, 
    COUNT(*) AS total_purchases
  FROM sales s
  JOIN menu m
    ON m.product_id = s.product_id
  GROUP BY m.product_name
)
SELECT 
  product_name as most_popular, 
  total_purchases
FROM product_counts
WHERE total_purchases = (SELECT MAX(total_purchases) FROM product_counts);
```

***description:***

The SQL query retrieves the most popular menu item(s) based on the total number of purchases (total_purchases).

- It uses a Common Table Expression (CTE) named `product_counts` to calculate the total number of purchases for each product. The `COUNT(*)` function counts all purchases, while the results are grouped by product_name using the GROUP BY clause.
- The sales table is joined with the menu table on `product_id` to link sales records with their corresponding product names.
- The main query selects the `product_name` (renamed as most_popular) and `total_purchases` from the CTE.
- It includes a subquery to find the maximum value of total_purchases using the MAX() function. Only products with a purchase count equal to this maximum value are returned.
- This ensures that all products with the highest number of purchases are included, even if multiple products share the same maximum value.

The final result displays the name of the most popular product(s) and their total purchase count, accurately reflecting the most frequently ordered items.

***answer:***
| most_popular | total_purchases |
| ------------ | --------------- |
| ramen        | 8               |

5. **Which item was the most popular for each customer?**

***query:***
```SQL
WITH sold_per_customer AS (
  SELECT
    customer_id,
    product_id,
    COUNT(product_id) AS items_sold,
    DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS rank_value
  FROM sales
  GROUP BY customer_id, product_id
)
SELECT customer_id, m.product_name, sold_per_customer.items_sold
FROM sold_per_customer
JOIN menu m ON sold_per_customer.product_id = m.product_id
WHERE rank_value = 1
ORDER BY customer_id;
```

***description:***

The SQL query uses a Common Table Expression (CTE) named `sold_per_customer` to generate a temporary result set that calculates the popularity of items purchased by each customer.

- Within the CTE, it selects the `customer_id`, `product_id`, and calculates the number of times each product was purchased by the customer (items_sold) using the `COUNT(product_id)` function.
- The results are grouped by `customer_id` and `product_id` using the GROUP BY clause, ensuring that the purchase count is calculated for each product of every customer.
- The `DENSE_RANK()` function assigns a rank to each row within the partition of each `customer_id`, ordered by the purchase count (items_sold) in descending order. The most popular product(s) for each customer receive a rank of 1.
- Next, the main query retrieves the `customer_id`, the `purchase count` (items_sold), and the product name (product_name) by joining the CTE with the menu table on the product_id field.
It filters the results to include only the rows where the rank (rank_value) is equal to 1, which corresponds to the most purchased product(s) for each customer.
- Finally, the results are sorted by customer_id, returning each customer's ID, their most popular product, and the number of times it was purchased. This query accounts for cases where multiple products are tied as the most popular for a single customer by including all such products in the result.

The JOIN between the sold_per_customer CTE and the menu table is performed in the main query rather than inside the CTE. This approach ensures that the grouping, ranking, and filtering are applied to a smaller subset of data (only the sales table), which can improve performance when the menu table is large. By deferring the JOIN to the main query, the data processing is streamlined, focusing on only the necessary rows with rank_value = 1.

***answer:***
| customer_id | product_name | items_sold |
| ----------- | ------------ | ---------- |
| A           | ramen        | 3          |
| B           | sushi        | 2          |
| B           | curry        | 2          |
| B           | ramen        | 2          |
| C           | ramen        | 3          |

6. **Which item was purchased first by the customer after they became a member?**

***query:***
```SQL
WITH first_purchase_date AS (
  SELECT
    s.customer_id,
    MIN(s.order_date) AS first_order_date
  FROM sales s
  JOIN members mmbr ON s.customer_id = mmbr.customer_id
  WHERE s.order_date >= mmbr.join_date
  GROUP BY s.customer_id
)
SELECT
    s.customer_id,
    m.product_name,
    s.order_date
FROM sales s
JOIN first_purchase_date fpd ON s.customer_id = fpd.customer_id AND s.order_date = fpd.first_order_date
JOIN menu m ON s.product_id = m.product_id
ORDER BY s.customer_id, s.order_date;
```

***description:***

The SQL query retrieves the first product purchased by each customer after they joined the loyalty program.

- It uses a Common Table Expression (CTE) named `first_purchase_date` to calculate the first available order date (`first_order_date`) for each customer after their membership date.
- Within the CTE, the `MIN(s.order_date)` function is used to identify the earliest order date for each customer, and the results are grouped by `customer_id` to ensure the calculation is accurate for each individual.
- The main query joins the `sales` table with the CTE on `customer_id` and `first_order_date`, selecting only the orders that match the earliest date for each customer.
- The query also joins the `sales` table with the `menu` table on `product_id`, enabling the retrieval of the product name (`product_name`) for the identified orders.
- The `WHERE` clause in the CTE filters the results to include only orders made on or after the customer's `join_date`, ensuring that purchases prior to membership are excluded.
- The `ORDER BY s.customer_id, s.order_date` clause ensures the results are sorted by customer and by the date of the order for clear organization.

This query efficiently determines the first product purchased by each customer after joining, including all products purchased on the same first date.


***answer:***
| customer_id | product_name | order_date |
| ----------- | ------------ | ---------- |
| A           | curry        | 2021-01-07 |
| B           | sushi        | 2021-01-11 |

7. **Which item was purchased just before the customer became a member?**

***query:***
```SQL
WITH last_purchase_before_membership AS (
  SELECT
    s.customer_id,
    MAX(s.order_date) AS last_order_date
  FROM sales s
  JOIN members mmbr ON s.customer_id = mmbr.customer_id
  WHERE s.order_date < mmbr.join_date
  GROUP BY s.customer_id
)
SELECT
    s.customer_id,
    m.product_name,
    s.order_date
FROM sales s
JOIN last_purchase_before_membership lpbm ON s.customer_id = lpbm.customer_id AND s.order_date = lpbm.last_order_date
JOIN menu m ON s.product_id = m.product_id
ORDER BY s.customer_id, s.order_date;
```

***description:***

The SQL query retrieves the last product purchased by each customer before they joined the loyalty program.

- It uses a Common Table Expression (CTE) named `last_purchase_before_membership` to calculate the last available order date (`last_order_date`) for each customer before their membership date.
- Within the CTE, the `MAX(s.order_date)` function is used to identify the most recent order date for each customer before their membership, and the results are grouped by `customer_id` to ensure the calculation is specific to each individual.
- The main query joins the `sales` table with the CTE on `customer_id` and `last_order_date`, selecting only the orders that match the last date for each customer.
- The query also joins the `sales` table with the `menu` table on `product_id`, enabling the retrieval of the product name (`product_name`) for the identified orders.
- The `WHERE` clause in the CTE filters the results to include only orders made before the customer's `join_date`, ensuring that purchases on or after membership are excluded.
- The `ORDER BY s.customer_id, s.order_date` clause ensures the results are sorted by customer and by the date of the order for clear organization.

This query efficiently determines the last product purchased by each customer before joining, including all products purchased on the same last date.

***answer:***
| customer_id | product_name | order_date |
| ----------- | ------------ | ---------- |
| A           | sushi        | 2021-01-01 |
| A           | curry        | 2021-01-01 |
| B           | sushi        | 2021-01-04 |

8. **What is the total items and amount spent for each member before they became a member?**

***query:***

***description:***

***answer:***

9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

***query:***

***description:***

***answer:***

10. **In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

***query:***

***description:***

***answer:***

## Bonus Questions & Solutions
...
## Key Insights