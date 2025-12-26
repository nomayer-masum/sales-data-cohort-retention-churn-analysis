# Sales Data Cohort, Retention & Churn Analysis

This project demonstrates how to perform **Cohort Analysis**, **Customer Retention Rate**, and **Customer Churn Analysis** using SQL on a sales dataset. The analysis was performed by connecting a **Supabase (PostgreSQL)** database to **Metabase** for query execution and visualization.

## Tech Stack

- **Database**: Supabase (PostgreSQL)
- **Analytics & Visualization Tool**: Metabase
- **Language**: SQL
- **Analysis Types**: 
  - Monthly Cohort Analysis
  - Customer Retention Analysis
  - Customer Churn Analysis

## Dataset Overview

**Table**: `sales`

| Column       | Description                        | Type         |
|--------------|------------------------------------|--------------|
| `id`         | Unique order ID                    | Integer      |
| `customer_id`| Unique customer identifier         | Integer      |
| `created_at` | Order creation timestamp           | Timestamp    |

## Dataset Link
https://drive.google.com/file/d/1_CyLkI5J3wOXmnlaHP3Fha8XMv_n24j4/view?usp=sharing

## Project Structure

1. **Cohort Analysis** (Monthly)  
   → Group customers by their first purchase month and track their activity in subsequent months

2. **Retention Analysis**  
   → Customers active in Dec 2010 who remained active in Jan 2011

3. **Churn Analysis**  
   → Customers active in Dec 2010 but inactive in Jan 2011

## SQL Queries

### 1. Monthly Cohort Analysis

```sql
-- Step 1️: Find each customer's first order month (their cohort)
WITH first_order AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(created_at)) AS first_order_month
    FROM sales
    GROUP BY customer_id
),


-- Step 2️: Clean sales table with monthly order info
clean_sales AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', created_at) AS order_month
    FROM sales
)

  
SELECT first_order_month, order_month, count(distinct customer_id)
FROM
  (
    SELECT
      order_month,
      clean_sales.customer_id AS customer_id,
      first_order_month
    FROM
      clean_sales
      LEFT JOIN first_order ON clean_sales.customer_id = first_order.customer_id
  ) x

group by 1, 2;
```

**Visualization in Metabase**
![Cohort Pivot](https://github.com/nomayer-masum/sales-data-cohort-retention-churn-analysis/blob/main/Cohort%20Pivot.png)

### 2. Retention Analysis (Dec 2010 → Jan 2011)

```sql
-- Customers active in December 2010
WITH active_month AS (
    SELECT 
        customer_id, 
        COUNT(id) AS orders
    FROM sales
    WHERE created_at BETWEEN '2010-12-01' AND '2010-12-31'
    GROUP BY 1
),

-- Customers active in January 2011
Inactive_month AS (
    SELECT 
        customer_id, 
        COUNT(id) AS orders
    FROM sales
    WHERE created_at BETWEEN '2011-01-01' AND '2011-01-31'
    GROUP BY 1
),

-- Retained customers (present in both months)
final AS (
    SELECT 
        am.customer_id AS active_month_customers,
        am.orders AS active_month_orders,
        im.customer_id AS inactive_month_customers,
        im.orders AS inactive_month_orders
    FROM active_month am
    LEFT JOIN Inactive_month im 
        ON am.customer_id = im.customer_id
    WHERE im.customer_id IS NOT NULL
)

-- Total retained customers
SELECT
    COUNT(active_month_customers) AS Total_customers_of_Dec,
    COUNT(inactive_month_customers) AS Total_customers_of_Jan
FROM final;
```

### 3. Churn Analysis (Dec 2010 → Jan 2011)

```sql
-- Customers active in December 2010
WITH active_month AS (
    SELECT 
        customer_id, 
        COUNT(id) AS orders
    FROM sales
    WHERE created_at BETWEEN '2010-12-01' AND '2010-12-31'
    GROUP BY 1
),

-- Customers active in January 2011
Inactive_month AS (
    SELECT 
        customer_id, 
        COUNT(id) AS orders
    FROM sales
    WHERE created_at BETWEEN '2011-01-01' AND '2011-01-31'
    GROUP BY 1
),

-- Churned customers (missing in January)
final AS (
    SELECT 
        am.customer_id AS active_month_customers,
        am.orders AS active_month_orders,
        im.customer_id AS inactive_month_customers,
        im.orders AS inactive_month_orders
    FROM active_month am
    LEFT JOIN Inactive_month im 
        ON am.customer_id = im.customer_id
    WHERE im.customer_id IS NULL
)

-- Total churned customers
SELECT
    COUNT(active_month_customers) AS Total_customers_of_Dec,
    COUNT(inactive_month_customers) AS Total_customers_of_Jan
FROM final;

