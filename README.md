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
-- Step 1: Identify each customer's first order month (cohort)
WITH first_order AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(created_at)) AS first_order_month
    FROM sales
    GROUP BY customer_id
),

-- Step 2: Get monthly order activity
clean_sales AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', created_at) AS order_month
    FROM sales
)

-- Step 3: Join and count active customers per cohort per month
SELECT 
    first_order_month AS cohort_month,
    order_month,
    COUNT(DISTINCT customer_id) AS active_customers
FROM (
    SELECT
        cs.customer_id,
        cs.order_month,
        fo.first_order_month
    FROM clean_sales cs
    LEFT JOIN first_order fo ON cs.customer_id = fo.customer_id
) x
GROUP BY 1, 2
ORDER BY 1, 2;
```

**Visualization in Metabase**


### 2. Retention Analysis (Dec 2010 → Jan 2011)

```sql
-- Customers active in December 2010
WITH active_dec AS (
    SELECT 
        customer_id,
        COUNT(id) AS dec_orders
    FROM sales
    WHERE created_at BETWEEN '2010-12-01' AND '2010-12-31 23:59:59'
    GROUP BY customer_id
),

-- Customers active in January 2011
active_jan AS (
    SELECT 
        customer_id,
        COUNT(id) AS jan_orders
    FROM sales
    WHERE created_at BETWEEN '2011-01-01' AND '2011-01-31 23:59:59'
    GROUP BY customer_id
),

-- Retained = present in both months
retained AS (
    SELECT
        ad.customer_id,
        ad.dec_orders,
        aj.jan_orders
    FROM active_dec ad
    INNER JOIN active_jan aj ON ad.customer_id = aj.customer_id
)

SELECT
    COUNT(*) AS retained_customers,
    COUNT(*)::float / (SELECT COUNT(*) FROM active_dec) * 100 AS retention_rate_pct
FROM retained;
```

### 3. Churn Analysis (Dec 2010 → Jan 2011)

```sql
-- Customers active in December 2010
WITH active_dec AS (
    SELECT 
        customer_id,
        COUNT(id) AS dec_orders
    FROM sales
    WHERE created_at BETWEEN '2010-12-01' AND '2010-12-31 23:59:59'
    GROUP BY customer_id
),

-- Customers active in January 2011
active_jan AS (
    SELECT 
        customer_id
    FROM sales
    WHERE created_at BETWEEN '2011-01-01' AND '2011-01-31 23:59:59'
    GROUP BY customer_id
)

-- Churned = active in Dec but NOT in Jan
SELECT
    COUNT(ad.customer_id) AS churned_customers,
    COUNT(ad.customer_id)::float / (SELECT COUNT(*) FROM active_dec) * 100 AS churn_rate_pct
FROM active_dec ad
LEFT JOIN active_jan aj ON ad.customer_id = aj.customer_id
WHERE aj.customer_id IS NULL;
``
