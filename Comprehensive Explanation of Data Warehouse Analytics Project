# **Comprehensive Explanation of Data Warehouse Analytics Project**

---

## **1. Project Overview**
This SQL project analyzes a retail data warehouse containing **customer**, **product**, and **sales** data. The goal is to extract business insights, clean and transform data, and generate reports for decision-making.

---

## **2. Database Setup & Configuration**
### **a) Selecting the Database**
```sql
USE DataWarehouseAnalytics;
```
- Sets the active database to `DataWarehouseAnalytics`.

### **b) Modifying SQL Mode**
```sql
SET session sql_mode=(SELECT REPLACE(@@sql_mode,"ONLY_FULL_GROUP_BY",""));
```
- Disables `ONLY_FULL_GROUP_BY` mode to allow non-aggregated columns in `GROUP BY` queries.

### **c) Disabling Safe Updates (Temporarily)**
```sql
SET sql_safe_updates=0;
```
- Allows `UPDATE` and `DELETE` operations without a `WHERE` clause (used carefully for data cleaning).

---

## **3. Data Exploration**
### **a) Customer Demographics**
#### **i) Countries Served**
```sql
SELECT DISTINCT(country) FROM customers;
```
- Lists all unique countries where customers are located.

#### **ii) Gender Distribution**
```sql
SELECT gender, COUNT(customer_id) FROM customers GROUP BY 1;
```
- Counts customers by gender (`Male`/`Female`).

#### **iii) Youngest & Oldest Customers**
```sql
SELECT * FROM customers
WHERE birthdate = (SELECT MIN(birthdate) FROM customers)
   OR birthdate = (SELECT MAX(birthdate) FROM customers);
```
- Identifies the oldest and youngest customers based on birthdate.

---

### **b) Product Analysis**
#### **i) Product Hierarchy**
```sql
SELECT DISTINCT(category), subcategory, product_name 
FROM products ORDER BY 1, 2, 3;
```
- Shows the full product catalog structure (Category → Subcategory → Product).

#### **ii) Average Cost by Category**
```sql
SELECT category, CONCAT(ROUND(AVG(cost), 2), ' $') AS avg_cost
FROM products GROUP BY 1 ORDER BY 2 DESC;
```
- Computes the average cost per product category (sorted highest to lowest).

#### **iii) Product Performance**
```sql
-- Top 5 Best-Selling Products
SELECT product_name, SUM(sales_amount) AS Total_sales
FROM products RIGHT JOIN sales USING (product_key)
GROUP BY 1 ORDER BY 2 DESC LIMIT 5;

-- 5 Worst-Selling Products
SELECT product_name, SUM(sales_amount) AS Total_sales
FROM products RIGHT JOIN sales USING (product_key)
GROUP BY 1 ORDER BY 2 ASC LIMIT 5;
```
- Identifies the **highest and lowest revenue-generating products**.

---

## **4. Data Cleaning & Transformation**
### **a) Fixing Birthdates**
```sql
UPDATE customers 
SET birthdate = '1928-12-07' 
WHERE birthdate = '';
```
- Replaces empty birthdates with a default value (`1928-12-07`).

### **b) Standardizing Date Formats**
```sql
-- For customers
UPDATE customers SET birthdate = STR_TO_DATE(birthdate, '%Y-%m-%d');
UPDATE customers SET create_date = STR_TO_DATE(create_date, '%Y-%m-%d');

-- For products
UPDATE products SET start_date = STR_TO_DATE(start_date, '%Y-%m-%d');

-- For sales
UPDATE sales SET order_date = DATE_SUB(shipping_date, INTERVAL 7 DAY);
UPDATE sales SET 
    order_date = STR_TO_DATE(order_date, '%Y-%m-%d'),
    shipping_date = STR_TO_DATE(shipping_date, '%Y-%m-%d'),
    due_date = STR_TO_DATE(due_date, '%Y-%m-%d');
```
- Converts all date fields to a **standardized `YYYY-MM-DD` format**.
- Adjusts `order_date` to be **7 days before shipping** for consistency.

---

## **5. Key Business Metrics**
### **a) Sales Performance**
```sql
-- Total Revenue
SELECT SUM(sales_amount) AS total_sales FROM sales;

-- Total Items Sold
SELECT SUM(quantity) AS total_quantity FROM sales;

-- Average Selling Price
SELECT AVG(price) AS Avg_price FROM sales;

-- Total Orders Placed
SELECT COUNT(DISTINCT(order_number)) AS total_orders FROM sales;
```
- Computes **revenue, volume, pricing, and order count**.

### **b) Customer Metrics**
```sql
-- Total Customers
SELECT COUNT(customer_key) AS total_customers FROM customers;

-- Active Customers (Placed Orders)
SELECT COUNT(DISTINCT(customer_key)) AS total_customers FROM sales;
```
- Differentiates between **registered customers** vs. **active buyers**.

### **c) Product Metrics**
```sql
-- Total Products in Catalog
SELECT COUNT(product_name) AS total_products FROM products;

-- Revenue by Category
SELECT category, SUM(sales_amount) AS total_Sales
FROM sales LEFT JOIN products USING (product_key)
GROUP BY category ORDER BY 2 DESC;
```
- Analyzes **product distribution** and **category-wise revenue**.

---

## **6. Advanced Analytics**
### **a) Time-Series Analysis**
```sql
-- Monthly Sales Trends
SELECT 
    MONTH(order_date) AS Month,
    COUNT(DISTINCT(customer_key)) AS Customer_Count,
    SUM(sales_amount) AS Total_Sales,
    SUM(quantity) AS Total_Quantity
FROM sales
WHERE order_date IS NOT NULL
GROUP BY 1 ORDER BY 1;
```
- Breaks down sales **by month**, including **customer count, revenue, and units sold**.

### **b) Running Sales Total (Window Function)**
```sql
SELECT 
    yr, mo, total_sales,
    SUM(total_sales) OVER(PARTITION BY yr ORDER BY mo) AS running_sales
FROM (
    SELECT 
        YEAR(order_date) AS yr,
        MONTH(order_date) AS mo,
        SUM(sales_amount) AS total_sales
    FROM sales
    GROUP BY yr, mo
) t;
```
- Computes **cumulative revenue** by year.

### **c) Customer Segmentation**
```sql
WITH cte AS (
    SELECT 
        customer_key,
        SUM(sales_amount) AS total_sales,
        TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
    FROM sales
    GROUP BY customer_key
)
SELECT
    CASE 
        WHEN lifespan >= 12 AND total_sales > 5000 THEN "VIP"
        WHEN lifespan >= 12 THEN "Regular"
        ELSE "New"
    END AS Segment,
    COUNT(customer_key) AS Count
FROM cte GROUP BY 1;
```
- Segments customers into:
  - **VIP** (High spenders with long history)
  - **Regular** (Established but lower spend)
  - **New** (Recent buyers)

---

## **7. Business Reports**
### **a) Customer Report**
```sql
WITH cte AS (
    SELECT 
        customer_key,
        CONCAT(first_name, " ", last_name) AS Customer_name,
        TIMESTAMPDIFF(YEAR, birthdate, CURDATE()) AS age,
        SUM(sales_amount) AS total_sales,
        COUNT(DISTINCT order_number) AS Total_orders,
        TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
        MAX(order_date) AS last_order_date
    FROM sales LEFT JOIN customers USING (customer_key)
    GROUP BY customer_key
)
SELECT 
    customer_name,
    age,
    total_sales,
    Total_orders,
    lifespan,
    TIMESTAMPDIFF(MONTH, last_order_date, CURDATE()) AS recency,
    CASE 
        WHEN age < 20 THEN "Under 20"
        WHEN age BETWEEN 21 AND 29 THEN "21-29"
        WHEN age BETWEEN 31 AND 39 THEN "31-39"
        ELSE "40+"
    END AS age_group
FROM cte;
```
- Provides a **360° customer view**, including:
  - **Demographics** (age, name)
  - **Purchase behavior** (total spend, order count)
  - **Engagement metrics** (lifespan, recency)

### **b) Product Report**
```sql
WITH cte AS (
    SELECT 
        product_key,
        product_name,
        category,
        subcategory,
        SUM(sales_amount) AS total_sales,
        COUNT(DISTINCT customer_key) AS total_customers,
        MAX(order_date) AS last_order_date
    FROM products RIGHT JOIN sales USING (product_key)
    GROUP BY product_key
)
SELECT 
    product_name,
    category,
    total_sales,
    total_customers,
    CASE 
        WHEN total_sales > 50000 THEN "High Performer"
        WHEN total_sales >= 10000 THEN "Mid-Range"
        ELSE "Low Performer"
    END AS performance_segment
FROM cte;
```
- Classifies products into **performance tiers** for inventory optimization.

---

## **8. Key Takeaways**
### **Insights Generated**
1. **Customer Segmentation** → Identified VIP vs. Regular vs. New buyers.
2. **Product Performance** → Ranked best/worst sellers.
3. **Sales Trends** → Monthly revenue patterns.
4. **Demographic Analysis** → Age/gender-based purchasing behavior.

### **SQL Techniques Used**
✅ **Joins** (`INNER`, `LEFT`, `RIGHT`)  
✅ **Aggregations** (`SUM`, `COUNT`, `AVG`)  
✅ **Window Functions** (`OVER`, `PARTITION BY`)  
✅ **Data Cleaning** (`UPDATE`, `STR_TO_DATE`)  
✅ **CTEs (Common Table Expressions)** for modular queries  
✅ **CASE Statements** for dynamic segmentation  

---

## **Conclusion**
This project demonstrates **end-to-end data analysis**—from **data cleaning** to **advanced analytics**—providing actionable insights for business decision-making. The SQL techniques used are **industry-standard** and applicable to real-world analytics scenarios. 🚀
