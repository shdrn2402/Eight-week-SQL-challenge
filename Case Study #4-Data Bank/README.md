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

>**\*Note**:
>The phrasing of the question allows for ambiguous interpretation.  
>If we consider it in conjunction with the previous one — *What is the percentage of customers who increase their closing balance by more than 5%?* — the calculation would need to be performed for each month.  
>On the other hand, the question itself does not specify a monthly comparison of closing balances.  
>
>In addition, the result depends on whether the first month is included in the calculation. In that case, the closing balance would be compared to $0.00, as there is no closing balance for the previous month.  
>
>In my calculations, I will use the following methodology:  
>- comparison based on the closing balance on the last date for all customer records;  
>- the first month will not be counted as growth; however, its closing balance will be used for the calculations in subsequent months.

***query:***
```SQL
WITH monthly_txn_summary AS (
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
  ),
customer_closing_balances AS (
  SELECT
    customer_id,
    current_month,
    SUM(txn_amount_with_sign) 
      OVER (PARTITION BY customer_id ORDER BY current_month) AS closing_balance,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY current_month ASC) AS row_num_start,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY current_month DESC) AS row_num_end
  FROM
    monthly_txn_summary
),
growth_calculation AS (
  SELECT
    start_balances.customer_id,
    ROUND(((end_balances.closing_balance - start_balances.closing_balance) / 
      ABS(start_balances.closing_balance)) * 100, 2) AS growth_percentage
  FROM
    customer_closing_balances start_balances
  JOIN
    customer_closing_balances end_balances USING(customer_id)
  WHERE
    start_balances.row_num_start = 1 AND end_balances.row_num_end = 1
),
filtered_growth AS (
  SELECT
    customer_id
  FROM
    growth_calculation
  WHERE
    growth_percentage > 5
)
SELECT
  ROUND((COUNT(DISTINCT filtered_growth.customer_id) * 100.0) /
  (SELECT COUNT(DISTINCT customer_id) FROM customer_transactions), 2) AS percentage_of_customers
FROM
  filtered_growth;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The query calculates the percentage of customers whose closing balance increased by more than 5% over the entire observation period.

- CTE `monthly_txn_summary`:
  - Computes the net transaction amount (`txn_amount_with_sign`) for each customer for each month.
  - Uses `DATE_TRUNC('month', txn_date)::DATE` to truncate transaction dates to the start of the month.
  - Calculates the net transaction amount by summing deposits and subtracting withdrawals using `SUM(CASE ... END)`.
  - Groups results by `customer_id` and `current_month`.

- CTE `customer_closing_balances`:
  - Calculates the cumulative closing balance for each customer using `SUM(txn_amount_with_sign) OVER (PARTITION BY customer_id ORDER BY current_month)`.
  - Adds `row_num_start` and `row_num_end` using `ROW_NUMBER` to identify the first and last rows for each customer.

- CTE `growth_calculation`:
  - Joins the first and last rows for each customer based on `row_num_start` and `row_num_end`.
  - Calculates the percentage growth of the closing balance as `(end_balance - start_balance) / ABS(start_balance) * 100`.
  - Rounds the growth percentage to two decimal places.

- CTE `filtered_growth`:
  - Filters customers whose growth percentage is greater than 5%.

- Final query:
  - Calculates the percentage of customers with growth > 5% by dividing the count of filtered customers by the total number of unique customers (`COUNT(DISTINCT customer_id)`).
  - Rounds the result to two decimal places.

</details>

***answer:***
| percentage_of_customers |
| ----------------------- |
| 33.20                   |

---

### C. Data Allocation Challenge

> To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:
> - option 1: data is allocated based off the amount of money at the end of the previous month  
> - option 2: data is allocated on the average amount of money kept in the account in the previous 30 days  
> - option 3: data is updated real-time  
>
> For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:
> - customer balance at the end of each month  
> - average values of the running balance for each customer  
> - running customer balance column that includes the impact of each transaction  
>
> Since the task does not specify rules for calculating storage volumes based on balances, we will define these rules ourselves.
>
> **Storage volumes calculating rules:**
>
> 1. Each customer receives 100 GB of cloud storage upon starting to use our services. This 100 GB remains with the customer permanently.
> 2. A negative balance is considered a credit and does not decrease the storage volume. Moreover, an increasing coefficient is applied to calculate the storage volume, as the bank benefits from the customer’s use of borrowed funds.
> 3. Basic formula for storage calculation:
> - for a positive balance: Storage volume (GB) = (relevant-balance / 10) + 100
> - for a negative balance: Storage volume (GB) = abs(relevant-balance) / 8 + 100

<details>
  <summary><em>show examples:</em></summary>

---

> a. Positive balance: 500.
> 
> Storage volume = (500 / 10) + 100 = 50 + 100 = **150 GB**.
>
> b. Negative balance: -400.
> 
> Storage volume = abs(-400) / 8 + 100 = 50 + 100 = **150 GB**.

---

</details>

---

#### 1. How much data would be required on a monthly basis, based on the customer's balance at the end of each month?

***query:***
```SQL
WITH month_balance_counting AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE AS transaction_month,
    SUM(
      CASE
        WHEN txn_type = 'deposit'
        THEN txn_amount
        ELSE -txn_amount
      END
      ) AS month_balance
  FROM
    customer_transactions
  GROUP BY
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE
  ),
сumulative_balance_counting AS (
  SELECT
    customer_id,
    transaction_month,
    SUM(month_balance) 
      OVER (PARTITION BY customer_id ORDER BY transaction_month) AS closing_balance
  FROM
    month_balance_counting
)

SELECT
  *,
  CASE
    WHEN closing_balance >= 0
    THEN ROUND((closing_balance / 10) + 100, 0)
    ELSE ROUND(abs(closing_balance) / 8 + 100, 0)
  END AS storage_volume_Gb
FROM
  сumulative_balance_counting
ORDER BY
  customer_id,
  transaction_month
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the closing balance for each customer at the end of each month and determines the corresponding storage volume based on predefined rules.

- CTE `month_balance_counting`:
  - Aggregates transaction data for each customer on a monthly basis.
  - Uses `DATE_TRUNC('month', txn_date)::DATE` to group transactions by the start of the month.
  - Calculates the net transaction amount (`txn_amount_with_sign`) by summing deposits and subtracting withdrawals for each month and customer.

- CTE `closing_balance_counting`:
  - Calculates the cumulative closing balance for each customer.
  - Uses `SUM(txn_amount_with_sign) OVER (PARTITION BY customer_id ORDER BY transaction_month)` to compute the running total of transaction amounts for each customer, ensuring chronological order by month.
  - Includes the `transaction_month` column for monthly grouping.

- Main `SELECT` statement:
  - Adds a computed column for storage volume (`storage_volume_Gb`):
    - For positive balances: `(closing_balance / 10) + 100`.
    - For negative balances: `abs(closing_balance) / 8 + 100`.
    - The `ROUND` function ensures the storage volume is rounded to the nearest whole number.
  - Displays all columns, including `customer_id`, `transaction_month`, `closing_balance`, and the calculated `storage_volume_Gb`.

- `ORDER BY customer_id, transaction_month`:
  - Ensures the results are sorted by `customer_id` and then by `transaction_month`.

This query provides a detailed monthly view of closing balances and their corresponding storage volumes for all customers. The use of CTEs simplifies the logical steps by isolating the transaction aggregation and running balance calculations.

</details>

***answer:***
| customer_id | transaction_month | closing_balance | storage_volume_gb |
| ----------- | ----------------- | --------------- | ----------------- |
| 1           | 2020-01-01        | 312             | 131               |
| 1           | 2020-03-01        | -640            | 180               |
| 2           | 2020-01-01        | 549             | 155               |
| 2           | 2020-03-01        | 610             | 161               |
| 3           | 2020-01-01        | 144             | 114               |
| 3           | 2020-02-01        | -821            | 203               |
| 3           | 2020-03-01        | -1222           | 253               |
| 3           | 2020-04-01        | -729            | 191               |
| 4           | 2020-01-01        | 848             | 185               |
| 4           | 2020-03-01        | 655             | 166               |
| ...         | ...               | ...             | ...               |
|500	        | 2020-01-01	      | 1594	          | 259               |
|500	        | 2020-02-01	      | 2981	          | 398               |
|500	        | 2020-03-01	      | 2251	          | 325               |

---

#### 2. How much data would be required on a monthly basis, based on the customer's average monthly balance?

>The original goal was formulated as 'data is allocated on the average amount of money kept in the account in the previous 30 days.' However, since the data in our database is limited to the year 2021 and no rules were specified regarding how often the storage volume should be recalculated, it is unclear from which date to count the 'previous 30 days.' In light of this, 'previous 30 days' is interpreted as the previous month.

***query:***
```SQL
WITH apply_sign_to_tnx AS (
  SELECT
    customer_id,
    DATE_TRUNC('month', txn_date)::DATE AS txn_month,
    CASE
      WHEN txn_type = 'deposit'
      THEN txn_amount
      ELSE -txn_amount
    END AS txn_amount_signed
  FROM
    customer_transactions
),
avg_month_balance_counting AS (
  SELECT
    customer_id,
    txn_month,
    ROUND(AVG(txn_amount_signed), 2) AS avg_month_balance
  FROM
    apply_sign_to_tnx
  GROUP BY
    customer_id,
    txn_month
)

SELECT
  *,
  CASE
    WHEN avg_month_balance >= 0
    THEN ROUND((avg_month_balance / 10) + 100, 0)
    ELSE ROUND(abs(avg_month_balance) / 8 + 100, 0)
  END AS storage_volume_Gb
FROM
  avg_month_balance_counting
ORDER BY
  customer_id,
  txn_month
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the average monthly balance for each customer and determines the corresponding storage volume based on predefined rules.

- CTE `apply_sign_to_tnx`:
  - Processes transaction data by applying a positive or negative sign to transaction amounts based on their type.
  - Groups transactions by `customer_id` and truncates the transaction date to the start of the month using `DATE_TRUNC('month', txn_date)::DATE`.
  - Produces a signed transaction amount column (`txn_amount_signed`).

- CTE `avg_month_balance_counting`:
  - Aggregates the data from `apply_sign_to_tnx` to calculate the average transaction amount for each month and customer.
  - Uses the `AVG` function to compute the monthly average balance and rounds the result to two decimal places (`ROUND(AVG(txn_amount_signed), 2)`).
  - Groups data by `customer_id` and `txn_month`.

- Main `SELECT` statement:
  - Adds a computed column for storage volume (`storage_volume_Gb`):
    - For positive average balances: `(avg_month_balance / 10) + 100`.
    - For negative average balances: `abs(avg_month_balance) / 8 + 100`.
    - The `ROUND` function ensures the storage volume is rounded to the nearest whole number.
  - Displays all columns, including `customer_id`, `txn_month`, `avg_month_balance`, and the calculated `storage_volume_gb`.

- `ORDER BY customer_id, txn_month`:
  - Ensures the results are sorted by `customer_id` and then by `txn_month`.

This query provides a detailed monthly view of average balances and their corresponding storage volumes for all customers. The use of CTEs organizes the process into logical steps, making the calculations and transformations more manageable.

</details>

***answer:***
| customer_id | txn_month  | avg_month_balance | storage_volume_gb |
| ----------- | ---------- | ----------------- | ----------------- |
| 1           | 2020-01-01 | 312.00            | 131               |
| 1           | 2020-03-01 | -317.33           | 140               |
| 2           | 2020-01-01 | 549.00            | 155               |
| 2           | 2020-03-01 | 61.00             | 106               |
| 3           | 2020-01-01 | 144.00            | 114               |
| 3           | 2020-02-01 | -965.00           | 221               |
| 3           | 2020-03-01 | -200.50           | 125               |
| 3           | 2020-04-01 | 493.00            | 149               |
| 4           | 2020-01-01 | 424.00            | 142               |
| 4           | 2020-03-01 | -193.00           | 124               |
| ...         | ...        | ...               | ...               |
| 500         | 2020-01-01 | 265.67            | 127               |
| 500         | 2020-02-01 | 462.33            | 146               |
| 500         | 2020-03-01 | -104.29           | 113               |
---

### D. Extra Challenge
...

### Extension Request
... 