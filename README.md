# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `p1_retail_db`

This project is designed to demonstrate fundamental SQL skills typically used by data analysts to explore, clean, and analyze retail sales data. You’ll set up a retail database, perform exploratory data analysis (EDA), clean the data, and answer specific business questions using SQL queries.

## 🎯 Objectives

1. **✅ Set up a retail sales database and tables**: Create and populate a retail sales database with the provided sales data.
2. **✅ Clean the data (remove missing/null records)**: Identify and remove any records with missing or null values.
3. **✅ Perform EDA to understand customer and sales behavior**: Perform basic exploratory data analysis to understand the dataset.
4. **✅ Derive insights by answering business questions using SQL**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. 🛠️ Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. 🔍 Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*) FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;

-- Check Nulls

SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

-- Delete Nulls

DELETE FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **✅ Sales on a Specific Date**:
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **✅ Clothing Category with Quantity > 4 in Nov-2022**:
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

3. **✅ Total Sales per Category**:
```sql
SELECT 
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```

4. **✅ Avg Age of Beauty Product Buyers**:
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

5. **✅ High-Value Transactions (> 1000)**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000;
```

6. **✅ Transaction Count by Gender & Category**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

7. **✅ Monthly Avg Sales & Best Month per Year**:
```sql
SELECT 
    year,
    month,
    avg_sale
FROM (
    SELECT 
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) AS rank
    FROM retail_sales
    GROUP BY EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date)
) AS ranked
WHERE rank = 1;
```

8. **✅ Top 5 Customers by Total Sales**:
```sql
SELECT 
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

9. **✅ Unique Customers per Category**:
```sql
SELECT 
    category,
    COUNT(DISTINCT customer_id) AS cnt_unique_cs
FROM retail_sales
GROUP BY category;
```

10. **✅ Orders by Day Shift**:
```sql
WITH hourly_sale AS (
    SELECT *,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) AS total_orders
FROM hourly_sale
GROUP BY shift;
```

11. **What is the gender-wise average purchase amount across all categories?**:
```sql
-- Understand how spending behavior varies between male and female customers

SELECT 
    gender,
    ROUND(AVG(total_sale), 2) AS avg_purchase
FROM retail_sales
GROUP BY gender
ORDER BY avg_purchase DESC;
```
12. **Which product category has the highest average price per unit?**:
```sql
-- Identify premium product categories based on average unit pricing

SELECT 
    category,
    ROUND(AVG(price_per_unit), 2) AS avg_price
FROM retail_sales
GROUP BY category
ORDER BY avg_price DESC
LIMIT 1;
```
13. **How many transactions occurred on weekends vs weekdays?**:
```sql
-- Compare customer behavior on weekdays vs weekends to support marketing or staffing decisions

SELECT 
    CASE 
        WHEN EXTRACT(DOW FROM sale_date) IN (0, 6) THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY day_type;
```
14. **What is the age group distribution of customers making purchases?**:
```sql
-- Segment customer base by age for targeted marketing campaigns.

SELECT 
    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        ELSE '50+'
    END AS age_group,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY age_group
ORDER BY age_group;
```
15. **What is the profit margin by category?**:
```sql
-- Evaluate the profitability of each product category.

SELECT 
    category,
    ROUND(SUM(total_sale - cogs), 2) AS total_profit,
    ROUND(AVG(total_sale - cogs), 2) AS avg_profit_per_transaction
FROM retail_sales
GROUP BY category
ORDER BY total_profit DESC;
```


## 📌 Key Findings

- **Diverse Demographics**: Wide customer age range; gender-based purchasing trends are visible.
- **High-Value Purchases**: Several customers made purchases exceeding 1000 units in value.
- **Sales Trends**: Certain months & shifts show peak sales activity.
- **Top Customers & Categories**: Helps identify customer loyalty and best-selling categories.

## 📊 Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.

## How to Use

1. **Clone the Repository**: Clone this project repository from GitHub.
2. **Set Up the Database**: Run the SQL scripts provided in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries provided in the `analysis_queries.sql` file to perform your analysis.
4. **Explore and Modify**: Feel free to modify the queries to explore different aspects of the dataset or answer additional business questions.

## Author - Sujit Kumar

This project is part of my portfolio, showcasing the SQL skills essential for data analyst roles. If you have any questions, feedback, or would like to collaborate, feel free to get in touch!

### Stay Updated and Join the Community

For more content on SQL, data analysis, and other data-related topics, make sure to follow me on social media:

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/sujitkumar5789/))

Thank you for your support, and I look forward to connecting with you!
