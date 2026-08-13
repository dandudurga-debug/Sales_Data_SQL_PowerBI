-- ============================================================
-- SALES ANALYTICS & CUSTOMER/PRODUCT INTELLIGENCE
-- SQL OPERATIONS
-- Scope: Power BI Page 1 + Page 2
-- ============================================================

-- TABLES:
-- sales_transactions
-- sales_customers
-- sales_products
-- sales_markets
-- sales_date

-- ============================================================
-- 1. BASIC DATA VALIDATION
-- ============================================================

SELECT COUNT(*) AS total_transactions
FROM sales_transactions;

SELECT COUNT(*) AS total_customers
FROM sales_customers;

SELECT COUNT(*) AS total_products
FROM sales_products;

SELECT COUNT(*) AS total_markets
FROM sales_markets;


-- ============================================================
-- 2. BASIC SELECT OPERATIONS
-- ============================================================

SELECT *
FROM sales_transactions;

SELECT
    order_id,
    customer_id,
    product_code,
    sales_qty,
    sales_amount
FROM sales_transactions;


-- ============================================================
-- 3. FILTERING
-- ============================================================

SELECT
    order_id,
    customer_id,
    sales_amount
FROM sales_transactions
WHERE sales_amount > 10000;

SELECT
    order_id,
    product_code,
    sales_qty
FROM sales_transactions
WHERE sales_qty > 10;

SELECT *
FROM sales_transactions
WHERE customer_id IN ('CUST001', 'CUST002', 'CUST003');


-- ============================================================
-- 4. EXECUTIVE SALES KPIs
-- ============================================================

-- Total Revenue
SELECT SUM(sales_amount) AS total_revenue
FROM sales_transactions;

-- Total Sales Quantity
SELECT SUM(sales_qty) AS total_sales_qty
FROM sales_transactions;

-- Average Revenue per Transaction
SELECT AVG(sales_amount) AS avg_transaction_revenue
FROM sales_transactions;

-- Revenue per Unit
SELECT
    SUM(sales_amount) / NULLIF(SUM(sales_qty), 0) AS revenue_per_unit
FROM sales_transactions;


-- ============================================================
-- 5. CUSTOMER ANALYTICS
-- ============================================================

-- Active Customers
SELECT COUNT(DISTINCT customer_id) AS active_customers
FROM sales_transactions;

-- Revenue by Customer
SELECT
    customer_id,
    SUM(sales_amount) AS total_revenue
FROM sales_transactions
GROUP BY customer_id
ORDER BY total_revenue DESC;

-- Top 10 Customers
SELECT
    customer_id,
    SUM(sales_amount) AS total_revenue
FROM sales_transactions
GROUP BY customer_id
ORDER BY total_revenue DESC
LIMIT 10;

-- Customer Purchase Frequency
SELECT
    customer_id,
    COUNT(DISTINCT order_id) AS customer_orders
FROM sales_transactions
GROUP BY customer_id
ORDER BY customer_orders DESC;


-- ============================================================
-- 6. CUSTOMER SEGMENT ANALYSIS
-- ============================================================

SELECT
    c.customer_type,
    SUM(t.sales_amount) AS total_revenue,
    COUNT(DISTINCT t.customer_id) AS active_customers
FROM sales_transactions t
INNER JOIN sales_customers c
    ON t.customer_id = c.customer_id
GROUP BY c.customer_type
ORDER BY total_revenue DESC;


-- ============================================================
-- 7. PRODUCT ANALYTICS
-- ============================================================

-- Revenue by Product
SELECT
    product_code,
    SUM(sales_amount) AS total_revenue
FROM sales_transactions
GROUP BY product_code
ORDER BY total_revenue DESC;

-- Quantity by Product
SELECT
    product_code,
    SUM(sales_qty) AS total_sales_qty
FROM sales_transactions
GROUP BY product_code
ORDER BY total_sales_qty DESC;

-- Revenue by Product Type
SELECT
    p.product_type,
    SUM(t.sales_amount) AS total_revenue,
    SUM(t.sales_qty) AS total_sales_qty
FROM sales_transactions t
INNER JOIN sales_products p
    ON t.product_code = p.product_code
GROUP BY p.product_type
ORDER BY total_revenue DESC;


-- ============================================================
-- 8. PRODUCT REVENUE PER UNIT
-- ============================================================

SELECT
    product_code,
    SUM(sales_amount) AS total_revenue,
    SUM(sales_qty) AS total_quantity,
    SUM(sales_amount) / NULLIF(SUM(sales_qty), 0) AS revenue_per_unit
FROM sales_transactions
GROUP BY product_code
ORDER BY revenue_per_unit DESC;


-- ============================================================
-- 9. CUSTOMER × PRODUCT ANALYSIS
-- ============================================================

SELECT
    t.customer_id,
    t.product_code,
    SUM(t.sales_amount) AS total_revenue,
    SUM(t.sales_qty) AS total_quantity
FROM sales_transactions t
GROUP BY t.customer_id, t.product_code
ORDER BY total_revenue DESC;


-- ============================================================
-- 10. CUSTOMER × PRODUCT TYPE ANALYSIS
-- ============================================================

SELECT
    t.customer_id,
    p.product_type,
    SUM(t.sales_amount) AS total_revenue
FROM sales_transactions t
INNER JOIN sales_products p
    ON t.product_code = p.product_code
GROUP BY t.customer_id, p.product_type
ORDER BY total_revenue DESC;


-- ============================================================
-- 11. CITY / MARKET REVENUE ANALYSIS
-- ============================================================

SELECT
    m.City,
    m.zone,
    SUM(t.sales_amount) AS total_revenue,
    SUM(t.sales_qty) AS total_sales_qty
FROM sales_transactions t
INNER JOIN sales_markets m
    ON t.markets_code = m.markets_code
GROUP BY m.City, m.zone
ORDER BY total_revenue DESC;


-- ============================================================
-- 12. INDIA MAP DATA VALIDATION
-- ============================================================

SELECT
    markets_code,
    City,
    zone,
    Latitude,
    Longitude
FROM sales_markets
ORDER BY City;

-- Missing coordinates
SELECT
    markets_code,
    City,
    Latitude,
    Longitude
FROM sales_markets
WHERE Latitude IS NULL
   OR Longitude IS NULL;

-- Bhubaneshwar validation
SELECT
    markets_code,
    City,
    Latitude,
    Longitude
FROM sales_markets
WHERE City = 'Bhubaneshwar';


-- ============================================================
-- 13. DATA QUALITY
-- ============================================================

-- Duplicate Customers
SELECT
    customer_id,
    COUNT(*) AS record_count
FROM sales_customers
GROUP BY customer_id
HAVING COUNT(*) > 1;

-- Duplicate Products
SELECT
    product_code,
    COUNT(*) AS record_count
FROM sales_products
GROUP BY product_code
HAVING COUNT(*) > 1;

-- NULL Customer IDs
SELECT COUNT(*) AS null_customer_ids
FROM sales_transactions
WHERE customer_id IS NULL;

-- NULL Product Codes
SELECT COUNT(*) AS null_product_codes
FROM sales_transactions
WHERE product_code IS NULL;

-- NULL Sales Amount
SELECT COUNT(*) AS null_sales_amount
FROM sales_transactions
WHERE sales_amount IS NULL;

-- Negative Revenue
SELECT
    order_id,
    sales_amount
FROM sales_transactions
WHERE sales_amount < 0;


-- ============================================================
-- 14. CASE STATEMENT
-- ============================================================

SELECT
    order_id,
    sales_amount,
    CASE
        WHEN sales_amount >= 10000 THEN 'High'
        WHEN sales_amount >= 5000 THEN 'Medium'
        ELSE 'Low'
    END AS revenue_category
FROM sales_transactions;


-- ============================================================
-- 15. GROUP BY + HAVING
-- ============================================================

SELECT
    customer_id,
    SUM(sales_amount) AS total_revenue
FROM sales_transactions
GROUP BY customer_id
HAVING SUM(sales_amount) > 50000
ORDER BY total_revenue DESC;


-- ============================================================
-- 16. CTE — CUSTOMER REVENUE
-- ============================================================

WITH customer_revenue AS
(
    SELECT
        customer_id,
        SUM(sales_amount) AS total_revenue
    FROM sales_transactions
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_revenue
FROM customer_revenue
ORDER BY total_revenue DESC;


-- ============================================================
-- 17. ROW_NUMBER — TOP CUSTOMERS
-- ============================================================

WITH customer_revenue AS
(
    SELECT
        customer_id,
        SUM(sales_amount) AS total_revenue
    FROM sales_transactions
    GROUP BY customer_id
),
ranked_customers AS
(
    SELECT
        customer_id,
        total_revenue,
        ROW_NUMBER() OVER (
            ORDER BY total_revenue DESC
        ) AS customer_rank
    FROM customer_revenue
)
SELECT *
FROM ranked_customers
WHERE customer_rank <= 10;


-- ============================================================
-- 18. RANK — CUSTOMER REVENUE
-- ============================================================

WITH customer_revenue AS
(
    SELECT
        customer_id,
        SUM(sales_amount) AS total_revenue
    FROM sales_transactions
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_revenue,
    RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS revenue_rank
FROM customer_revenue;


-- ============================================================
-- 19. DENSE_RANK — PRODUCT RANKING
-- ============================================================

WITH product_revenue AS
(
    SELECT
        product_code,
        SUM(sales_amount) AS total_revenue
    FROM sales_transactions
    GROUP BY product_code
)
SELECT
    product_code,
    total_revenue,
    DENSE_RANK() OVER (
        ORDER BY total_revenue DESC
    ) AS product_rank
FROM product_revenue;


-- ============================================================
-- 20. CUSTOMER VALUE ANALYSIS
-- ============================================================

SELECT
    customer_id,
    COUNT(DISTINCT order_id) AS order_frequency,
    SUM(sales_amount) AS total_revenue,
    SUM(sales_amount) /
    NULLIF(COUNT(DISTINCT order_id), 0) AS revenue_per_order
FROM sales_transactions
GROUP BY customer_id
ORDER BY total_revenue DESC;


-- ============================================================
-- 21. PRODUCT PERFORMANCE
-- ============================================================

SELECT
    p.product_code,
    p.product_type,
    SUM(t.sales_qty) AS total_quantity,
    SUM(t.sales_amount) AS total_revenue,
    SUM(t.sales_amount) /
    NULLIF(SUM(t.sales_qty), 0) AS revenue_per_unit
FROM sales_transactions t
INNER JOIN sales_products p
    ON t.product_code = p.product_code
GROUP BY p.product_code, p.product_type
ORDER BY total_revenue DESC;


-- ============================================================
-- 22. TOP PRODUCT REVENUE CONTRIBUTION
-- ============================================================

WITH product_revenue AS
(
    SELECT
        product_code,
        SUM(sales_amount) AS product_revenue
    FROM sales_transactions
    GROUP BY product_code
),
total_revenue AS
(
    SELECT SUM(sales_amount) AS total_revenue
    FROM sales_transactions
)
SELECT
    p.product_code,
    p.product_revenue,
    p.product_revenue / NULLIF(t.total_revenue, 0)
        AS revenue_contribution
FROM product_revenue p
CROSS JOIN total_revenue t
ORDER BY revenue_contribution DESC;


-- ============================================================
-- 23. LEFT JOIN — CUSTOMERS WITHOUT TRANSACTIONS
-- ============================================================

SELECT
    c.customer_id,
    c.customer_name
FROM sales_customers c
LEFT JOIN sales_transactions t
    ON c.customer_id = t.customer_id
WHERE t.customer_id IS NULL;


-- ============================================================
-- 24. REFERENTIAL INTEGRITY CHECK
-- ============================================================

SELECT DISTINCT
    t.customer_id
FROM sales_transactions t
LEFT JOIN sales_customers c
    ON t.customer_id = c.customer_id
WHERE c.customer_id IS NULL;


-- ============================================================
-- 25. CONSOLIDATED KPI VALIDATION
-- ============================================================

SELECT
    COUNT(DISTINCT customer_id) AS active_customers,
    COUNT(DISTINCT product_code) AS active_products,
    COUNT(DISTINCT order_id) AS total_orders,
    SUM(sales_qty) AS total_sales_qty,
    SUM(sales_amount) AS total_revenue,
    SUM(sales_amount) /
    NULLIF(SUM(sales_qty), 0) AS revenue_per_unit
FROM sales_transactions;


-- ============================================================
-- 26. SQL → POWER BI KPI MAPPING
-- ============================================================

-- Total Revenue              -> SUM(sales_amount)
-- Total Sales Quantity       -> SUM(sales_qty)
-- Active Customers           -> COUNT(DISTINCT customer_id)
-- Active Products            -> COUNT(DISTINCT product_code)
-- Revenue per Unit            -> Revenue / Quantity
-- Customer Orders             -> COUNT(DISTINCT order_id)
-- Customer Segment Revenue     -> GROUP BY customer_type
-- Product Portfolio Revenue    -> GROUP BY product_type
-- City Revenue                -> GROUP BY City
-- Customer × Product Revenue   -> GROUP BY customer_id, product_type
-- Customer Ranking             -> ROW_NUMBER / RANK
-- Product Ranking              -> DENSE_RANK

-- ============================================================
-- END OF SQL OPERATIONS
-- ============================================================
