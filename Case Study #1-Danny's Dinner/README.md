![Project Logo](project_images/logo.png)

## Contents:
- [Introduction](#introduction)
- [Problem Statement](#problem-statment) 
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
- [Bonus Questions & Solutions](#bonus-questions--solutions)
- [Key Insights](#key-insights)
  
### Introduction
Danny is a big fan of Japanese cuisine, and at the start of 2021, he took a bold step by opening a small, charming restaurant specializing in his three favorite dishes: sushi, curry, and ramen.

Now, Danny’s Diner needs your help to keep the business thriving. The restaurant has collected some basic data from its first months of operation but lacks the expertise to analyze and utilize this information effectively for better decision-making.

### Problem Statement
Danny wants to analyze his customer data to understand their visiting habits, spending patterns, and favorite menu items. These insights will help him improve customer experience and decide whether to expand the loyalty program.

He also needs simple, pre-generated datasets for his team to inspect without using SQL. Due to privacy concerns, Danny has provided a sample of customer data, which includes three key datasets:

- sales
- menu
- members
### Entity Relationship Diagram
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

### Case Study Questions & Solutions
1. **What is the total amount each customer spent at the restaurant?**

***query:***
```SQL
SELECT s.customer_id, SUM(m.price) AS total_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_spent DESC;
```

***description:***

The SQL query retrieves the customer_id and calculates the total amount spent (total_spent) by each customer at the restaurant.

- It joins the sales table (s) and the menu table (m) using the product_id column, which serves as a common key between the two tables.
- The query then groups the results by customer_id, aggregating the total price (SUM(m.price)) for each customer.
- The SUM() function calculates the total amount spent by each customer across all their purchases.
- Finally, the results are sorted in descending order (DESC) based on the total_spent value, so customers who spent the most appear at the top.

***answer:***
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

2. **How many days has each customer visited the restaurant?**

***query:***
```SQL
SELECT customer_id, COUNT(DISTINCT order_date) AS visits_amount
FROM sales
GROUP BY customer_id
ORDER BY visits_amount ASC;
```

***description:***

The SQL query retrieves the customer_id and calculates the total number of unique visit dates (visits_amount) for each customer at the restaurant.

- It uses the COUNT(DISTINCT order_date) function to count the number of unique dates each customer visited the restaurant, ensuring duplicate dates are not counted.
- The query groups the results by customer_id, so the count of unique dates is calculated for each individual customer.
- The results are then sorted in ascending order (ASC) based on visits_amount, so customers with the fewest visits appear first.

***answer:***
| customer_id | visits_amount |
| ----------- | ------------- |
| C           | 2             |
| A           | 4             |
| B           | 6             |

3. **What was the first item from the menu purchased by each customer?**

***query:***

***description:***

***answer:***

### Bonus Questions & Solutions
...
### Key Insights