![Project Logo](../images/case5_logo.png)

## Contents:
- [Introduction](#introduction)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Case Study Questions](#case-study-questions)
  - [A. Data Cleansing Steps](#a-data-cleansing-steps)
  - [B. Data Exploration](#b-data-exploration)
  - [C. Before & After Analysis](#c-before--after-analysis)
  - [D. Bonus Question](#d-bonus-question)
- [Conclusion](#conclusion)

## Introduction
>Data Mart is an analytical project aimed at studying the impact of supply chain changes on sales. In June 2020, the company switched to fully sustainable packaging materials covering the entire process—from farm to customer.
>
>The project's goal is to quantify the effects of these changes and identify key factors affected by the updated supply strategy.
>
>Key research objectives:
>
>- Determine the impact of implementing sustainable packaging on sales.
>- Identify the platforms, regions, segments, and customer types most affected by the changes.
>- Develop recommendations to minimize potential negative effects during future implementations of similar initiatives.

## Entity Relationship Diagram

<details>
  <summary><em><strong>show database schema*</strong></em></summary>

```SQL
CREATE SCHEMA data_mart;
SET search_path = data_mart;


DROP TABLE IF EXISTS data_mart.weekly_sales;
CREATE TABLE data_mart.weekly_sales (
  "week_date" VARCHAR(7),
  "region" VARCHAR(13),
  "platform" VARCHAR(7),
  "segment" VARCHAR(4),
  "customer_type" VARCHAR(8),
  "transactions" INTEGER,
  "sales" INTEGER
);

INSERT INTO data_mart.weekly_sales
  ("week_date", "region", "platform", "segment", "customer_type", "transactions", "sales")
VALUES
  ('31/8/20', 'ASIA', 'Retail', 'C3', 'New', '120631', '3656163'),
  ('31/8/20', 'ASIA', 'Retail', 'F1', 'New', '31574', '996575'),
  ('31/8/20', 'USA', 'Retail', 'null', 'Guest', '529151', '16509610'),
  ('31/8/20', 'EUROPE', 'Retail', 'C1', 'New', '4517', '141942'),
  ('31/8/20', 'AFRICA', 'Retail', 'C2', 'New', '58046', '1758388'),
  ('31/8/20', 'CANADA', 'Shopify', 'F2', 'Existing', '1336', '243878'),
  ('31/8/20', 'AFRICA', 'Shopify', 'F3', 'Existing', '2514', '519502'),
  ('31/8/20', 'ASIA', 'Shopify', 'F1', 'Existing', '2158', '371417'),
  ('31/8/20', 'AFRICA', 'Shopify', 'F2', 'New', '318', '49557'),
  ('31/8/20', 'AFRICA', 'Retail', 'C3', 'New', '111032', '3888162'),
  ...
  ('26/3/18', 'SOUTH AMERICA', 'Shopify', 'F1', 'New', '3', '677'),
  ('26/3/18', 'ASIA', 'Retail', 'F3', 'New', '81842', '2673553'),
  ('26/3/18', 'CANADA', 'Shopify', 'C3', 'New', '48', '7672'),
  ('26/3/18', 'EUROPE', 'Shopify', 'F3', 'New', '2', '300'),
  ('26/3/18', 'USA', 'Retail', 'C3', 'New', '39356', '1617709'),
  ('26/3/18', 'AFRICA', 'Retail', 'C3', 'New', '98342', '3706066'),
  ('26/3/18', 'USA', 'Shopify', 'C4', 'New', '16', '2784'),
  ('26/3/18', 'USA', 'Retail', 'F2', 'New', '25665', '1064172'),
  ('26/3/18', 'EUROPE', 'Retail', 'C4', 'New', '883', '33523'),
  ('26/3/18', 'AFRICA', 'Retail', 'C3', 'Existing', '218516', '12083475');
```

**\*Note**:
1. Primary keys are not explicitly defined in the tables. This might be intentional due to the educational nature of the project:  
  - The data is artificially generated and static, minimizing the risk of integrity violations.  
  - In real-world scenarios, primary keys are essential to enforce data integrity and uniqueness.  

2. Data type inconsistencies are present in inserted values:
  - For example, the week_date column is defined as VARCHAR(7), while storing date-like values.
  - PostgreSQL implicitly converts values when necessary, but this practice is discouraged in production environments.
  - Explicit type casting should be used to ensure data consistency and prevent unexpected errors.

</details>

![Project Logo](../images/case4_diagram.png)


## Case Study Questions
### A. Data Cleansing Steps

***Task Description***
>In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
>
>- Convert the week_date to a DATE format
>
>- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
>
>- Add a month_number with the calendar month for each week_date value as the 3rd column
>
>- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
>
>- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value
>
>| segment	| age_band    |
>|:--------|:------------|
>|1        |Young Adults |
>|2        |Middle Aged  |
>|3 or 4   | Retirees    |
>
>- Add a new demographic column using the following mapping for the first letter in the segment values:
>
>| segment | demographic |
>|:--------|:------------|
>|C        |Couples      |
>|F	      |Families     |
>
>- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
>
>- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record

**\*Note**: 
While the original task does not specify the inclusion of `sales`, `platform` and `transactions` columns in the `clean_weekly_sales` table, they were added to streamline subsequent analysis. This decision is made to avoid redundant data extraction and transformations from the `weekly_sales` table in later queries, particularly when calculating metrics that rely on the 'sales' data. Given that the original weekly_sales table lacks a primary key, which could have facilitated efficient joins, two main approaches were considered:
1. Adding a primary key to the clean_weekly_sales table, either by identifying a unique combination of columns or creating a surrogate key;
2. Including the `sales`, `platform` and `transactions` columns directly. Opting for the latter, despite slightly deviating from the original task, simplifies the overall analysis workflow and ensures data consistency.

***Solution:***
```SQL
CREATE TABLE IF NOT EXISTS data_mart.clean_weekly_sales (
  "week_date" DATE,
  "week_number" INTEGER,
  "month_number" INTEGER,
  "calendar_year"  INTEGER,
  "platform" VARCHAR(7),
  "segment" VARCHAR(7),
  "age_band" VARCHAR(12),
  "demographic" VARCHAR(8),
  "sales" NUMERIC(15, 2),
  "transactions" INTEGER,
  "avg_transaction" NUMERIC(5, 2)
  );
INSERT INTO data_mart.clean_weekly_sales
SELECT
  week_date,
  EXTRACT('week' FROM week_date) AS week_number,
  EXTRACT('month' FROM week_date) AS month_number,
  EXTRACT('year' FROM week_date) AS calendar_year,
  platform,
  segment,
  CASE
    WHEN age_band = '1' THEN 'Young Adults'
    ELSE 
      CASE
        WHEN age_band = '2' THEN 'Middle Aged'
        ELSE 
          CASE
            WHEN age_band = 'unknown' THEN age_band
            ELSE 'Retirees'
          END
      END
  END AS age_band,
  CASE
    WHEN demographic = 'unknown' THEN demographic
    ELSE
      CASE
        WHEN demographic = 'C' THEN 'Couples'
        ELSE 'Families'
      END
  END AS demographic,
  sales,
  transactions,
  ROUND(sales / transactions, 2) AS avg_transaction 
FROM (
  SELECT
    to_date(week_date, 'DD/MM/YY') AS week_date,
    platform,
    CASE
      WHEN segment = 'null' THEN 'unknown'
      ELSE segment
    END AS segment,
    CASE
      WHEN segment = 'null' THEN 'unknown'
      ELSE RIGHT(segment, 1)
    END AS age_band,
    CASE
      WHEN segment = 'null' THEN 'unknown'
      ELSE LEFT(segment, 1)
    END AS demographic,
    transactions,
    sales
  FROM
    weekly_sales
  ) sub
;

SELECT * FROM clean_weekly_sales LIMIT 10;
```

<details>
  <summary><em><strong>show description</strong></em></summary>

- Subquery `sub`:  
  - Converts `week_date` from `VARCHAR` to `DATE` using `TO_DATE()`.  
  - Replaces `'null'` values in `segment` with `'unknown'`.  
  - Extracts the last character of `segment` as `age_band`.  
  - Extracts the first character of `segment` as `demographic`.  

- Main `SELECT` Statement:  
  - Extracts `week_number`, `month_number`, and `calendar_year` using `EXTRACT()`.  
  - Maps `age_band` based on predefined categories:  
    - `'1'` → `'Young Adults'`  
    - `'2'` → `'Middle Aged'`  
    - `'3'` or `'4'` → `'Retirees'`  
    - `'unknown'` remains unchanged.  
  - Maps `demographic` values:  
    - `'C'` → `'Couples'`  
    - `'F'` → `'Families'`  
    - `'unknown'` remains unchanged.  
  - Calculates `avg_transaction` as `sales / transactions`, rounded to 2 decimal places.  

- `Final Step`:  
  - Inserts transformed data into `data_mart.clean_weekly_sales`.  
  - Ensures structured and cleaned dataset for further analysis.  

</details>

***Result table:***

| week_date  | week_number | month_number | calendar_year | platform | segment | age_band     | demographic | sales       | transactions | avg_transaction |
| ---------- | ----------- | ------------ | ------------- | -------- | ------- | ------------ | ----------- | ----------- | ------------ | --------------- |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | C3      | Retirees     | Couples     | 3656163.00  | 120631       | 30.00           |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | F1      | Young Adults | Families    | 996575.00   | 31574        | 31.00           |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | unknown | unknown      | unknown     | 16509610.00 | 529151       | 31.00           |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | C1      | Young Adults | Couples     | 141942.00   | 4517         | 31.00           |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | C2      | Middle Aged  | Couples     | 1758388.00  | 58046        | 30.00           |
| 2020-08-31 | 36          | 8            | 2020          | Shopify  | F2      | Middle Aged  | Families    | 243878.00   | 1336         | 182.00          |
| 2020-08-31 | 36          | 8            | 2020          | Shopify  | F3      | Retirees     | Families    | 519502.00   | 2514         | 206.00          |
| 2020-08-31 | 36          | 8            | 2020          | Shopify  | F1      | Young Adults | Families    | 371417.00   | 2158         | 172.00          |
| 2020-08-31 | 36          | 8            | 2020          | Shopify  | F2      | Middle Aged  | Families    | 49557.00    | 318          | 155.00          |
| 2020-08-31 | 36          | 8            | 2020          | Retail   | C3      | Retirees     | Couples     | 3888162.00  | 111032       | 35.00           |

---

### B. Data Exploration
#### 1. What day of the week is used for each `week_date` value?

***query:***
```SQL
SELECT DISTINCT
  TO_CHAR(week_date, 'Day') AS week_day
FROM
  clean_weekly_sales;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query retrieves the distinct day names from the `week_date` column in the `clean_weekly_sales` table.

- `TO_CHAR(week_date, 'Day')`: Converts the `week_date` values into their corresponding weekday names.
- `DISTINCT`: Ensures that only unique weekday names appear in the result.

</details>

***answer:***
| week_day |
| ---------|
| Monday   |

---

#### 2. What range of week numbers are missing from the dataset?

***query:***
```SQL
WITH week_numbers AS (
  SELECT
    *
  FROM
    generate_series(
      (SELECT MIN(week_number) FROM clean_weekly_sales),
      (SELECT MAX(week_number) FROM clean_weekly_sales)
    ) AS week_number
  )

SELECT
  wn.week_number
FROM
  week_numbers wn
LEFT JOIN 
  clean_weekly_sales cws USING (week_number)
WHERE cws.week_number IS NULL;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query identifies week numbers that are missing from the `clean_weekly_sales` table within the range of existing week numbers.

-   `WITH week_numbers AS (...)`: Generates a series of all week numbers between the minimum and maximum week numbers found in the `clean_weekly_sales` table.
-   `generate_series( (SELECT MIN(week_number) FROM clean_weekly_sales), (SELECT MAX(week_number) FROM clean_weekly_sales) )`: Creates a sequential list of integers from the minimum to the maximum `week_number` values.
-   `LEFT JOIN clean_weekly_sales cws USING (week_number)`: Performs a left join between the generated week numbers and the `clean_weekly_sales` table, keeping all generated week numbers.
-   `WHERE cws.week_number IS NULL`: Filters the results to include only those generated week numbers that do not have a corresponding entry in the `clean_weekly_sales` table, indicating missing week numbers.

</details>

***answer:***
| week_number |
| ------------|
|             |

***\*The query returned an empty table, no week numbers are missing***

---

#### 3. How many total transactions were there for each year in the dataset?

***query:***
```SQL
SELECT
  calendar_year,
  COUNT(avg_transaction) AS transactions_amount
FROM
  clean_weekly_sales
GROUP BY
  calendar_year
ORDER BY
  calendar_year;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the total number of transactions for each year in the `clean_weekly_sales` dataset.

-   `SELECT calendar_year, COUNT(avg_transaction) AS transactions_amount`: Selects the calendar year and counts the number of transactions, aliasing the count as `transactions_amount`.
-   `COUNT(avg_transaction)`: Counts the number of non-NULL values in the `avg_transaction` column, effectively counting the number of transactions.
-   `FROM clean_weekly_sales`: Specifies the table from which to retrieve the data.
-   `GROUP BY calendar_year`: Groups the results by calendar year, so the count is performed for each year separately.
-   `ORDER BY calendar_year`: Orders the results by calendar year in ascending order.

</details>

***answer:***
| calendar_year | transactions_amount |
| ------------- | ------------------- |
| 2018          | 5698                |
| 2019          | 5708                |
| 2020          | 5711                |

---

#### 4. What is the total sales for each region for each month?

***query:***
```SQL
WITH short_weekly_sales AS (
  SELECT
    EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number,
    region,
    sales
  FROM
    weekly_sales
)

SELECT
  region,
  month_number,
  SUM(sales)::money AS total_sales
FROM
  short_weekly_sales
GROUP BY
  region,
  month_number
ORDER BY
  region,
  month_number;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the total sales for each region for each month, represented by its numerical value.

-   `WITH short_weekly_sales AS (...)`: Creates a common table expression (CTE) named `short_weekly_sales` to simplify the main query.
-   `EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number`: Extracts the month number from the `week_date` column, converting it to a date using the specified format.
-   `region, sales`: Selects the region and sales columns.
-   `SELECT region, month_number, SUM(sales) AS total_sales`: Selects the region, month number, and the sum of sales, aliasing the sum as `total_sales`.
-   `SUM(sales)`: Calculates the sum of the sales for each group.
-   `FROM short_weekly_sales`: Specifies the CTE as the data source.
-   `GROUP BY region, month_number`: Groups the results by region and month number.
-   `ORDER BY region, month_number`: Orders the results by region and month number in ascending order.

</details>

***answer:***
| region        | month_number | total_sales       |
| ------------- | ------------ | ----------------- |
| AFRICA        | 3            | $567,767,480.00   |
| AFRICA        | 4            | $1,911,783,504.00 |
| AFRICA        | 5            | $1,647,244,738.00 |
| AFRICA        | 6            | $1,767,559,760.00 |
| AFRICA        | 7            | $1,960,219,710.00 |
| AFRICA        | 8            | $1,809,596,890.00 |
| AFRICA        | 9            | $276,320,987.00   |
| ---           | ---          | ---               |
| USA           | 3            | $225,353,043.00   |
| USA           | 4            | $759,786,323.00   |
| USA           | 5            | $655,967,121.00   |
| USA           | 6            | $703,878,990.00   |
| USA           | 7            | $760,331,754.00   |
| USA           | 8            | $712,002,790.00   |
| USA           | 9            | $110,532,368.00   |

---

#### 5. What is the total count of transactions for each platform?

***query:***
```SQL
SELECT
 platform,
 COUNT(transactions) AS transactions_amount
FROM
  weekly_sales
GROUP BY
  platform;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query counts the number of transactions for each platform in the `weekly_sales` table.

-   `SELECT platform, COUNT(transactions) AS transactions_amount`: Selects the platform and counts the number of transactions, aliasing the count as `transactions_amount`.
-   `COUNT(transactions)`: Counts the number of non-NULL values in the `transactions` column, effectively counting the number of transactions for each platform.
-   `FROM weekly_sales`: Specifies the table from which to retrieve the data.
-   `GROUP BY platform`: Groups the results by platform, so the count is performed for each unique platform.

</details>

***answer:***
| platform | transactions_amount |
| -------- | ------------------- |
| Shopify  | 8549                |
| Retail   | 8568                |

---

#### 6. What is the percentage of sales for Retail vs Shopify for each month?

***query:***
```SQL
WITH total_month_sales AS (
  SELECT
    EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number,
    SUM(sales) AS total_sales
  FROM
    weekly_sales
  GROUP BY
    month_number
  ),
platform_month_sales AS (
  SELECT
    EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number,
    platform,
    SUM(sales) AS platform_sales
  FROM
    weekly_sales
  GROUP BY
    month_number,
    platform
)

SELECT
  pms.month_number,
  pms.platform,
  pms.platform_sales::money,
  tms.total_sales::money,
  ROUND((pms.platform_sales::numeric / tms.total_sales) * 100, 2) AS sales_percentage  
FROM
  platform_month_sales pms
JOIN
  total_month_sales tms USING(month_number)
ORDER BY
  pms.month_number,
  pms.platform
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the percentage of sales for each platform (Retail and Shopify) for each month, displaying sales values in money format.

-   `WITH total_month_sales AS (...)`: Creates a Common Table Expression (CTE) named `total_month_sales` to calculate the total sales for each month.
    -   `EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number`: Extracts the month number from the `week_date` column, converting it to a date using the specified format.
    -   `SUM(sales) AS total_sales`: Calculates the sum of sales for each month.
    -   `GROUP BY month_number`: Groups the results by month number.
-   `WITH platform_month_sales AS (...)`: Creates another CTE named `platform_month_sales` to calculate the total sales for each platform within each month.
    -   `EXTRACT('month' FROM to_date(week_date, 'DD/MM/YY')) AS month_number`: Extracts the month number from the `week_date` column.
    -   `platform`: Selects the platform.
    -   `SUM(sales) AS platform_sales`: Calculates the sum of sales for each platform within each month.
    -   `GROUP BY month_number, platform`: Groups the results by month number and platform.
-   `SELECT pms.month_number, pms.platform, pms.platform_sales::money, tms.total_sales::money, ROUND((pms.platform_sales::numeric / tms.total_sales) * 100, 2) AS sales_percentage`: Selects the month number, platform, platform sales (formatted as money), total month sales (formatted as money), and calculates the percentage of sales for each platform within each month.
    -   `pms.platform_sales::money` and `tms.total_sales::money`: Converts sales values to the money data type for display.
    -   `ROUND((pms.platform_sales::numeric / tms.total_sales) * 100, 2)`: Calculates the percentage of platform sales to total month sales and rounds the result to 2 decimal places.
-   `FROM platform_month_sales pms JOIN total_month_sales tms USING(month_number)`: Joins the two CTEs on the `month_number` column.
-   `ORDER BY pms.month_number, pms.platform`: Orders the results by month number and platform.

</details>

***answer:***
| month_number | platform | platform_sales    | total_sales       | sales_percentage |
| ------------ | -------- | ----------------- | ----------------- | ---------------- |
| 3            | Retail   | $2,299,188,417.00 | $2,357,168,735.00 | 97.54            |
| 3            | Shopify  | $57,980,318.00    | $2,357,168,735.00 | 2.46             |
| 4            | Retail   | $7,735,592,234.00 | $7,926,304,534.00 | 97.59            |
| 4            | Shopify  | $190,712,300.00   | $7,926,304,534.00 | 2.41             |
| 5            | Retail   | $6,585,838,223.00 | $6,768,263,125.00 | 97.30            |
| 5            | Shopify  | $182,424,902.00   | $6,768,263,125.00 | 2.70             |
| 6            | Retail   | $7,049,949,260.00 | $7,247,714,362.00 | 97.27            |
| 6            | Shopify  | $197,765,102.00   | $7,247,714,362.00 | 2.73             |
| 7            | Retail   | $7,688,091,448.00 | $7,902,330,809.00 | 97.29            |
| 7            | Shopify  | $214,239,361.00   | $7,902,330,809.00 | 2.71             |
| 8            | Retail   | $7,191,449,998.00 | $7,407,576,007.00 | 97.08            |
| 8            | Shopify  | $216,126,009.00   | $7,407,576,007.00 | 2.92             |
| 9            | Retail   | $1,104,506,857.00 | $1,134,276,655.00 | 97.38            |
| 9            | Shopify  | $29,769,798.00    | $1,134,276,655.00 | 2.62             |

---

#### 7. What is the percentage of sales by demographic for each year in the dataset?

***query:***
```SQL
WITH total_yearly_sales AS (
  SELECT
    calendar_year,
    SUM(sales) AS total_sales
  FROM
    clean_weekly_sales
  GROUP BY
    calendar_year
  ),
demographic_yearly_sales AS (
  SELECT
    calendar_year,
    demographic,
    SUM(sales) AS demographic_sales
  FROM
    clean_weekly_sales
  GROUP BY
    calendar_year,
    demographic
  )

SELECT
  dys.calendar_year,
  dys.demographic,
  dys.demographic_sales::money,
  tys.total_sales::money,
  ROUND((dys.demographic_sales::numeric / tys.total_sales) * 100, 2) AS sales_percentage  
FROM
  demographic_yearly_sales dys
JOIN
  total_yearly_sales tys USING(calendar_year)
ORDER BY
  dys.calendar_year,
  dys.demographic
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the percentage of sales by demographic for each year in the `clean_weekly_sales` dataset.

-   `WITH total_yearly_sales AS (...)`: Creates a Common Table Expression (CTE) named `total_yearly_sales` to calculate the total sales for each calendar year.
    -   `calendar_year`: Selects the calendar year.
    -   `SUM(sales) AS total_sales`: Calculates the sum of sales for each year.
    -   `GROUP BY calendar_year`: Groups the results by calendar year.
-   `WITH demographic_yearly_sales AS (...)`: Creates another CTE named `demographic_yearly_sales` to calculate the total sales for each demographic within each calendar year.
    -   `calendar_year, demographic`: Selects the calendar year and demographic.
    -   `SUM(sales) AS demographic_sales`: Calculates the sum of sales for each demographic within each year.
    -   `GROUP BY calendar_year, demographic`: Groups the results by calendar year and demographic.
-   `SELECT dys.calendar_year, dys.demographic, dys.demographic_sales::money, tys.total_sales::money, ROUND((dys.demographic_sales::numeric / tys.total_sales) * 100, 2) AS sales_percentage`: Selects the calendar year, demographic, demographic sales (formatted as money), total yearly sales (formatted as money), and calculates the percentage of sales for each demographic within each year.
    -   `dys.demographic_sales::money` and `tys.total_sales::money`: Converts sales values to the money data type for display.
    -   `ROUND((dys.demographic_sales::numeric / tys.total_sales) * 100, 2)`: Calculates the percentage of demographic sales to total yearly sales and rounds the result to 2 decimal places.
-   `FROM demographic_yearly_sales dys JOIN total_yearly_sales tys USING(calendar_year)`: Joins the two CTEs on the `calendar_year` column.
-   `ORDER BY dys.calendar_year, dys.demographic`: Orders the results by calendar year and demographic.

</details>

***answer:***

| calendar_year | demographic | demographic_sales | total_sales        | sales_percentage |
| ------------- | ----------- | ----------------- | ------------------ | ---------------- |
| 2018          | Couples     | $3,402,388,688.00 | $12,897,380,827.00 | 26.38            |
| 2018          | Families    | $4,125,558,033.00 | $12,897,380,827.00 | 31.99            |
| 2018          | unknown     | $5,369,434,106.00 | $12,897,380,827.00 | 41.63            |
| 2019          | Couples     | $3,749,251,935.00 | $13,746,032,500.00 | 27.28            |
| 2019          | Families    | $4,463,918,344.00 | $13,746,032,500.00 | 32.47            |
| 2019          | unknown     | $5,532,862,221.00 | $13,746,032,500.00 | 40.25            |
| 2020          | Couples     | $4,049,566,928.00 | $14,100,220,900.00 | 28.72            |
| 2020          | Families    | $4,614,338,065.00 | $14,100,220,900.00 | 32.73            |
| 2020          | unknown     | $5,436,315,907.00 | $14,100,220,900.00 | 38.55            |

---

#### 8. Which `age_band` and `demographic` values contribute the most to Retail sales?

***query:***
```SQL
SELECT
  age_band,
  demographic,
  SUM(sales)::money AS sales_per_group
FROM
  clean_weekly_sales
WHERE
  platform = 'Retail'
GROUP BY
  age_band,
  demographic
ORDER BY
  sales_per_group DESC;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the total sales for each combination of `age_band` and `demographic` for Retail sales, and orders the results by total sales in descending order.

-   `SELECT age_band, demographic, SUM(sales)::money AS sales_per_group`: Selects the `age_band`, `demographic`, and the sum of `sales` (formatted as money), aliasing the sum as `sales_per_group`.
-   `SUM(sales)::money`: Calculates the sum of sales and converts the result to the `money` data type.
-   `FROM clean_weekly_sales`: Specifies the `clean_weekly_sales` table as the data source.
-   `WHERE platform = 'Retail'`: Filters the results to include only records where the `platform` is 'Retail'.
-   `GROUP BY age_band, demographic`: Groups the results by `age_band` and `demographic`.
-   `ORDER BY sales_per_group DESC`: Orders the results by `sales_per_group` in descending order, showing the highest sales groups first.

</details>

***answer:***

| age_band     | demographic | sales_per_group    |
| ------------ | ----------- | ------------------ |
| unknown      | unknown     | $16,067,285,533.00 |
| Retirees     | Families    | $6,634,686,916.00  |
| Retirees     | Couples     | $6,370,580,014.00  |
| Middle Aged  | Families    | $4,354,091,554.00  |
| Young Adults | Couples     | $2,602,922,797.00  |
| Middle Aged  | Couples     | $1,854,160,330.00  |
| Young Adults | Families    | $1,770,889,293.00  |

---

#### 9. Can we use the `avg_transaction` column to find the average transaction size for each year for *Retail* vs *Shopify*? If not - how would you calculate it instead?

***answer:***

The `avg_transaction` column represents the average transaction value at the individual record level, calculated as `sales` / `transactions`, as per the initial data transformation requirements.

We must understand that this column does not reflect the aggregated average transaction size for each year, segmented by Retail and Shopify platforms. To calculate the latter, we need to aggregate sales and transactions by year and platform, then divide the total sales by the total transactions for each group. This is because avg_transaction is a record-level calculation, whereas the required average is a group-level calculation.

**Average transaction size for each year for Retail vs Shopify calculation:**

***query:***
```SQL

SELECT
    calendar_year,
    platform,
    ROUND(SUM(sales) / SUM(transactions), 2)::money AS average_transaction_size
FROM
    clean_weekly_sales
GROUP BY
    calendar_year,
    platform
ORDER BY
    calendar_year,
    platform;

```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the average transaction size for each year for Retail vs Shopify, displaying the result in money format.

-   `SELECT calendar_year, platform, ROUND(SUM(sales) / SUM(transactions), 2)::money AS average_transaction_size`: Selects the calendar year, platform, and calculates the average transaction size, aliasing it as `average_transaction_size`.
    -   `SUM(sales) / SUM(transactions)`: Calculates the average transaction size by dividing the sum of sales by the sum of transactions.
    -   `ROUND(..., 2)`: Rounds the result to two decimal places.
    -   `::money`: Converts the rounded result to the money data type for display.
-   `FROM clean_weekly_sales`: Specifies the `clean_weekly_sales` table as the data source.
-   `GROUP BY calendar_year, platform`: Groups the results by calendar year and platform.
-   `ORDER BY calendar_year, platform`: Orders the results by calendar year and platform in ascending order.

</details>

***Result table:***

| calendar_year | platform | average_transaction_size |
| ------------- | -------- | ------------------------ |
| 2018          | Retail   | $36.56                   |
| 2018          | Shopify  | $192.48                  |
| 2019          | Retail   | $36.83                   |
| 2019          | Shopify  | $183.36                  |
| 2020          | Retail   | $36.56                   |
| 2020          | Shopify  | $179.03                  |

---

### C. Before & After Analysis

> For this task we will take the `week_date` value of **2020-06-15** as the baseline week where the Data Mart sustainable packaging changes came into effect.
>
> We would include all `week_date` values for **2020-06-15** as the start of the period **after** the change and the previous `week_date` values would be **before**

#### 1. Determine the total sales and the growth/reduction rate (in actual values and percentage) before and after 2020-06-15 for:
####   a) 4 weeks before and after the specified date.

***query:***
```SQL
WITH total_sales_counting AS (
  SELECT
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') - INTERVAL '4 weeks' AND to_date('2020-06-14', 'YYYY-MM-DD')
    ) AS total_sales_before,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') AND to_date('2020-06-15', 'YYYY-MM-DD') + INTERVAL '4 weeks'
    ) AS total_sales_after
  ),
sales_difference AS (
  SELECT
    total_sales_before,
    total_sales_after,
    CASE
      WHEN total_sales_after > total_sales_before THEN 'growth'
      ELSE 'reduction'
    END AS rate_type,
      total_sales_after - total_sales_before AS rate
  FROM total_sales_counting
  )
SELECT
  total_sales_before::money,
  total_sales_after::money,
  rate_type,
  rate::money,
  ROUND((rate::numeric / total_sales_before) * 100, 2) AS percentage_rate
FROM sales_difference;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query calculates the total sales for the 4 weeks before and after '2020-06-15', determines the growth or reduction, and calculates the percentage change.

-   `WITH total_sales_counting AS (...)`: Creates a Common Table Expression (CTE) named `total_sales_counting` to calculate the total sales for the periods before and after '2020-06-15'.
    -   `(SELECT SUM(sales) FROM clean_weekly_sales WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') - INTERVAL '4 weeks' AND to_date('2020-06-14', 'YYYY-MM-DD')) AS total_sales_before`: Calculates the sum of sales for the 4 weeks before '2020-06-15'.
    -   `(SELECT SUM(sales) FROM clean_weekly_sales WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') AND to_date('2020-06-15', 'YYYY-MM-DD') + INTERVAL '4 weeks') AS total_sales_after`: Calculates the sum of sales for the 4 weeks including and after '2020-06-15'.
-   `WITH sales_difference AS (...)`: Creates another CTE named `sales_difference` to calculate the difference in sales and determine the type of change (growth or reduction).
    -   `total_sales_before, total_sales_after`: Selects the calculated total sales.
    -   `CASE WHEN total_sales_after > total_sales_before THEN 'growth' ELSE 'reduction' END AS rate_type`: Determines the type of change based on the sales difference.
    -   `total_sales_after - total_sales_before AS rate`: Calculates the difference in sales.
-   `SELECT total_sales_before::money, total_sales_after::money, rate_type, rate::money, ROUND((rate::numeric / total_sales_before) * 100, 2) AS percentage_rate`: Selects the total sales (formatted as money), the type of change, the difference in sales (formatted as money), and calculates the percentage change.
    -   `total_sales_before::money, total_sales_after::money, rate::money`: Converts sales values to the money data type for display.
    -   `ROUND((rate::numeric / total_sales_before) * 100, 2) AS percentage_rate`: Calculates the percentage change and rounds the result to 2 decimal places.
-   `FROM sales_difference`: Specifies the `sales_difference` CTE as the data source.

</details>

***answer:***

| total_sales_before | total_sales_after | rate_type | rate            | percentage_rate |
| ------------------ | ----------------- | --------- | --------------- | --------------- |
| $2,345,878,357.00  | $2,904,930,571.00 | growth    | $559,052,214.00 | 23.83           |

---

####   b) 12 weeks before and after the specified date.

**To answer sub-question (b), the query from the previous solution is used, with INTERVAL '4 weeks' replaced by INTERVAL '12 weeks' in both subqueries.**

***answer:***

| total_sales_before | total_sales_after | rate_type | rate             | percentage_rate |
| ------------------ | ----------------- | --------- | ---------------- | --------------- |
| $7,126,273,147.00  | $6,973,947,753.00 | reduction | -$152,325,394.00 | -2.14           |

---

#### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
####   a) 4 weeks before and after the specified date.

***query:***
```SQL
WITH total_2018_sales_counting AS (
  SELECT
    2018 AS calendar_year,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2018-06-15', 'YYYY-MM-DD') - INTERVAL '4 weeks' AND to_date('2018-06-    14', 'YYYY-MM-DD')
    ) AS total_sales_before,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2018-06-15', 'YYYY-MM-DD') AND to_date('2018-06-15', 'YYYY-MM-DD') + INTERVAL '4 weeks'
    ) AS total_sales_after
  ),
total_2019_sales_counting AS (
  SELECT
    2019 AS calendar_year,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2019-06-15', 'YYYY-MM-DD') - INTERVAL '4 weeks' AND to_date('2019-06-    14', 'YYYY-MM-DD')
    ) AS total_sales_before,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2019-06-15', 'YYYY-MM-DD') AND to_date('2019-06-15', 'YYYY-MM-DD') + INTERVAL '4 weeks'
    ) AS total_sales_after
  ),
total_2020_sales_counting AS (
  SELECT
    2020 AS calendar_year,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') - INTERVAL '4 weeks' AND to_date('2020-06-    14', 'YYYY-MM-DD')
    ) AS total_sales_before,
    (SELECT SUM(sales)
     FROM clean_weekly_sales
     WHERE week_date BETWEEN to_date('2020-06-15', 'YYYY-MM-DD') AND to_date('2020-06-15', 'YYYY-MM-DD') + INTERVAL '4 weeks'
    ) AS total_sales_after
  ),
sales_difference AS (
  SELECT
    calendar_year,
    total_sales_before,
    total_sales_after,
    CASE
      WHEN total_sales_after > total_sales_before THEN 'growth'
      ELSE 'reduction'
    END AS rate_type,
      total_sales_after - total_sales_before AS rate
  FROM (
    SELECT *
    FROM total_2018_sales_counting
    UNION ALL
    SELECT *
    FROM total_2019_sales_counting
    UNION ALL
    SELECT *
    FROM total_2020_sales_counting
  ) sub
)

SELECT
  calendar_year,
  total_sales_before::money,
  total_sales_after::money,
  rate_type,
  rate::money,
  ROUND((rate::numeric / total_sales_before) * 100, 2) AS percentage_rate
FROM sales_difference;
```

<details>
  <summary><em><strong>show description:</strong></em></summary>

The SQL query compares total sales for 4-week periods before and after June 15th across three consecutive years (2018, 2019, 2020) to determine sales growth or reduction.

-   `WITH total_2018_sales_counting AS (...)`, `total_2019_sales_counting AS (...)`, `total_2020_sales_counting AS (...)`: Three Common Table Expressions (CTEs) are created to calculate the total sales for the 4 weeks before and after June 15th for each year.
    -   Each CTE calculates `total_sales_before` and `total_sales_after` by summing the `sales` column from the `clean_weekly_sales` table, filtering by `week_date` using `BETWEEN` and `INTERVAL` to define the 4-week periods.
    -   The `calendar_year` is added as a constant to identify the year for each set of results.
-   `sales_difference AS (...)`: A CTE named `sales_difference` is created to calculate the difference in sales and determine the type of change (growth or reduction).
    -   It selects `calendar_year`, `total_sales_before`, and `total_sales_after` from the combined results of the previous CTEs (using `UNION ALL`).
    -   A `CASE` statement determines `rate_type` as 'growth' if `total_sales_after` is greater than `total_sales_before`, or 'reduction' otherwise.
    -   `rate` is calculated as the difference between `total_sales_after` and `total_sales_before`.
-   `SELECT total_sales_before::money, total_sales_after::money, rate_type, rate::money, ROUND((rate::numeric / total_sales_before) * 100, 2) AS percentage_rate`: The final `SELECT` statement retrieves and formats the results.
    -   `total_sales_before` and `total_sales_after` are converted to the `money` data type.
    -   `rate` is also converted to `money`.
    -   `percentage_rate` is calculated as the percentage change in sales and rounded to 2 decimal places.
-   `FROM sales_difference`: The data source for the final `SELECT` statement is the `sales_difference` CTE.

The query effectively compares sales performance across years, highlighting both the absolute and percentage changes in sales before and after the specified date.

</details>

***answer:***

| calendar_year | total_sales_before | total_sales_after | rate_type | rate            | percentage_rate |
| ------------- | ------------------ | ----------------- | --------- | --------------- | --------------- |
| 2018          | $2,125,140,809.00  | $2,129,242,914.00 | growth    | $4,102,105.00   | 0.19            |
| 2019          | $2,249,989,796.00  | $2,252,326,390.00 | growth    | $2,336,594.00   | 0.10            |
| 2020          | $2,345,878,357.00  | $2,904,930,571.00 | growth    | $559,052,214.00 | 23.83           |

---

####   b) 12 weeks before and after the specified date.
**Similar to the previous question, the changes will only affect the time interval, which will be increased from 4 to 12 weeks in both subqueries.**

***answer:***

| calendar_year | total_sales_before | total_sales_after | rate_type | rate             | percentage_rate |
| ------------- | ------------------ | ----------------- | --------- | ---------------- | --------------- |
| 2018          | $6,396,562,317.00  | $6,500,818,510.00 | growth    | $104,256,193.00  | 1.63            |
| 2019          | $6,883,386,397.00  | $6,862,646,103.00 | reduction | -$20,740,294.00  | -0.30           |
| 2020          | $7,126,273,147.00  | $6,973,947,753.00 | reduction | -$152,325,394.00 | -2.14           |

>***Before & After Analysis Summary:***
>
>In June 2020, Data Mart implemented large-scale supply chain changes, transitioning to sustainable packaging. To assess the impact of these changes on sales, an analysis of sales for 4 and 12 weeks before and after June 15, 2020, was conducted, along with a comparison with similar periods in 2018 and 2019.
>
>The 4-week sales analysis revealed a significant sales increase in 2020 of 23.83%, equivalent to $559,052,214. This contrasts sharply with the marginal growth in 2018 and 2019 (0.19% and 0.10%, respectively). This may indicate a positive short-term impact of the shift to sustainable packaging.
>
>However, the 12-week sales analysis showed a sales reduction in 2020 of 2.14%, amounting to -$152,325,394. This also differs from the results of 2018 and 2019, where 2018 saw a growth of 1.63% and 2019 saw a reduction of 0.30%. This may suggest that the short-term positive effect was not sustained in the long term.
>
>For a more comprehensive understanding of the impact of the changes, further analysis by platform, region, segment, and customer type is necessary, as outlined in Danny's key questions. This will identify which business areas were most affected and develop recommendations to minimize negative impacts when introducing similar changes in the future."

---

### D. Bonus Question

---

## Conclusion

---