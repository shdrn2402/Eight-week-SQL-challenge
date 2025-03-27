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
SET search_path = clique_bait;

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
