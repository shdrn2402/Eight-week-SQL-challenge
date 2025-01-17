![Project Logo](../images/case4_logo.png)

## Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
  - [A. Customer Nodes Exploration](#a-customer-nodes-exploration)
  - [B. Customer Transactions](#b-customer-transactions)
  - [C. Data Allocation Challenge](#c-data-allocation-challenge)
  - [D. Extra Challenge](#d-extra-challenge)
- [Extension Request](#extantion-request)

## Introduction
> Data Bank is a cutting-edge initiative that merges the worlds of digital-only neo-banks, cryptocurrency, and secure data storage. Unlike traditional banks, Data Bank provides its customers not only with financial services but also with highly secure, distributed cloud storage directly tied to their account balances.
>
>The goal of this case study is to explore customer data, calculate key metrics, and analyze trends to help the management team improve customer growth strategies and optimize data storage forecasting for future scalability.

## Entity Relationship Diagram

<details>
  <summary><em><strong>show database schema*</strong></em></summary>

```SQL
CREATE SCHEMA data_bank;
SET search_path = data_bank;

CREATE TABLE regions (
  region_id INTEGER,
  region_name VARCHAR(9)
);

INSERT INTO regions
  (region_id, region_name)
VALUES
  ('1', 'Australia'),
  ('2', 'America'),
  ('3', 'Africa'),
  ('4', 'Asia'),
  ('5', 'Europe');

CREATE TABLE customer_nodes (
  customer_id INTEGER,
  region_id INTEGER,
  node_id INTEGER,
  start_date DATE,
  end_date DATE
);

INSERT INTO customer_nodes
  (customer_id, region_id, node_id, start_date, end_date)
VALUES
  ('1', '3', '4', '2020-01-02', '2020-01-03'),
  ('2', '3', '5', '2020-01-03', '2020-01-17'),
  ('3', '5', '4', '2020-01-27', '2020-02-18'),
  ('4', '5', '4', '2020-01-07', '2020-01-19'),
  ('5', '3', '3', '2020-01-15', '2020-01-23'),
  -- ...
  ('498', '1', '2', '2020-04-05', '9999-12-31'),
  ('499', '5', '1', '2020-02-03', '9999-12-31'),
  ('500', '2', '2', '2020-04-15', '9999-12-31');

CREATE TABLE customer_transactions (
  customer_id INTEGER,
  txn_date DATE,
  txn_type VARCHAR(10),
  txn_amount INTEGER
);

INSERT INTO customer_transactions
  (customer_id, txn_date, txn_type, txn_amount)
VALUES
  ('429', '2020-01-21', 'deposit', '82'),
  ('155', '2020-01-10', 'deposit', '712'),
  ('398', '2020-01-01', 'deposit', '196'),
  ('255', '2020-01-14', 'deposit', '563'),
  ('185', '2020-01-29', 'deposit', '626'),
  -- ...
  ('189', '2020-02-06', 'purchase', '393'),
  ('189', '2020-01-22', 'deposit', '302'),
  ('189', '2020-01-27', 'withdrawal', '861');
```

**\*Note**:
1. Primary keys are not explicitly defined in the tables. This might be intentional due to the educational nature of the project:  
  - The data is artificially generated and static, minimizing the risk of integrity violations.  
  - In real-world scenarios, primary keys are essential to enforce data integrity and uniqueness.  

2. Data type mismatches are present in the inserted values:  
  - For example, string values are being inserted into columns with numeric data types.  
  - PostgreSQL implicitly converts these values, allowing the data to be stored. However, this practice is discouraged in production systems.  
  - Explicit type casting should be used to ensure data consistency and to prevent unexpected errors.  

</details>

![Project Logo](../images/case4_diagram.png)


## Case Study Questions & Solutions
### A. Customer Nodes Exploration
#### 1. How many unique nodes are there on the Data Bank system?

***query:***
```SQL
SELECT
  COUNT(DISTINCT node_id) AS unique_nodes_amount
FROM
  customer_nodes;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the total number of unique `node_id` values in the `customer_nodes` table.

- `COUNT(DISTINCT node_id)`: Counts only the distinct (unique) `node_id` values, ensuring duplicates are not included in the result.
- `AS unique_nodes_amount`: Assigns an alias to the resulting column for better readability.

</details>

***answer:***
| unique_nodes_amount |
| -------------------- |
| 5                    |

---

#### 2. What is the number of nodes per region?

***query:***
```SQL
SELECT
  R.region_name,
  COUNT(DISTINCT CN.node_id) AS unique_nodes_amount
FROM 
  customer_nodes CN
JOIN
  regions R USING(region_id)
GROUP BY
  region_name
ORDER BY
  unique_nodes_amount DESC,
  region_name ASC;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the number of unique nodes (`node_id`) in each region (`region_name`) and sorts the results in descending order of the node count, with a secondary sort by region name in ascending order.

- `COUNT(DISTINCT CN.node_id)`:
  Counts the distinct `node_id` values in each region to determine the number of unique nodes.

- `JOIN regions R USING(region_id)`:
  Joins the `customer_nodes` table with the `regions` table based on the common `region_id` column to retrieve the corresponding region names.

- `GROUP BY region_name`:
  Groups the data by each `region_name` to calculate the count of unique nodes for each region.

- `ORDER BY unique_nodes_amount DESC, region_name ASC`:
  Sorts the results first by the number of unique nodes in descending order, and for regions with the same count, by the region name in ascending alphabetical order.

The query provides a list of regions and the number of unique nodes they contain.

</details>

***answer:***
| region_name | unique_nodes_amount |
| ----------- | -------------------- |
| Africa      | 5                    |
| America     | 5                    |
| Asia        | 5                    |
| Australia   | 5                    |
| Europe      | 5                    |

---

#### 3. How many customers are allocated to each region?

***query:***
```SQL
SELECT
  R.region_name,
  COUNT(DISTINCT CN.customer_id) AS customers_amount
FROM
  customer_nodes CN
JOIN
  regions R USING(region_id)
GROUP BY
  region_name
ORDER BY
  customers_amount DESC,
  region_name ASC;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the number of unique customers allocated to each region.

- `COUNT(DISTINCT CN.customer_id)`: Counts the unique customers in each region by using the `DISTINCT` keyword to ensure no duplicates are included.
  
- `FROM customer_nodes CN JOIN regions R USING(region_id)`:
  - Combines the `customer_nodes` table and the `regions` table.
  - The `USING(region_id)` clause joins the tables based on their shared `region_id` column.

- `GROUP BY region_name`: Groups the data by `region_name`, ensuring the customer count is calculated for each region.

- `ORDER BY customers_amount DESC, region_name ASC`:
  - Sorts the results in descending order of `customers_amount` (most customers appear first).
  - In case of a tie in `customers_amount`, the results are sorted alphabetically by `region_name` in ascending order.

</details>

***answer:***
| region_name | customers_amount |
| ----------- | ---------------- |
| Australia   | 110              |
| America     | 105              |
| Africa      | 102              |
| Asia        | 95               |
| Europe      | 88               |

---

#### 4. How many days on average are customers reallocated to a different node?

***query:***
```SQL
WITH days_on_node_count AS (
  SELECT
    customer_id,
    node_id,
    SUM(end_date - start_date) AS days_on_node
  FROM
    customer_nodes
  WHERE
    end_date != '9999-12-31'
  GROUP BY
    customer_id, node_id
)
SELECT
  FLOOR(AVG(days_on_node)) AS avg_days_on_node
FROM
  days_on_node_count;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The query calculates the average number of days customers spend on a specific node, excluding records with an `end_date` of `'9999-12-31'`, which is assumed to represent active records.

CTE: `days_on_node_count`:
   - Groups data by `customer_id` and `node_id` to calculate the total number of days (`days_on_node`) each customer spends on a specific node.
   - The difference between `end_date` and `start_date` is computed for each record, with a filter to exclude rows where `end_date = '9999-12-31'`.

Final Query:
   - Computes the average number of days customers spend on a node by taking the average (`AVG`) of the `days_on_node` values.
   - The result is rounded down to the nearest whole number using the `FLOOR` function.

**\*Note**: 
- `'9999-12-31'` is presumed to indicate active records based on the dataset analysis. These records are excluded to avoid skewing the average with incomplete data.

</details>

***answer:***
| avg_days_on_node |
| ---------------- |
| 23               |

---

#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

***query:***
```SQL
WITH days_on_node_count AS (
  SELECT
    customer_id,
    region_id,
    node_id,
    SUM(end_date - start_date) AS days_on_node
  FROM
    customer_nodes
  WHERE
    end_date != '9999-12-31'
  GROUP BY
    customer_id, region_id, node_id
)

SELECT
  R.region_name,
  PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY DNC.days_on_node) AS median,
  PERCENTILE_DISC(0.8) WITHIN GROUP (ORDER BY DNC.days_on_node) AS percentile_80,
  PERCENTILE_DISC(0.95) WITHIN GROUP (ORDER BY DNC.days_on_node) AS percentile_95
FROM
  days_on_node_count DNC
JOIN
  regions R USING(region_id)
GROUP BY
  R.region_name
ORDER BY
  R.region_name
;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This query calculates the median, 80th percentile, and 95th percentile of the days customers are reallocated to different nodes, grouped by region, while considering both the node and the region level.

- `WITH days_on_node_count`:
  - Aggregates the total days spent on each node for every customer.
  - Filters out rows with `end_date = '9999-12-31'`, which indicates active or incomplete records.
  - Groups the data by `customer_id`, `region_id`, and `node_id`.
  - Calculates the total days spent on each node (`SUM(end_date - start_date)`).

- Main `SELECT` statement:
  - Joins the aggregated node data (`days_on_node_count`) with the `regions` table using `region_id`.
  - Calculates the following metrics for each region:
    - `PERCENTILE_CONT()` computes the Nth percentile, selecting an actual value from the dataset without interpolation.
  - Groups the results by `region_name`.

- `ORDER BY` ensures the output is sorted alphabetically by `region_name`.

This approach accounts for detailed node-level data while providing insights at the regional level.

</details>

***answer:***
| region_name | median | percentile_80 | percentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 22     | 35            | 54            |
| America     | 22     | 34            | 54            |
| Asia        | 22     | 35            | 52            |
| Australia   | 21     | 34            | 51            |
| Europe      | 23     | 34            | 52            |

---

### B. Customer Transactions
#### 1. What is the unique count and total amount for each transaction type?

***query:***
```SQL
SELECT
  txn_type,
  COUNT(DISTINCT txn_amount) AS unique_txn_amount,
  SUM(txn_amount) AS total_txn_amount
FROM
  customer_transactions
GROUP BY
  txn_type
ORDER BY
  txn_type ASC
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates metrics related to transaction types from the `customer_transactions` table.

- `txn_type`: Retrieves the type of transaction (e.g., deposit, withdrawal, purchase).
- `COUNT(DISTINCT txn_amount)`: Counts the unique transaction amounts for each transaction type.
- `SUM(txn_amount)`: Computes the total transaction amount for each transaction type.
- `FROM customer_transactions`: Specifies the table containing the transaction data.
- `GROUP BY txn_type`: Groups the results by transaction type to calculate metrics for each type.
- `ORDER BY txn_type ASC`: Ensures the output is sorted in ascending order of transaction type.

The query returns a table with each transaction type, the count of unique transaction amounts, and the total transaction amount.

</details>

***answer:***
| txn_type   | unique_txn_amount | total_txn_amount |
| ---------- | ----------------- | ---------------- |
| deposit    | 929               | 1359168          |
| purchase   | 815               | 806537           |
| withdrawal | 804               | 793003           |

---

#### 2. What is the average total historical deposit counts and amounts for all customers?

***query:***
```SQL
WITH totals AS (
  SELECT
    customer_id,
    COUNT(*) AS total_deposits_amount,
    SUM(txn_amount) AS total_deposits_sum
  FROM
    customer_transactions
  WHERE
    txn_type = 'deposit'
  GROUP BY
    customer_id,
    txn_type
)
SELECT
  ROUND(AVG(total_deposits_amount)) AS avg_deposits_amount,
  ROUND(AVG(total_deposits_sum)) AS avg_deposits_sum
FROM
  totals;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This query calculates the average total deposit counts and amounts for all customers.

- `WITH totals AS`:
  - Creates a Common Table Expression (CTE) named `totals` that calculates the total number of deposits (`COUNT(*)`) and the total sum of deposit amounts (`SUM(txn_amount)`) for each customer.
  - Filters the data to include only rows where the `txn_type` is 'deposit'.
  - Groups the data by `customer_id` and `txn_type` to compute the aggregated metrics per customer.

- Main `SELECT` statement:
  - Uses the `totals` CTE as the data source.
  - `ROUND(AVG(total_deposits_amount))`: Calculates the average number of deposits made by each customer and rounds the result.
  - `ROUND(AVG(total_deposits_sum))`: Calculates the average total deposit amount for all customers and rounds the result.

- The query returns two metrics:
  - `avg_deposits_amount`: Average number of deposits per customer.
  - `avg_deposits_sum`: Average total amount of deposits per customer.

</details>

***answer:***
| avg_deposits_amount | avg_deposits_sum |
| ------------------- | ---------------- |
| 5                   | 2718             |

---

#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

***query:***
```SQL
WITH monthly_activity AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE AS current_month,
    txn_type,
    COUNT(txn_type) AS txn_count
  FROM
    customer_transactions
  GROUP BY
    customer_id,
    DATE_TRUNC('month', txn_date),
    txn_type
),
filtered_deposits AS (
  SELECT
    customer_id,
    current_month
  FROM
    monthly_activity
  WHERE
    txn_type = 'deposit' AND txn_count > 1
),
filtered_purchases_withdrawals AS (
  SELECT
    customer_id,
    current_month
  FROM
    monthly_activity
  WHERE
    txn_type IN ('purchase', 'withdrawal') AND txn_count = 1
),
combined_data AS (
  SELECT DISTINCT
    d.customer_id,
    d.current_month
  FROM
    filtered_deposits d
  JOIN
    filtered_purchases_withdrawals p
  ON
    d.customer_id = p.customer_id AND d.current_month = p.current_month
)
SELECT
  current_month,
  COUNT(DISTINCT customer_id) AS customer_count
FROM
  combined_data
GROUP BY
  current_month
ORDER BY
  current_month;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

This query calculates the number of Data Bank customers who, in a single month, made more than 1 deposit and at least 1 purchase or withdrawal.

- `WITH monthly_activity AS (...)`:
  - Groups transactions by `customer_id`, month (`current_month`), and `txn_type`.
  - Calculates the count of transactions for each type using `COUNT(txn_type)`.

- `WITH filtered_deposits AS (...)`:
  - Selects rows from `monthly_activity` where `txn_type = 'deposit'` and `txn_count > 1`.

- `WITH filtered_purchases_withdrawals AS (...)`:
  - Selects rows from `monthly_activity` where `txn_type` is `'purchase'` or `'withdrawal'` and `txn_count = 1`.

- `WITH combined_data AS (...)`:
  - Joins `filtered_deposits` and `filtered_purchases_withdrawals` on `customer_id` and `current_month`.
  - Ensures both conditions are met for the same customer in the same month.

- Final `SELECT`:
  - Groups the resulting data by `current_month`.
  - Counts distinct `customer_id` in each group.
  - Orders the results chronologically by `current_month`.

This query methodically filters, joins, and aggregates data to provide the required customer activity metrics.

</details>

***answer:***
| current_month | customer_count |
| ------------- | -------------- |
| 2020-01-01    | 115            |
| 2020-02-01    | 108            |
| 2020-03-01    | 113            |
| 2020-04-01    | 50             |

---

#### 4. What is the closing balance for each customer at the end of the month?

***query:***
```SQL
WITH current_month_balance AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE AS current_month,
    SUM(
      CASE
        WHEN
          txn_type = 'deposit'
        THEN
          txn_amount
        ELSE
          -txn_amount
      END
    ) AS txn_amount_with_sign
  FROM
    customer_transactions
  GROUP BY
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE
  )
 
SELECT
  customer_id,
  current_month,
  SUM(txn_amount_with_sign) 
    OVER (PARTITION BY customer_id ORDER BY current_month) AS closing_balance
FROM
  current_month_balance
ORDER BY
  customer_id,
  current_month
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The query calculates the closing balance for each customer at the end of each month based on transaction data.

- The `WITH` clause defines a Common Table Expression (CTE) named `current_month_balance` that computes the net transaction amount for each customer for every month.
- Inside the CTE:
  - `DATE_TRUNC('month', txn_date)::DATE` truncates the transaction date to the start of the month.
  - `SUM(CASE ... END)` calculates the net transaction amount by adding deposit amounts and subtracting withdrawal amounts.
  - The results are grouped by customer ID and truncated month using `GROUP BY`.
- The main query:
  - Selects the `customer_id`, `current_month`, and computes the cumulative sum of transaction amounts for each customer using `SUM(txn_amount_with_sign) OVER (PARTITION BY customer_id ORDER BY current_month)`.
  - The `PARTITION BY customer_id` ensures the cumulative sum is calculated separately for each customer.
  - The `ORDER BY current_month` ensures transactions are processed in chronological order.
- The final result includes the closing balance for each customer at the end of each month, sorted by `customer_id` and `current_month`.

</details>

***answer:***
| customer_id | current_month | closing_balance |
| ----------- | ------------- | --------------- |
| 1           | 2020-01-01    | 312             |
| 1           | 2020-03-01    | -640            |
| 2           | 2020-01-01    | 549             |
| 2           | 2020-03-01    | 610             |
| 3           | 2020-01-01    | 144             |
| 3           | 2020-02-01    | -821            |
| 3           | 2020-03-01    | -1222           |
| 3           | 2020-04-01    | -729            |
| ...         | ...           | ...             |
| 500         | 2020-01-01    | 1594            |
| 500         | 2020-02-01    | 2981            |
| 500         | 2020-03-01    | 2251            |

---

#### 5. What is the percentage of customers who increase their closing balance by more than 5%?

***query:***

```SQL

```

<details>
  <summary><em><strong>show description:</strong></em></summary>


</details>

***answer:***

### C. Data Allocation Challenge
...

### D. Extra Challenge
...

### Extension Request
... 