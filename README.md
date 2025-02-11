# Retail Sales Analysis SQL Project

## Project Overview

This project demonstrates SQL skills for exploring, cleaning, and analyzing retail sales data. It involves setting up a database, performing exploratory data analysis (EDA), and answering business questions through SQL queries.

---

## Objectives

1. **Database Setup**: Create and populate a retail sales database.
2. **Data Cleaning**: Identify and remove records with missing values.
3. **Exploratory Data Analysis (EDA)**: Understand sales data distribution and trends.
4. **Business Analysis**: Answer specific business questions using SQL queries.

---

## Project Structure

### 1. Database Setup

#### Database & Table Creation
```sql
CREATE DATABASE retail_db;

CREATE TABLE retail_sales (
    transaction_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(10),
    age INT,
    category VARCHAR(50),
    quantity INT,
    price_per_unit DECIMAL(10,2),
    cogs DECIMAL(10,2),
    total_sales DECIMAL(10,2)
);
```

---

### 2. Data Exploration & Cleaning

#### Basic Data Exploration
```sql
-- Total record count
SELECT COUNT(*) FROM retail_sales;

-- Unique customers count
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;

-- Unique product categories
SELECT DISTINCT category FROM retail_sales;
```

#### Check & Remove Null Values
```sql
SELECT * FROM retail_sales
WHERE sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
      gender IS NULL OR age IS NULL OR category IS NULL OR 
      quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

DELETE FROM retail_sales
WHERE sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
      gender IS NULL OR age IS NULL OR category IS NULL OR 
      quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

---

### 3. Data Analysis & Insights

#### Key Business Queries

1. Retrieve all sales from November 5, 2022
```sql
SELECT * FROM retail_sales WHERE sale_date = '2022-11-05';
```

2. Find transactions where category is 'Clothing' and quantity sold >= 4 in November 2022
```sql
SELECT * FROM retail_sales
WHERE category = 'Clothing'
AND EXTRACT(YEAR FROM sale_date) = 2022
AND EXTRACT(MONTH FROM sale_date) = 11
AND quantity >= 4;
```

3. Calculate total sales for each category
```sql
SELECT category, SUM(total_sales) AS total_sales, COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```

4. Find the average age of customers purchasing from 'Beauty' category
```sql
SELECT ROUND(AVG(age), 2) AS avg_age FROM retail_sales WHERE category = 'Beauty';
```

5. Retrieve transactions where total sales exceed 1000
```sql
SELECT * FROM retail_sales WHERE total_sales > 1000;
```

6. Total transactions by gender in each category
```sql
SELECT category, gender, COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

7. Best-selling month each year (by average sales)
```sql
WITH sales_ranking AS (
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year, 
        EXTRACT(MONTH FROM sale_date) AS month, 
        AVG(total_sales) AS avg_sales,
        RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sales) DESC) AS rank
    FROM retail_sales
    GROUP BY year, month
)
SELECT year, month, avg_sales
FROM sales_ranking
WHERE rank = 1;
```

8. Top 5 customers based on total sales
```sql
SELECT customer_id, SUM(total_sales) AS total_spent
FROM retail_sales
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 5;
```

9. Unique customers per product category
```sql
SELECT category, COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category;
```

10. Sales distribution across different time shifts
```sql
WITH shift_sales AS (
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT shift, COUNT(*) AS total_orders FROM shift_sales GROUP BY shift;
```

---

## Findings & Insights

- Customer demographics: Sales are distributed across different age groups and genders.
- High-value transactions: Transactions exceeding $1000 indicate premium purchases.
- Sales trends: Seasonal patterns help identify peak and low-sales periods.
- Top-spending customers: The highest spenders can be targeted for loyalty programs.
- Category performance: Understanding top-performing categories aids inventory planning.

---

## Reports

- Sales summary: Overview of total sales, customer demographics, and category performance.
- Trend analysis: Sales fluctuations across months and shifts.
- Customer insights: Identifying high-value customers and their purchase behavior.

---

## Conclusion

This project showcases SQL-based retail sales analysis, covering database setup, data cleaning, EDA, and business-driven insights. The findings help optimize sales strategies, understand customer behavior, and improve decision-making in retail operations.
