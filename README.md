# Retail Sales Analysis | SQL Project

**Author:** Prachay Khatu | Computer Engineering graduate, Mumbai
**Level:** Beginner
**Database:** PostgreSQL (`p1_retail_db`)
**Tools:** SQL, PostgreSQL, pgAdmin

## About This Project

I built this project to practise SQL on a realistic retail dataset. I set up a database, checked the data for quality problems, explored it, and wrote queries that answer 10 business questions.

My background is in software testing (manual, functional and regression testing). Because of that, I paid extra attention to **data validation**: checking for NULLs, duplicates and impossible values before trusting any result.

> This project follows the structure of the "Retail Sales Analysis" SQL project from Zero Analyst's tutorial series. I worked through it myself, fixed a few issues I found in the original queries, and added my own data validation checks.

## Objectives

1. **Set up** a retail sales database and load the data.
2. **Clean** the data by finding and removing records with missing values.
3. **Explore** the dataset (record count, customers, categories).
4. **Analyse** the data with SQL to answer business questions.
5. **Validate** the data with extra QA-style checks.

## 1. Database Setup

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date       DATE,
    sale_time       TIME,
    customer_id     INT,
    gender          VARCHAR(10),
    age             INT,
    category        VARCHAR(35),
    quantity        INT,
    price_per_unit  FLOAT,
    cogs            FLOAT,
    total_sale      FLOAT
);
```

Load the CSV file into the `retail_sales` table (in pgAdmin: right click the table, then **Import/Export Data**).

## 2. Data Exploration and Cleaning

```sql
-- Total number of records
SELECT COUNT(*) FROM retail_sales;

-- Number of unique customers
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;

-- All product categories
SELECT DISTINCT category FROM retail_sales;

-- Find rows with missing values
SELECT * FROM retail_sales
WHERE
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR
    gender IS NULL OR age IS NULL OR category IS NULL OR
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL OR
    total_sale IS NULL;

-- Delete rows with missing values
DELETE FROM retail_sales
WHERE
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR
    gender IS NULL OR age IS NULL OR category IS NULL OR
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL OR
    total_sale IS NULL;
```

**Note:** The original check did not include `total_sale`. I added it so no NULL rows are missed.

## 3. Extra Data Validation Checks (my addition)

```sql
-- Duplicate transaction IDs (should return 0 rows)
SELECT transactions_id, COUNT(*)
FROM retail_sales
GROUP BY transactions_id
HAVING COUNT(*) > 1;

-- Impossible values (should return 0 rows)
SELECT * FROM retail_sales
WHERE quantity <= 0 OR price_per_unit <= 0 OR age <= 0 OR age > 100;

-- Does total_sale = quantity * price_per_unit? (should return 0 rows)
SELECT * FROM retail_sales
WHERE ROUND(total_sale::numeric, 2) <> ROUND((quantity * price_per_unit)::numeric, 2);
```

## 4. Business Questions and Queries

**Q1. Retrieve all columns for sales made on '2022-11-05'.**
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

**Q2. Retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in November 2022.**
```sql
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
  AND quantity > 4;
```
*Fix: the original used `>=` 4, which does not match "more than 4".*

**Q3. Calculate the total sales (total_sale) for each category.**
```sql
SELECT
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*)        AS total_orders
FROM retail_sales
GROUP BY category;
```

**Q4. Find the average age of customers who purchased items from the 'Beauty' category.**
```sql
SELECT
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

**Q5. Find all transactions where total_sale is greater than 1000.**
```sql
SELECT *
FROM retail_sales
WHERE total_sale > 1000;
```

**Q6. Find the total number of transactions made by each gender in each category.**
```sql
SELECT
    category,
    gender,
    COUNT(*) AS total_trans
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

**Q7. Calculate the average sale for each month, and find the best selling month in each year.**
```sql
SELECT year, month, avg_sale
FROM (
    SELECT
        EXTRACT(YEAR FROM sale_date)  AS year,
        EXTRACT(MONTH FROM sale_date) AS month,
        AVG(total_sale)               AS avg_sale,
        RANK() OVER (
            PARTITION BY EXTRACT(YEAR FROM sale_date)
            ORDER BY AVG(total_sale) DESC
        ) AS rank
    FROM retail_sales
    GROUP BY 1, 2
) AS t1
WHERE rank = 1;
```

**Q8. Find the top 5 customers based on the highest total sales.**
```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```

**Q9. Find the number of unique customers who purchased items from each category.**
```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS cnt_unique_customers
FROM retail_sales
GROUP BY category;
```

**Q10. Create each shift and count the number of orders (Morning < 12, Afternoon 12 to 17, Evening > 17).**
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

## Key Findings

*(Fill these in after running the queries on your data, using your own numbers.)*

- **Top category by sales:** ...
- **Best selling month each year:** ...
- **Average age of Beauty customers:** ...
- **Busiest shift:** ...
- **High-value orders (over 1000):** ...

## What I Learned

- Writing `GROUP BY`, `HAVING`, `CASE WHEN` and window functions like `RANK()`.
- Cleaning data by finding NULLs, duplicates and invalid values before analysis.
- Reading a business question carefully (for example, "more than 4" means `> 4`, not `>= 4`).
- Using my testing mindset to double-check query results.

## How to Run

1. Clone this repository.
2. Create the database and table using the setup script above.
3. Import the sales CSV into the `retail_sales` table.
4. Run the cleaning and validation queries.
5. Run the 10 analysis queries and compare the results.

## Contact

**Prachay Khatu**
Mumbai, India
LinkedIn: *add your link*
GitHub: *add your link*
Email: *add your email*
