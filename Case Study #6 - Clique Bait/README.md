![Project Logo](../images/case6_logo.png)

## Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Digital Analysis](#digital-analysis)
- [Product Funnel Analysis](#product-funnel-analysis)
- [Campaigns Analysis](#campaigns-analysis)
- [Conclusion](#conclusion)

## Introduction
>Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!
>
>In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

## Entity Relationship Diagram

<details>
  <summary><em><strong>show database schema*</strong></em></summary>

```SQL
CREATE SCHEMA clique_bait;
SET SEARCH_PATH TO clique_bait;

CREATE TABLE event_identifier (
  "event_type" INTEGER,
  "event_name" VARCHAR(13)
);

INSERT INTO event_identifier
  ("event_type", "event_name")
VALUES
  ('1', 'Page View'),
  ('2', 'Add to Cart'),
  ('3', 'Purchase'),
  ('4', 'Ad Impression'),
  ('5', 'Ad Click');

CREATE TABLE campaign_identifier (
  "campaign_id" INTEGER,
  "products" VARCHAR(3),
  "campaign_name" VARCHAR(33),
  "start_date" TIMESTAMP,
  "end_date" TIMESTAMP
);

INSERT INTO campaign_identifier
  ("campaign_id", "products", "campaign_name", "start_date", "end_date")
VALUES
  ('1', '1-3', 'BOGOF - Fishing For Compliments', '2020-01-01', '2020-01-14'),
  ('2', '4-5', '25% Off - Living The Lux Life', '2020-01-15', '2020-01-28'),
  ('3', '6-8', 'Half Off - Treat Your Shellf(ish)', '2020-02-01', '2020-03-31');

CREATE TABLE page_hierarchy (
  "page_id" INTEGER,
  "page_name" VARCHAR(14),
  "product_category" VARCHAR(9),
  "product_id" INTEGER
);

INSERT INTO page_hierarchy
  ("page_id", "page_name", "product_category", "product_id")
VALUES
  ('1', 'Home Page', null, null),
  ('2', 'All Products', null, null),
  ('3', 'Salmon', 'Fish', '1'),
  ('4', 'Kingfish', 'Fish', '2'),
  ('5', 'Tuna', 'Fish', '3'),
  ('6', 'Russian Caviar', 'Luxury', '4'),
  ('7', 'Black Truffle', 'Luxury', '5'),
  ('8', 'Abalone', 'Shellfish', '6'),
  ('9', 'Lobster', 'Shellfish', '7'),
  ('10', 'Crab', 'Shellfish', '8'),
  ('11', 'Oyster', 'Shellfish', '9'),
  ('12', 'Checkout', null, null),
  ('13', 'Confirmation', null, null);

CREATE TABLE users (
  "user_id" INTEGER,
  "cookie_id" VARCHAR(6),
  "start_date" TIMESTAMP
);

INSERT INTO users
  ("user_id", "cookie_id", "start_date")
VALUES
  ('1', 'c4ca42', '2020-02-04'),
  ('2', 'c81e72', '2020-01-18'),
  ('3', 'eccbc8', '2020-02-21'),
  ('4', 'a87ff6', '2020-02-22'),
  ('5', 'e4da3b', '2020-02-01'),
  ...
  ('355a6a', '87a4ba', '8', '2', '12', '2020-03-18 22:42:33.090104'),
  ('355a6a', '87a4ba', '9', '1', '13', '2020-03-18 22:42:59.762131'),
  ('355a6a', '87a4ba', '9', '2', '14', '2020-03-18 22:43:57.15588'),
  ('355a6a', '87a4ba', '10', '1', '15', '2020-03-18 22:44:16.541396'),
  ('355a6a', '87a4ba', '11', '1', '16', '2020-03-18 22:44:18.90083'),
  ('355a6a', '87a4ba', '11', '2', '17', '2020-03-18 22:45:12.670472'),
  ('355a6a', '87a4ba', '12', '1', '18', '2020-03-18 22:45:54.081818'),
  ('355a6a', '87a4ba', '13', '3', '19', '2020-03-18 22:45:54.984666');
```

**\*Note**:
1.  **Primary Key Absence:**
    * Primary keys are intentionally omitted for this educational dataset.
    * This is acceptable due to the static and artificially generated nature of the data.
    * **In production, primary keys are crucial for data integrity and uniqueness.**

2.  **Data Type Inconsistencies:**
    * Inserted values may not match column data types (e.g., string values in INTEGER columns).
    * While PostgreSQL performs implicit conversions, **explicit type casting is recommended for production to ensure data consistency and prevent errors.**

</details>

![Project Logo](../images/case6_diagram.png)

## Digital Analysis
### 1. How many users are there?

***query:***
```SQL
SELECT
  COUNT(DISTINCT user_id) AS unique_users
FROM
  users;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the total number of unique user IDs present in the `users` table.

-   `SELECT COUNT(DISTINCT user_id) AS unique_users`: 
    -   `DISTINCT user_id`: Selects all the unique values from the `user_id` column, eliminating any duplicate user IDs.
    -   `COUNT(...)`: Counts the number of distinct (unique) user IDs.
    -   `AS unique_users`: Renames the resulting count to the column alias `unique_users`.
-   `FROM users`:
    -   Specifies that the data is retrieved from the `users` table.

</details>

***result table:***

| unique_users |
| ------------ |
| 500          |

---

### 2. How many cookies does each user have on average?

***query:***
```SQL
SELECT
  ROUND(AVG(cookies_amount)) AS average_cookies_amount
FROM (
  SELECT
    user_id,
    COUNT(DISTINCT cookie_id) AS cookies_amount
  FROM
    users
  GROUP BY  user_id) sub;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the average number of unique cookies per user from the `users` table.

-   `SELECT ROUND(AVG(cookie_count) AS average_cookies_per_user`:
    -   Calculates the average number of cookies per user, rounding the result to the nearest integer.
    -   `AVG(cookie_count)`: Computes the average of the `cookie_count` values.
    -   `ROUND(..., 2)`: Rounds the average to two decimal places for readability.
    -   `AS average_cookies_per_user`: Aliases the resulting column as `average_cookies_per_user`.
-   `FROM (...) sub`:
    -   Specifies the subquery as the data source.
-   Subquery:
    -   `SELECT user_id, COUNT(DISTINCT cookie_id) AS cookie_count FROM users GROUP BY user_id`:
        -   Calculates the number of unique cookies (`cookie_id`) for each user (`user_id`).
        -   `COUNT(DISTINCT cookie_id)`: Counts the distinct cookies for each user.
        -   `GROUP BY user_id`: Groups the results by `user_id` to count cookies per user.
        -   `AS cookie_count`: Aliases the count of cookies as `cookie_count`.
        -   `FROM users`: Specifies that the data is retrieved from the `users` table.

</details>

***result table:***

| averege_cookies_amount |
| ---------------------- |
| 4                      |

---

### 3. What is the unique number of visits by all users per month?

***query:***
```SQL
SELECT
  EXTRACT(YEAR FROM event_time) AS year,
  TO_CHAR(event_time, 'Month') AS month,
  COUNT(DISTINCT visit_id) AS unique_visits
FROM
  events
GROUP BY
  year,
  month
ORDER BY
  unique_visits DESC;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the unique number of visits (identified by `visit_id`) per month and year from the `events` table.

-   `SELECT EXTRACT(YEAR FROM event_time) AS year, TO_CHAR(event_time, 'Month') AS month, COUNT(DISTINCT visit_id) AS unique_visits`:
    -   `EXTRACT(YEAR FROM event_time) AS year`: Extracts the year from the `event_time` column and aliases it as `year`.
    -   `TO_CHAR(event_time, 'Month') AS month`: Extracts the month from the `event_time` column and aliases it as `month`.
    -   `COUNT(DISTINCT visit_id) AS unique_visits`: Counts the number of unique `visit_id` values, representing unique visits, and aliases the result as `unique_visits`.
-   `FROM events`:
    -   Specifies the `events` table as the data source.
-   `GROUP BY year, month`:
    -   Groups the results by `year` and `month` to count unique visits per month and year.
-   `ORDER BY unique_visits DESC`:
    -   Orders the results in descending order based on the `unique_visits` count.

</details>

***result table:***

| year | month     | unique_visits |
| ---- | --------- | ------------- |
| 2020 | February  | 1488          |
| 2020 | March     | 916           |
| 2020 | January   | 876           |
| 2020 | April     | 248           |
| 2020 | May       | 36            |

---

### 4. What is the number of events for each event type?

***query:***
```SQL
SELECT
  ei.event_name,
  sub.events_amount
FROM
  (SELECT 
    event_type,
    COUNT(*) AS events_amount
  FROM
    events
  GROUP BY
    event_type) sub
JOIN event_identifier ei
USING (event_type)
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the number of events for each event type, displaying the corresponding event name.

-   `SELECT ei.event_name, sub.events_amount`:
    -   Selects the `event_name` from the `event_identifier` table.
    -   Selects the `events_amount` (count of events) from the subquery.
-   `FROM (SELECT event_type, COUNT(*) AS events_amount FROM events GROUP BY event_type) sub`:
    -   Subquery:
        -   `SELECT event_type, COUNT(*) AS events_amount`: Counts all events for each `event_type`.
        -   `FROM events`: Specifies the `events` table as the data source.
        -   `GROUP BY event_type`: Groups the results by `event_type`.
-   `JOIN event_identifier ei USING (event_type)`:
    -   Joins the subquery results with the `event_identifier` table using the `event_type` column.

</details>

***result table:***

| event_name    | events_amount |
| ------------- | ------------- |
| Page View     | 20928         |
| Add to Cart   | 8451          |
| Purchase      | 1777          |
| Ad Impression | 876           |
| Ad Click      | 702           |

---

### 5. What is the percentage of visits which have a purchase event?

***query:***
```SQL
SELECT
  ROUND((COUNT(DISTINCT CASE WHEN ei.event_name = 'Purchase' THEN e.visit_id END)::decimal / COUNT(DISTINCT e.visit_id)) * 100, 2) AS purchase_percentage
FROM
  events e
JOIN
  event_identifier ei USING(event_type);
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the percentage of unique visits (identified by `visit_id`) that include a purchase event.

-   `SELECT ROUND((COUNT(DISTINCT CASE WHEN ei.event_name = 'Purchase' THEN e.visit_id END)::decimal / COUNT(DISTINCT e.visit_id)) * 100, 2) AS purchase_percentage`:
    -   Calculates the percentage of visits with a purchase.
    -   `COUNT(DISTINCT CASE WHEN ei.event_name = 'Purchase' THEN e.visit_id END)`: Counts the number of unique `visit_id` values associated with purchase events.
        -   `CASE WHEN ei.event_name = 'Purchase' THEN e.visit_id END`: Selects the `visit_id` only when the `event_name` is 'Purchase', otherwise returns NULL.
        -   `COUNT(DISTINCT ...)`: Counts the distinct `visit_id` values, effectively counting unique visits with purchases.
    -   `COUNT(DISTINCT e.visit_id)`: Counts the total number of unique `visit_id` values, representing all unique visits.
    -   `::decimal`: Casts the count to decimal to ensure accurate percentage calculation.
    -   `(... / ...) * 100`: Calculates the percentage.
    -   `ROUND(..., 2)`: Rounds the percentage to two decimal places.
    -   `AS purchase_percentage`: Aliases the resulting percentage as `purchase_percentage`.
-   `FROM events e JOIN event_identifier ei USING(event_type)`:
    -   Joins the `events` table (aliased as `e`) with the `event_identifier` table (aliased implicitly by `USING`) using the `event_type` column to access the `event_name`.

</details>

***result table:***
| purchase_percentage |
| ------------------- |
| 49.86               |

---

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

***query:***
```SQL
WITH checkout_visits AS (
  SELECT DISTINCT
    e.visit_id
  FROM
    clique_bait.events e
  JOIN
    clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
  WHERE
    ph.page_name = 'Checkout'
),
purchase_visits AS (
  SELECT DISTINCT
    e.visit_id
  FROM
    clique_bait.events e
  JOIN
    clique_bait.event_identifier ei ON e.event_type = 3
  WHERE
    ei.event_name = 'Purchase'
),
checkout_no_purchase AS (
  SELECT
    cv.visit_id
  FROM
    checkout_visits cv
  LEFT JOIN
    purchase_visits pv ON cv.visit_id = pv.visit_id
  WHERE
    pv.visit_id IS NULL
)
SELECT
  ROUND((COUNT(cnp.visit_id)::decimal / (SELECT COUNT(DISTINCT visit_id) FROM checkout_visits)) * 100, 2) AS checkout_no_purchase_percentage
FROM
  checkout_no_purchase cnp;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the percentage of visits which viewed the 'Checkout' page but did not have a 'Purchase' event.

-   `WITH checkout_visits AS (...)`:
    -   Defines a Common Table Expression (CTE) named `checkout_visits` that selects the distinct `visit_id` of all visits where the `page_name` in the `page_hierarchy` table is 'Checkout'. It joins the `events` table with `page_hierarchy` on `page_id`.
-   `WITH purchase_visits AS (...)`:
    -   Defines a CTE named `purchase_visits` that selects the distinct `visit_id` of all visits where the `event_name` in the `event_identifier` table is 'Purchase'. It joins the `events` table with `event_identifier` on `event_type`.
-   `WITH checkout_no_purchase AS (...)`:
    -   Defines a CTE named `checkout_no_purchase` that selects the `visit_id` from `checkout_visits` that do not exist in `purchase_visits`. This is achieved using a `LEFT JOIN` and filtering for rows where `pv.visit_id` is `NULL`.
-   `SELECT ROUND((COUNT(cnp.visit_id)::decimal / (SELECT COUNT(DISTINCT visit_id) FROM checkout_visits)) * 100, 2) AS checkout_no_purchase_percentage`:
    -   Calculates the percentage of visits with a 'Checkout' view but no 'Purchase' event.
    -   `COUNT(cnp.visit_id)`: Counts the number of `visit_id` values in the `checkout_no_purchase` CTE.
    -   `(SELECT COUNT(DISTINCT visit_id) FROM checkout_visits)`: Counts the total number of distinct `visit_id` values in the `checkout_visits` CTE.
    -   `::decimal`: Casts the count to decimal for accurate division.
    -   `* 100`: Multiplies the result by 100 to get the percentage.
    -   `ROUND(..., 2)`: Rounds the percentage to two decimal places.
    -   `AS checkout_no_purchase_percentage`: Aliases the resulting column name.
-   `FROM checkout_no_purchase cnp`:
    -   Specifies the `checkout_no_purchase` CTE as the source for the final selection.

</details>

***result table:***

| checkout_no_purchase_percentage |
| ------------------------------- |
| 15.50                           |

---

### 7. What are the top 3 pages by number of views?

***query:***
```SQL
SELECT
    ph.page_name,
    COUNT(e.page_id) AS view_count
FROM
    clique_bait.events e
JOIN
    clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
WHERE
    e.event_type = 1
GROUP BY
    ph.page_name
ORDER BY
    view_count DESC
LIMIT 3;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query identifies the top 3 pages with the highest number of views.

-   `SELECT ph.page_name, COUNT(e.page_id) AS view_count`:
    -   Selects the `page_name` from the `page_hierarchy` table.
    -   Counts the occurrences of each `page_id` in the `events` table and aliases the count as `view_count`.
-   `FROM clique_bait.events e JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id`:
    -   Joins the `events` table (aliased as `e`) with the `page_hierarchy` table (aliased as `ph`) using the common column `page_id` to link events to page names.
-   `WHERE e.event_type = 1`:
    -   Filters the events to include only those where `event_type` is equal to `1`. This assumes that `event_type = 1` represents page view events. **You might need to adjust this value based on your actual data.**
-   `GROUP BY ph.page_name`:
    -   Groups the results by `page_name` so that the `COUNT()` function aggregates views for each unique page.
-   `ORDER BY view_count DESC`:
    -   Orders the grouped results in descending order based on the `view_count`, placing the pages with the most views at the top.
-   `LIMIT 3`:
    -   Restricts the output to the top 3 rows, effectively giving the top 3 pages by view count.

</details>

***result table:***

| page_name    | view_count |
| ------------ | ---------- |
| All Products | 3174       |
| Checkout     | 2103       |
| Home Page    | 1782       |

---

### 8. What is the number of views and cart adds for each product category?

***query:***
```SQL
SELECT
  ph.product_category,
  SUM(CASE WHEN ev.event_type = 1 THEN 1 ELSE 0 END) AS product_views,
  SUM(CASE WHEN ev.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds
FROM
  page_hierarchy AS ph
LEFT JOIN
  events AS ev ON ph.page_id = ev.page_id
GROUP BY
  ph.product_category
ORDER BY
  ph.product_category;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query calculates the number of views and cart adds for each product category.

-   `SELECT ph.product_category, SUM(CASE WHEN ev.event_type = 1 THEN 1 ELSE 0 END) AS product_views, SUM(CASE WHEN ev.event_type = 2 THEN 1 ELSE 0 END) AS cart_adds`:
    -   Selects the `product_category` from the `page_hierarchy` table (aliased as `ph`).
    -   Calculates the total number of events with `ev.event_type` equal to `1` for each category and names the resulting column `product_views`.
    -   Calculates the total number of events with `ev.event_type` equal to `2` for each category and names the resulting column `cart_adds`.
-   `FROM page_hierarchy AS ph LEFT JOIN events AS ev ON ph.page_id = ev.page_id`:
    -   Performs a `LEFT JOIN` between the `page_hierarchy` table (aliased as `ph`) and the `events` table (aliased as `ev`) based on the matching `page_id`. This ensures that all categories from `page_hierarchy` are included in the result.
-   `GROUP BY ph.product_category`:
    -   Aggregates the results, grouping rows with the same `product_category` together.
-   `ORDER BY ph.product_category`:
    -   Sorts the final result set alphabetically by the `product_category`.

</details>

***result table:***

| product_category | product_views | cart_adds |
| ---------------- | ------------- | --------- |
| Fish             | 4633          | 2789      |
| Luxury           | 3032          | 1870      |
| Shellfish        | 6204          | 3792      |
| null             | 7059          | 0         |

---

### 9. What are the top 3 products by purchases?

***query:***
```SQL
WITH purchase_visits AS (
    SELECT DISTINCT
        e.visit_id
    FROM
        clique_bait.events e
    JOIN
        clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
    JOIN
        clique_bait.event_identifier ei ON e.event_type = ei.event_type
    WHERE
        ei.event_name = 'Purchase' AND ph.page_name = 'Confirmation'
),
products_added_to_cart_in_purchase_visits AS (
    SELECT
        pv.visit_id,
        ph.page_name AS product_name
    FROM
        purchase_visits pv
    JOIN
        clique_bait.events e ON pv.visit_id = e.visit_id
    JOIN
        clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
    WHERE
        ph.product_id IS NOT NULL
        AND e.event_type = 2
)
SELECT
    product_name,
    COUNT(visit_id) AS purchase_count
FROM
    products_added_to_cart_in_purchase_visits
GROUP BY
    product_name
ORDER BY
    purchase_count DESC
LIMIT 3;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This SQL query identifies the top 3 products by the number of visits that resulted in a purchase, based on whether the product was added to the cart during those visits.

-   `WITH purchase_visits AS (...)`:
    -   Defines a CTE named `purchase_visits` that selects the distinct `visit_id` values for visits where a 'Purchase' event occurred on the 'Confirmation' page. This identifies sessions that ended in a purchase.
-   `WITH products_added_to_cart_in_purchase_visits AS (...)`:
    -   Defines a CTE named `products_added_to_cart_in_purchase_visits`. It joins `purchase_visits` with the `events` and `page_hierarchy` tables to find the `page_name` (assumed to be the product name) of pages with a non-null `product_id` that had an 'Add to Cart' event (`e.event_type = 2`) within those purchase visits.
-   `SELECT product_name, COUNT(visit_id) AS purchase_count`:
    -   Selects the `product_name` and counts the number of distinct `visit_id` values associated with each `product_name` within the `products_added_to_cart_in_purchase_visits` CTE. This effectively counts how many purchase visits included adding each specific product to the cart.
-   `FROM products_added_to_cart_in_purchase_visits`:
    -   Specifies the `products_added_to_cart_in_purchase_visits` CTE as the data source.
-   `GROUP BY product_name`:
    -   Groups the results by `product_name` to aggregate the purchase visit counts for each product.
-   `ORDER BY purchase_count DESC`:
    -   Orders the results in descending order based on the `purchase_count`, showing the products that were most frequently added to the cart in purchase sessions first.
-   `LIMIT 3`:
    -   Limits the output to the top 3 products.

</details>

***result table:***

| product_name | purchase_count |
| ------------ | -------------- |
| Lobster      | 754            |
| Oyster       | 726            |
| Crab         | 719            |

---