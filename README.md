# Retail Sales Analysis SQL Project

## Project Overview

This comprehensive SQL project demonstrates advanced data analysis techniques for retail sales data. It encompasses database design, data cleaning, exploratory data analysis (EDA), and business intelligence reporting using SQL queries.

## Objectives

1. **Database Implementation**: Design and implement a retail sales database schema
2. **Data Quality Management**: Implement robust data cleaning procedures
3. **Advanced Analytics**: Perform in-depth exploratory data analysis
4. **Business Intelligence**: Generate actionable insights through SQL-driven analysis

## Database Architecture

### Database Setup

First, create a dedicated database for the retail analysis:

```sql
CREATE DATABASE retail_db;
\c retail_db
```

### Schema Design

The `retail_sales` table is designed to capture comprehensive transaction data:

```sql
CREATE TABLE retail_sales (
    transaction_id INTEGER PRIMARY KEY,
    sale_date DATE NOT NULL,
    sale_time TIME NOT NULL,
    customer_id INTEGER NOT NULL,
    gender VARCHAR(10) CHECK (gender IN ('Male', 'Female', 'Other')),
    age INTEGER CHECK (age > 0),
    category VARCHAR(50) NOT NULL,
    quantity INTEGER CHECK (quantity > 0),
    price_per_unit DECIMAL(10,2) CHECK (price_per_unit > 0),
    cogs DECIMAL(10,2) CHECK (cogs > 0),
    total_sales DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price_per_unit) STORED,
    profit DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price_per_unit - cogs) STORED
);

-- Create indexes for improved query performance
CREATE INDEX idx_sale_date ON retail_sales(sale_date);
CREATE INDEX idx_customer_id ON retail_sales(customer_id);
CREATE INDEX idx_category ON retail_sales(category);
```

## Data Quality Management

### Initial Data Assessment

```sql
-- Check for total records
SELECT COUNT(*) AS total_records FROM retail_sales;

-- Verify distinct values in key columns
SELECT 
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(DISTINCT category) AS unique_categories,
    COUNT(DISTINCT sale_date) AS unique_dates
FROM retail_sales;

-- Display category distribution
SELECT 
    category,
    COUNT(*) AS transaction_count,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) AS percentage
FROM retail_sales
GROUP BY category
ORDER BY transaction_count DESC;
```

### Data Cleaning

```sql
-- Identify records with data quality issues
WITH data_quality_check AS (
    SELECT 
        transaction_id,
        CASE 
            WHEN sale_date IS NULL THEN 'Missing sale date'
            WHEN sale_date > CURRENT_DATE THEN 'Future date'
            WHEN age NOT BETWEEN 18 AND 100 THEN 'Invalid age'
            WHEN quantity <= 0 THEN 'Invalid quantity'
            WHEN price_per_unit <= 0 THEN 'Invalid price'
            WHEN cogs <= 0 THEN 'Invalid COGS'
            ELSE 'Valid'
        END AS quality_status
    FROM retail_sales
)
SELECT 
    quality_status,
    COUNT(*) AS record_count
FROM data_quality_check
GROUP BY quality_status
ORDER BY record_count DESC;

-- Remove or fix problematic records
DELETE FROM retail_sales
WHERE 
    sale_date IS NULL 
    OR sale_date > CURRENT_DATE
    OR age NOT BETWEEN 18 AND 100
    OR quantity <= 0
    OR price_per_unit <= 0
    OR cogs <= 0;
```

## Advanced Analytics

### Sales Performance Analysis

```sql
-- Monthly sales trends with year-over-year comparison
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', sale_date) AS sales_month,
        SUM(total_sales) AS total_sales,
        COUNT(DISTINCT customer_id) AS unique_customers,
        SUM(quantity) AS units_sold
    FROM retail_sales
    GROUP BY DATE_TRUNC('month', sale_date)
)
SELECT 
    sales_month,
    total_sales,
    unique_customers,
    units_sold,
    LAG(total_sales) OVER (ORDER BY sales_month) AS prev_month_sales,
    ROUND((total_sales - LAG(total_sales) OVER (ORDER BY sales_month)) * 100.0 / 
        LAG(total_sales) OVER (ORDER BY sales_month), 2) AS month_over_month_growth
FROM monthly_sales
ORDER BY sales_month;

-- Category performance matrix
SELECT 
    category,
    COUNT(*) AS transaction_count,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(quantity) AS total_units,
    ROUND(AVG(quantity), 2) AS avg_units_per_transaction,
    SUM(total_sales) AS total_revenue,
    ROUND(AVG(total_sales), 2) AS avg_transaction_value,
    SUM(profit) AS total_profit,
    ROUND(SUM(profit) * 100.0 / SUM(total_sales), 2) AS profit_margin
FROM retail_sales
GROUP BY category
ORDER BY total_revenue DESC;
```

### Customer Segmentation

```sql
-- RFM (Recency, Frequency, Monetary) Analysis
WITH customer_metrics AS (
    SELECT 
        customer_id,
        MAX(sale_date) AS last_purchase_date,
        COUNT(*) AS frequency,
        SUM(total_sales) AS monetary
    FROM retail_sales
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT 
        customer_id,
        NTILE(5) OVER (ORDER BY last_purchase_date DESC) AS recency_score,
        NTILE(5) OVER (ORDER BY frequency) AS frequency_score,
        NTILE(5) OVER (ORDER BY monetary) AS monetary_score
    FROM customer_metrics
)
SELECT 
    customer_id,
    recency_score,
    frequency_score,
    monetary_score,
    CASE 
        WHEN (recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4) THEN 'Champion'
        WHEN (recency_score >= 3 AND frequency_score >= 3 AND monetary_score >= 3) THEN 'Loyal Customer'
        WHEN (recency_score <= 2 AND frequency_score <= 2 AND monetary_score <= 2) THEN 'At Risk'
        ELSE 'Average'
    END AS customer_segment
FROM rfm_scores;
```

### Time-Based Analysis

```sql
-- Sales by time of day with profitability metrics
WITH time_periods AS (
    SELECT 
        transaction_id,
        CASE 
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) < 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS time_period,
        total_sales,
        profit
    FROM retail_sales
)
SELECT 
    time_period,
    COUNT(*) AS transaction_count,
    SUM(total_sales) AS total_revenue,
    ROUND(AVG(total_sales), 2) AS avg_transaction_value,
    SUM(profit) AS total_profit,
    ROUND(SUM(profit) * 100.0 / SUM(total_sales), 2) AS profit_margin
FROM time_periods
GROUP BY time_period
ORDER BY transaction_count DESC;
```

## Business Insights

1. **Sales Patterns**:
   - Analysis of peak sales periods
   - Category performance trends
   - Customer purchase behavior

2. **Customer Analysis**:
   - Demographics and segmentation
   - Purchase frequency patterns
   - Customer lifetime value

3. **Operational Efficiency**:
   - Time-based sales analysis
   - Inventory turnover rates
   - Profit margin analysis

## Recommendations

1. **Inventory Management**:
   - Stock optimization based on category performance
   - Time-based inventory allocation

2. **Customer Engagement**:
   - Targeted marketing based on RFM segments
   - Category-specific promotions

3. **Operational Improvements**:
   - Staffing optimization based on peak hours
   - Category-specific profit optimization

## Conclusion

This SQL project provides a comprehensive framework for retail sales analysis, enabling data-driven decision-making through robust database design, advanced analytics, and actionable business insights.
