# Fake Apple Retail Sales (1M Rows) - SQL

---

# ![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg)  
# Apple Retail Sales SQL Project  

## Project Overview  

This project analyzes over **1 million rows** of Apple retail sales data using **advanced SQL techniques**. It covers key insights on **sales trends, product performance, store analysis, and warranty claims** across various Apple locations. The project demonstrates SQL proficiency in **joins, window functions, aggregations, and query optimization** for large-scale datasets.  

## Database Schema  

The project includes five main tables:  

- **stores**: Store details (location, country, store name).  
- **category**: Product categories.  
- **products**: Product details (name, category, price, launch date).  
- **sales**: Sales transactions (date, store, product, quantity).  
- **warranty**: Warranty claims and repair status.  

## Key SQL Concepts Covered  

✔ Advanced Joins & Aggregations  
✔ Window Functions (RANK, ROW_NUMBER)  
✔ Subqueries & CTEs  
✔ Query Performance Optimization  
✔ Time-Based Sales Analysis  

## Sample Business Questions  

### Solving Real-World Business Problems with SQL  

#### **Problem 1:** How many warranty claims were filed in 2020?
```sql
SELECT
    COUNT(*) AS warranty_claims_2020
FROM warranty
WHERE EXTRACT(YEAR FROM claim_date) = 2020;
```

#### **Problem 2:** For each store, identify the best-selling day based on the highest quantity sold.
```sql
SELECT *
FROM (
    SELECT
        store_id,
        TO_CHAR(sale_date, 'Day') AS day_name,
        SUM(quantity) AS total_units_sold,
        RANK() OVER(PARTITION BY store_id ORDER BY SUM(quantity) DESC) AS rank
    FROM sales
    GROUP BY 1, 2
) AS ranked_sales
WHERE rank = 1;
```

#### **Problem 3:** Identify the least selling product in each country for each year based on total units sold.
```sql
WITH product_rank AS (
    SELECT
        st.country,
        p.product_name,
        SUM(s.quantity) AS total_qty_sold,
        RANK() OVER(PARTITION BY st.country ORDER BY SUM(s.quantity)) AS rank
    FROM sales AS s
    JOIN stores AS st ON s.store_id = st.store_id
    JOIN products AS p ON s.product_id = p.product_id
    GROUP BY 1, 2
)
SELECT *
FROM product_rank
WHERE rank = 1;
```

#### **Problem 4:** Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT
    COUNT(*)
FROM warranty AS w
LEFT JOIN sales AS s ON s.sale_id = w.sale_id
WHERE w.claim_date - sale_date <= 180;
```

#### **Problem 5:** Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT
    p.product_name,
    COUNT(w.claim_id) AS no_claim,
    COUNT(s.sale_id)
FROM warranty AS w
RIGHT JOIN sales AS s ON s.sale_id = w.sale_id
JOIN products AS p ON p.product_id = s.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 years'
GROUP BY 1
HAVING COUNT(w.claim_id) > 0;
```

#### **Problem 6:** List the months in the last three years where sales exceeded 5,000 units in the USA.
```sql
SELECT
    TO_CHAR(sale_date, 'MM-YYYY') AS month,
    SUM(s.quantity) AS total_unit_sold
FROM sales AS s
JOIN stores AS st ON s.store_id = st.store_id
WHERE st.country = 'USA' AND s.sale_date >= CURRENT_DATE - INTERVAL '3 year'
GROUP BY 1
HAVING SUM(s.quantity) > 5000;
```

#### **Problem 7:** Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT
    c.category_name,
    COUNT(w.claim_id) AS total_claims
FROM warranty AS w
LEFT JOIN sales AS s ON w.sale_id = s.sale_id
JOIN products AS p ON p.product_id = s.product_id
JOIN category AS c ON c.category_id = p.category_id
WHERE w.claim_date >= CURRENT_DATE - INTERVAL '2 year'
GROUP BY 1;
```

#### **Problem 8:** Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT
    country,
    total_unit_sold,
    total_claim,
    COALESCE(total_claim::numeric/total_unit_sold::numeric * 100, 0) AS risk
FROM
(SELECT
    st.country,
    SUM(s.quantity) AS total_unit_sold,
    COUNT(w.claim_id) AS total_claim
FROM sales AS s
JOIN stores AS st ON s.store_id = st.store_id
LEFT JOIN warranty AS w ON w.sale_id = s.sale_id
GROUP BY 1) t1
ORDER BY 4 DESC;
```

#### **Problem 9:** Analyze the year-by-year growth ratio for each store.
```sql
WITH yearly_sales AS (
    SELECT
        s.store_id,
        st.store_name,
        EXTRACT(YEAR FROM sale_date) AS year,
        SUM(s.quantity * p.price) AS total_sale
    FROM sales AS s
    JOIN products AS p ON s.product_id = p.product_id
    JOIN stores AS st ON st.store_id = s.store_id
    GROUP BY 1, 2, 3
    ORDER BY 2, 3
),
growth_ratio AS (
    SELECT
        store_name,
        year,
        LAG(total_sale, 1) OVER(PARTITION BY store_name ORDER BY year) AS last_year_sale,
        total_sale AS current_year_sale
    FROM yearly_sales
)
SELECT
    store_name,
    year,
    last_year_sale,
    current_year_sale,
    ROUND((current_year_sale - last_year_sale)::numeric/last_year_sale::numeric * 100, 3) AS growth_ratio
FROM growth_ratio
WHERE last_year_sale IS NOT NULL AND year <> EXTRACT(YEAR FROM CURRENT_DATE);
```

## Conclusion  

This project is a **real-world SQL case study**, ideal for data professionals looking to strengthen SQL skills and work with large datasets. It demonstrates **data-driven decision-making** and query optimization for **business analytics**. 
