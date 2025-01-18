![Project Logo](../images/case4_logo.png)

### Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions & Solutions](#case-study-questions--solutions)
  - [A. Customer Nodes Exploration](#a-customer-nodes-exploration)
  - [B. Customer Transactions](#b-customer-transactions)
  - [C. Data Allocation Challenge](#c-data-allocation-challenge)
  - [D. Extra Challenge](#d-extra-challenge)
- [Extension Request](#extantion-request)



### Introduction

> Data Bank is a cutting-edge initiative that merges the worlds of digital-only neo-banks, cryptocurrency, and secure data storage. Unlike traditional banks, Data Bank provides its customers not only with financial services but also with highly secure, distributed cloud storage directly tied to their account balances.
>
>The goal of this case study is to explore customer data, calculate key metrics, and analyze trends to help the management team improve customer growth strategies and optimize data storage forecasting for future scalability.

### Entity Relationship Diagram

<details>
  <summary><em>show database schema <b>*</b></em></summary>

```SQL
CREATE SCHEMA data_bank;
SET search_path = data_bank;

CREATE TABLE regions (
  region_id INTEGER,
  region_name VARCHAR(9)
);

CREATE TABLE customer_nodes (
  customer_id INTEGER,
  region_id INTEGER,
  node_id INTEGER,
  start_date DATE,
  end_date DATE
);

CREATE TABLE customer_transactions (
  customer_id INTEGER,
  txn_date DATE,
  txn_type VARCHAR(10),
  txn_amount INTEGER
);
```

**\*Note**: Primary keys are not explicitly defined in the tables, likely due to the educational nature of the project.
- The data is artificially generated and static, minimizing the risk of integrity violations.
- In real-world scenarios, defining primary keys is essential to ensure data integrity and uniqueness.

</details>


![Project Logo](../images/case4_diagram.png)


### Case Study Questions & Solutions
#### A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**

*query:*

```SQL
SELECT
  COUNT(DISTINCT node_id) AS unique_nodes_amount
FROM
  customer_nodes;
```

<details>
  <summary><em>show description</em></summary>

The SQL query calculates the total number of unique `node_id` values in the `customer_nodes` table.

- `COUNT(DISTINCT node_id)`: Counts only the distinct (unique) `node_id` values, ensuring duplicates are not included in the result.
- `AS unique_nodes_amount`: Assigns an alias to the resulting column for better readability.

</details>

*answer:*

| unique_nodes_amount |
| -------------------- |
| 5                    |

**2. What is the number of nodes per region?**

*query:*

```SQL
SELECT
  R.region_name,
  COUNT(DISTINCT CN.node_id) AS unique_nodes_amount
FROM 
  customer_nodes CN
JOIN
  regions R USING(region_id)
GROUP BY region_name
ORDER BY unique_nodes_amount DESC, region_name ASC;
```

<details>
  <summary><em>show description</em></summary>

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

*answer:*
| region_name | unique_nodes_amount |
| ----------- | -------------------- |
| Africa      | 5                    |
| America     | 5                    |
| Asia        | 5                    |
| Australia   | 5                    |
| Europe      | 5                    |

**3. How many customers are allocated to each region?**

*query:*

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
  customers_amount DESC, region_name ASC;
```

<details>
  <summary><em>show description</em></summary>

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

*answer:*
| region_name | customers_amount |
| ----------- | ---------------- |
| Australia   | 110              |
| America     | 105              |
| Africa      | 102              |
| Asia        | 95               |
| Europe      | 88               |

**4. How many days on average are customers reallocated to a different node?**

*query:*

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
  <summary><em>show description</em></summary>

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

*answer:*
| avg_days_on_node |
| ---------------- |
| 23               |

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

*query:*

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
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY DNC.days_on_node) AS median,
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
  <summary><em>show description</em></summary>

This query calculates the median, 80th percentile, and 95th percentile of the days customers are reallocated to different nodes, grouped by region, while considering both the node and the region level.

- `WITH days_on_node_count`:
  - Aggregates the total days spent on each node for every customer.
  - Filters out rows with `end_date = '9999-12-31'`, which indicates active or incomplete records.
  - Groups the data by `customer_id`, `region_id`, and `node_id`.
  - Calculates the total days spent on each node (`SUM(end_date - start_date)`).

- Main `SELECT` statement:
  - Joins the aggregated node data (`days_on_node_count`) with the `regions` table using `region_id`.
  - Calculates the following metrics for each region:
    - `PERCENTILE_CONT(0.5)` computes the median based on a continuous distribution of days.
    - `PERCENTILE_DISC(0.8)` computes the 80th percentile, selecting an actual value from the dataset without interpolation.
    - `PERCENTILE_DISC(0.95)` computes the 95th percentile, similar to the 80th percentile.
  - Groups the results by `region_name`.

- `ORDER BY` ensures the output is sorted alphabetically by `region_name`.

This approach accounts for detailed node-level data while providing insights at the regional level.

</details>

*answer:*
| region_name | median | percentile_80 | percentile_95 |
| ----------- | ------ | ------------- | ------------- |
| Africa      | 22     | 35            | 54            |
| America     | 22     | 34            | 54            |
| Asia        | 22     | 35            | 52            |
| Australia   | 21     | 34            | 51            |
| Europe      | 23     | 34            | 52            |

#### B. Customer Transactions
...

#### C. Data Allocation Challenge
...

#### D. Extra Challenge
...

### Extension Request
... 