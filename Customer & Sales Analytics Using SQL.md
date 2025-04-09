# Customer & Sales Analytics Using SQL


## 🗃️ Data Preparation and Cleaning – SQL Walkthrough

This section contains SQL scripts used to clean and explore the **DataWarehouseAnalytics** database, preparing it for deeper analysis.

---

## 🔧 Setup and SQL Mode Adjustment

```sql
USE DataWarehouseAnalytics;
SET SESSION sql_mode = (SELECT REPLACE(@@sql_mode, "ONLY_FULL_GROUP_BY", ""));
```
**Explanation:**
- Selects the working database.
- Disables `ONLY_FULL_GROUP_BY` mode to allow more flexible use of `GROUP BY`, especially useful when not all selected columns are aggregated.

---

## 🌍 Explore Customer Locations

```sql
SELECT DISTINCT (country)
FROM customers;
```
**Purpose:**  
Fetches all unique countries where customers are located.

---

## 🧩 Explore Product Hierarchy

```sql
SELECT DISTINCT (category), subcategory, product_name
FROM products
ORDER BY 1, 2, 3;
```
**Purpose:**  
Shows all combinations of **categories, subcategories, and product names** to understand the product structure.

---

## 🧹 Data Cleaning with Transactions

### 🔄 Clean and Format `birthdate` in customers

```sql
SET sql_safe_updates = 0;
START TRANSACTION;
UPDATE customers 
SET birthdate = '1928-12-07'
WHERE birthdate = '';

UPDATE customers 
SET birthdate = STR_TO_DATE(birthdate, '%Y-%m-%d');

SELECT * FROM customers;
COMMIT;
```
**Explanation:**
- Disables safe update mode to allow unrestricted updates.
- Replaces empty birthdates with a placeholder date.
- Converts birthdate values to proper **DATE format**.
- Uses transactions to ensure updates can be rolled back if needed.

---

### 📅 Format `create_date` in customers

```sql
START TRANSACTION;
UPDATE customers 
SET create_date = STR_TO_DATE(create_date, '%Y-%m-%d');

SELECT * FROM customers;
COMMIT;
```
**Purpose:**  
Converts `create_date` strings into a valid **DATE format**.

---

### 🕒 Format `start_date` in products

```sql
START TRANSACTION;
UPDATE products 
SET start_date = STR_TO_DATE(start_date, '%Y-%m-%d');
COMMIT;
```
**Purpose:**  
Cleans and converts **product start_date** into proper date format.

---

## 📦 Clean and Set Order Dates in sales

```sql
START TRANSACTION;
UPDATE sales 
SET order_date = DATE_SUB(shipping_date, INTERVAL 7 DAY);

UPDATE sales 
SET 
    order_date = STR_TO_DATE(order_date, '%Y-%m-%d'),
    shipping_date = STR_TO_DATE(shipping_date, '%Y-%m-%d'),
    due_date = STR_TO_DATE(due_date, '%Y-%m-%d');

COMMIT;
```
**Purpose:**  
- Sets `order_date` to **7 days before shipping_date** as a placeholder if missing.
- Ensures all date fields are correctly formatted.

---

## 📆 Date Range and Monthly Performance

### 🕰️ First and Last Order Date

```sql
SELECT 
    MIN(order_date) AS First_order_date,
    MAX(order_date) AS Last_order_date,
    TIMESTAMPDIFF(YEAR, MIN(order_date), MAX(order_date)) AS order_range
FROM sales;
```
**Purpose:**  
Finds the **earliest and latest order dates** and calculates the range in years.

---

### 📊 Monthly Sales Summary

```sql
SELECT 
    MONTH(order_date) AS Month,
    COUNT(DISTINCT customer_key) AS Count,
    SUM(sales_amount) AS Total_Sales,
    SUM(quantity) AS Total_Quantity
FROM sales
WHERE order_date IS NOT NULL
GROUP BY 1
ORDER BY 1;
```
**Purpose:**  
Aggregates **key monthly metrics**:
- Total **customers**
- Total **sales**
- Total **quantity ordered**

---

## 👶👵 Age Demographics

### 🔎 Youngest and Oldest Customers

```sql
SELECT *
FROM customers
WHERE birthdate = (SELECT MIN(birthdate) FROM customers)
   OR birthdate = (SELECT MAX(birthdate) FROM customers);
```
**Purpose:**  
Returns the **oldest and youngest customers** based on their birthdate.

# 📊 Descriptive Business Metrics – SQL Reporting & Aggregates

This section calculates key **business metrics** such as sales, orders, product counts, and customer breakdowns using **SQL queries** on the `sales`, `customers`, and `products` tables.

---

## 💰 Total Sales Amount
```sql
SELECT SUM(sales_amount) AS total_sales
FROM sales;
```
**Purpose:**  
Returns the total **revenue generated** from all sales.

---

## 📦 Total Quantity Sold
```sql
SELECT SUM(quantity) AS total_quantity
FROM sales;
```
**Purpose:**  
Calculates the total **number of items sold**.

---

## 💵 Average Selling Price
```sql
SELECT AVG(price) AS Avg_price
FROM sales;
```
**Purpose:**  
Gives the **average price per item sold**.

---

## 🧾 Total Number of Orders
```sql
SELECT COUNT(DISTINCT order_number) AS total_orders
FROM sales;
```
**Purpose:**  
Counts all **unique orders placed**.

---

## 🛍️ Total Number of Products
```sql
SELECT COUNT(product_name) AS total_products
FROM products;
```
**Purpose:**  
Counts all **products listed** in the database.

---

## 👥 Total Number of Customers
```sql
SELECT COUNT(customer_key) AS total_customers
FROM customers;
```
**Purpose:**  
Returns the **total number of customer records** in the database.

---

## 📈 Customers Who Placed Orders
```sql
SELECT COUNT(DISTINCT customer_key) AS total_customers
FROM sales;
```
**Purpose:**  
Counts how many **unique customers** actually placed an order.

---

## 🧾 Consolidated Metrics Report
```sql
SELECT 'Total Sales' AS measure_name, SUM(sales_amount) AS measure_value FROM sales
UNION ALL
SELECT 'Avg Price', ROUND(AVG(price)) FROM sales
UNION ALL
SELECT 'Total Orders', COUNT(DISTINCT order_number) FROM sales
UNION ALL
SELECT 'Total Products', COUNT(product_name) FROM products
UNION ALL
SELECT 'Total Customers', COUNT(customer_key) FROM sales
UNION ALL
SELECT 'Total customers', COUNT(DISTINCT customer_key) FROM sales;
```
**Purpose:**  
Combines all **major KPI metrics** into a single **vertical report**.  
Useful for dashboards or summary views.

---

# 🌍 Customer Demographics and Distributions

## 🔢 Total Customers by Country
```sql
SELECT country, COUNT(customer_id)
FROM customers
GROUP BY country
ORDER BY 2 DESC;
```
**Purpose:**  
Breaks down the **number of customers by country**.

---

## ⚥ Total Customers by Gender
```sql
SELECT gender, COUNT(customer_id)
FROM customers
GROUP BY 1;
```
**Purpose:**  
Shows **gender distribution** among all customers.

---

# 🧩 Product Distribution and Revenue

## 📂 Total Products by Category
```sql
SELECT category, COUNT(product_id)
FROM products
GROUP BY 1;
```
**Purpose:**  
Counts **how many products** exist within each **product category**.

---

## 💲 Average Product Cost per Category
```sql
SELECT category, CONCAT(ROUND(AVG(cost), 2), ' $') AS avg_cost
FROM products
GROUP BY 1
ORDER BY 2 DESC;
```
**Purpose:**  
Returns the **average product cost** per category, formatted with a **dollar sign**.

---

## 💵 Total Revenue by Category
```sql
SELECT category, SUM(sales_amount) AS total_Sales
FROM sales
LEFT JOIN products USING (product_key)
GROUP BY category
ORDER BY 2 DESC;
```

# 📈 Advanced Business Insights with SQL

These queries focus on **customer contributions, product performance, geographic sales distribution**, and applying **window functions** for advanced aggregations.

---

## 💸 Total Revenue by Each Customer

```sql
SELECT 
    customer_id,
    first_name,
    last_name,
    SUM(sales_amount) AS Total_revenue
FROM sales
LEFT JOIN customers USING (customer_key)
GROUP BY 1
ORDER BY 4 DESC;
```
**Purpose:**  
Identifies **how much total revenue** was generated by each individual customer.

---

## 🌍 Distribution of Sold Items Across Countries

```sql
SELECT 
    country, SUM(quantity) AS total_quantity
FROM customers
INNER JOIN sales USING (customer_key)
GROUP BY 1
ORDER BY 2 DESC;
```
**Purpose:**  
Shows **how many items** were sold in each country.

---

## 🏆 Top 5 Products by Revenue

```sql
SELECT 
    product_name, SUM(sales_amount) AS Total_sales
FROM products
RIGHT JOIN sales USING (product_key)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
```
**Purpose:**  
Lists the **five best-selling products** by total sales revenue.

---

## 📉 Bottom 5 Products by Revenue

```sql
SELECT 
    product_name, SUM(sales_amount) AS Total_sales
FROM products
RIGHT JOIN sales USING (product_key)
GROUP BY 1
ORDER BY 2 ASC
LIMIT 5;
```
**Purpose:**  
Identifies the **five lowest-performing products** in terms of revenue.

---

## 🔝 Top 2 Subcategories by Revenue

```sql
SELECT 
    subcategory, SUM(sales_amount) AS Total_sales
FROM products
RIGHT JOIN sales USING (product_key)
GROUP BY 1
ORDER BY 2 DESC
LIMIT 2;
```
**Purpose:**  
Highlights which **two subcategories** generated the highest revenue.

---

# 🪟 Window Functions

### 🎯 Top 5 Subcategories by Row Number (Revenue-based)

```sql
SELECT *
FROM (
    SELECT 
        subcategory,
        ROW_NUMBER() OVER (ORDER BY SUM(sales_amount) DESC) AS ranks
    FROM sales
    LEFT JOIN products USING (product_key)
    GROUP BY 1
) t
WHERE ranks <= 5;
```
**Purpose:**  
Ranks **subcategories by revenue** using `ROW_NUMBER()` and selects the **top 5**.

---

### 🥇 Top 5 Subcategories by Dense Rank (Revenue-based)

```sql
SELECT *
FROM (
    SELECT 
        subcategory,
        DENSE_RANK() OVER (ORDER BY SUM(sales_amount) DESC) AS ranks
    FROM sales
    LEFT JOIN products USING (product_key)
    GROUP BY 1
) t
WHERE ranks <= 5;
```
**Purpose:**  
Uses `DENSE_RANK()` to account for ties in revenue, listing the **top 5 revenue-generating subcategories**.

---

## 🧑‍💼 Top 10 Customers by Revenue

```sql
SELECT 
    CONCAT(first_name, ' ', last_name)
FROM sales
LEFT JOIN customers USING (customer_key)
GROUP BY customer_key
ORDER BY SUM(sales_amount) DESC
LIMIT 10;
```
**Purpose:**  
Identifies the **top 10 customers** who generated the most revenue.

---

## 🙋‍♂️ 3 Customers with Fewest Orders

```sql
SELECT 
    customer_key,
    CONCAT(first_name, ' ', last_name) AS name,
    COUNT(DISTINCT order_number) AS order_count
FROM sales
LEFT JOIN customers USING (customer_key)
GROUP BY customer_key
ORDER BY order_count ASC
LIMIT 3;
```
**Purpose:**  
Finds **the customers with the fewest unique orders**.

---

# 📅 Monthly Sales Trends

## 📆 Monthly Sales, Quantity & Customer Count

```sql
SELECT 
    MONTH(order_date) AS month,
    SUM(sales_amount) AS sales,
    SUM(quantity) AS quantity,
    COUNT(DISTINCT customer_key) AS Customer_count
FROM sales
GROUP BY 1
ORDER BY 1;
```
**Purpose:**  
Provides a **snapshot of sales performance** by month.

---

## 📊 Running Sales & Moving Average by Year and Month

```sql
SELECT 
    yr,
    mo,
    total_sales,
    SUM(total_sales) OVER (PARTITION BY yr ORDER BY mo) AS running_sales,
    ROUND(AVG(avg_price) OVER (PARTITION BY yr ORDER BY mo), 2) AS moving_avg
FROM (
    SELECT 
        YEAR(order_date) AS yr,
        MONTH(order_date) AS mo,
        SUM(sales_amount) AS total_sales,
        AVG(price) AS avg_price
    FROM sales
    GROUP BY yr, mo
    ORDER BY yr, mo
) t;
```
**Purpose:**  
- `running_sales`: **Cumulative sales** over months for each year.  
- `moving_avg`: **Monthly rolling average** price per item.

# 🧠 Advanced Product and Customer Segmentation Analysis

This section focuses on **yearly sales performance, contribution analysis, cost segmentation**, and **customer classification** based on behavioral metrics.

---

## 📅 Yearly Product Performance vs. Historical and Average Sales

```sql
WITH cte AS (
  SELECT 
    year,
    product_name,
    total_sales,
    LAG(total_sales) OVER (
      PARTITION BY product_name 
      ORDER BY year
    ) AS previous_year_sales,
    AVG(total_sales) OVER (
      PARTITION BY product_name
    ) AS average_sales
  FROM (
    SELECT 
      YEAR(order_date) AS year,
      product_name,
      SUM(sales_amount) AS total_sales
    FROM sales 
    LEFT JOIN products USING(product_key)
    GROUP BY product_key, year, product_name
  ) t
)
SELECT 
  year,
  product_name,
  total_sales,
  average_sales,
  total_sales - average_sales AS av_diff,
  CASE 
    WHEN total_sales - average_sales < 0 THEN 'Below Average'
    WHEN total_sales - average_sales > 0 THEN 'Above Average'
    ELSE 'Avg'
  END AS avg_change,
  previous_year_sales,
  total_sales - previous_year_sales AS sale_diff,
  CASE 
    WHEN total_sales - previous_year_sales < 0 THEN 'Sales Decreased'
    WHEN total_sales - previous_year_sales > 0 THEN 'Sales Improved'
    ELSE 'No Change'
  END AS sale_change
FROM cte;
```

**Purpose:**  
Compares each product’s **yearly sales** with:  
- Its **average sales** across all years.  
- The **previous year’s sales** to assess performance trends.

---

## 🧾 Category-Wise Contribution to Total Sales

```sql
WITH cte AS (
  SELECT 
    category,
    SUM(sales_amount) AS Sales
  FROM sales 
  LEFT JOIN products USING(product_key)
  GROUP BY 1
)
SELECT 
  category,
  Sales,
  SUM(Sales) OVER() AS overall_sales,
  CONCAT(ROUND((Sales / SUM(Sales) OVER()) * 100, 2), '%') AS Contribution
FROM cte;
```

**Purpose:**  
Shows how much each **product category** contributes to the **overall revenue**.

---

## 💰 Product Segmentation Based on Cost

```sql
WITH cte AS (
  SELECT 
    product_name,
    cost,
    CASE 
      WHEN cost < 100 THEN 'Below 100'
      WHEN cost BETWEEN 100 AND 500 THEN '100 - 500'
      WHEN cost BETWEEN 500 AND 1000 THEN '500 - 1000'
      ELSE 'Above 1000'
    END AS Cost_range
  FROM products
)
SELECT 
  Cost_range,
  COUNT(product_name) AS total_products
FROM cte 
GROUP BY 1 
ORDER BY 2 DESC;
```

**Purpose:**  
Segments all **products into cost brackets** to understand product **pricing distribution**.

---

## 👥 Customer Segmentation by History and Spending

```sql
WITH cte AS (
  SELECT 
    CONCAT(first_name, ' ', last_name) AS Name,
    customer_key,
    SUM(sales_amount) AS total_sales,
    TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS history
  FROM customers 
  RIGHT JOIN sales USING (customer_key)
  GROUP BY 2
)
SELECT
  CASE 
    WHEN history >= 12 AND total_sales > 5000 THEN 'VIP'
    WHEN history >= 12 AND total_sales <= 5000 THEN 'Regular'
    ELSE 'New'
  END AS Segment,
  COUNT(customer_key) AS Count
FROM cte 
GROUP BY 1;
```

**Purpose:**  
Segments customers into:  
- **VIP** – **Loyal, high-spending customers.**  
- **Regular** – **Loyal but lower-spending customers.**  
- **New** – **Less than 12 months of purchase history.**


# 📊 Customer Behavior and Segmentation Report

## **Overview**
This report consolidates **key customer metrics and behaviors**, offering a **comprehensive analysis** of customer **spending patterns, segmentation, and lifecycle trends**.

---

## **Objectives**
- Extract essential fields such as **names, ages, and transaction details**.
- Segment customers into categories **(VIP, Regular, New)** and **age groups**.
- Aggregate customer-level metrics:
  - **Total orders**
  - **Total sales**
  - **Total quantity purchased**
  - **Customer lifespan (in months)**
- Calculate valuable KPIs:
  - **Recency** (months since last order)
  - **Average order value**
  - **Average monthly spending**

---

## 🧮 **SQL Query**
```sql
WITH cte AS (
  SELECT 
    customer_key,
    order_date,
    order_number,
    sales_amount,
    quantity,
    customer_id,
    customer_number,
    CONCAT(first_name, " ", last_name) AS customer_name,
    TIMESTAMPDIFF(YEAR, birthdate, CURDATE()) AS age
  FROM sales
  LEFT JOIN customers USING (customer_key)
),
cte1 AS (
  SELECT 
    customer_key,
    customer_id,
    customer_number,
    customer_name,
    age,
    SUM(sales_amount) AS total_sales,
    SUM(quantity) AS total_quantity,
    COUNT(DISTINCT order_number) AS total_orders,
    TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan,
    MAX(order_date) AS last_order_date
  FROM cte
  GROUP BY 1
)
SELECT 
  customer_key,
  customer_id,
  customer_number,
  customer_name,
  age,
  total_sales,
  total_quantity,
  total_orders,
  lifespan,
  last_order_date,
  TIMESTAMPDIFF(MONTH, last_order_date, CURDATE()) AS recency,
  CASE 
    WHEN total_sales = 0 THEN 0
    ELSE ROUND(total_sales / total_orders, 2)
  END AS avg_order_value,
  CASE 
    WHEN lifespan = 0 THEN total_sales
    ELSE ROUND(total_sales / lifespan, 2)
  END AS avg_monthly_spending,
  CASE 
    WHEN age < 20 THEN "Under 20"
    WHEN age BETWEEN 21 AND 29 THEN "21-29"
    WHEN age BETWEEN 31 AND 39 THEN "31-39"
    WHEN age BETWEEN 41 AND 49 THEN "41-49"
    ELSE "50 and above"
  END AS age_group,
  CASE 
    WHEN lifespan >= 12 AND total_sales > 5000 THEN "VIP"
    WHEN lifespan >= 12 AND total_sales <= 5000 THEN "Regular"
    ELSE "New"
  END AS segment
FROM cte1;
```

---

## **Key Metrics Computed**
| **Metric** | **Description** |
|------------|---------------|
| **Total Sales** | Sum of all purchases made by the customer. |
| **Total Quantity** | Total number of items purchased. |
| **Total Orders** | Number of distinct orders placed. |
| **Lifespan** | Number of months between first and last purchase. |
| **Recency** | Months since last order. |
| **Average Order Value (AOV)** | Revenue per order. |
| **Average Monthly Spending** | Revenue divided by active months. |
| **Age Group** | Customer age bracket classification. |
| **Segment** | Customer classification as **VIP, Regular, or New** based on purchase behavior. |

---


# 📦 Product Performance & Segmentation Report

## **Overview**
This report consolidates key **product metrics and behaviors**, offering a **detailed analysis** of product **sales trends, customer engagement, and lifecycle performance**.

---

## **Objectives**
- Extract essential fields such as **product name, category, subcategory, and cost**.
- Segment products by revenue to classify:
  - **High-Performers**
  - **Mid-Range**
  - **Low-Performers**
- Aggregate product-level metrics:
  - **Total orders**
  - **Total sales**
  - **Total quantity sold**
  - **Unique customers**
  - **Product lifespan (months)**
- Calculate valuable KPIs:
  - **Recency** (months since last sale)
  - **Average order revenue (AOR)**
  - **Average monthly revenue**

---

## 🧮 **SQL Query**
```sql
WITH cte AS (
  SELECT 
    product_key,
    customer_key,
    product_name,
    category,
    subcategory,
    cost,
    order_number,
    order_date,
    sales_amount,
    quantity
  FROM products
  RIGHT JOIN sales USING (product_key)
),
cte1 AS (
  SELECT 
    product_key,
    product_name,
    category,
    subcategory,
    COUNT(DISTINCT order_number) AS total_orders,
    SUM(sales_amount) AS total_sales,
    SUM(quantity) AS total_quantity,
    COUNT(DISTINCT customer_key) AS total_customers,
    MAX(order_date) AS last_order_date,
    TIMESTAMPDIFF(MONTH, MIN(order_date), MAX(order_date)) AS lifespan
  FROM cte
  GROUP BY 1
)
SELECT 
  *,
  CASE 
    WHEN total_sales > 50000 THEN "High Performer"
    WHEN total_sales >= 10000 THEN "Mid Performer"
    ELSE "Low Performer"
  END AS product_segment,
  TIMESTAMPDIFF(MONTH, last_order_date, CURDATE()) AS recency,
  CASE 
    WHEN total_sales = 0 THEN 0
    ELSE ROUND(total_sales / total_orders, 2) 
  END AS average_order_revenue,
  CASE 
    WHEN lifespan < 0 THEN total_sales
    ELSE ROUND(total_sales / lifespan, 2)
  END AS average_monthly_revenue
FROM cte1;
```

---

## **Key Metrics Computed**
| **Metric** | **Description** |
|------------|---------------|
| **Total Sales** | Total revenue generated by each product. |
| **Total Quantity** | Number of units sold. |
| **Total Orders** | Number of unique orders placed. |
| **Total Customers** | Number of distinct customers who purchased the product. |
| **Lifespan** | Active period in months (**first sale to last sale**). |
| **Recency** | Months since last sale. |
| **Average Order Revenue (AOR)** | Revenue per order for each product. |
| **Average Monthly Revenue** | Monthly contribution of the product. |
| **Product Segment** | Categorization based on revenue (**High, Mid, Low performers**). |

---

## **Conclusion: Unlocking Retail Data Intelligence with SQL**  

This **Retail Data Intelligence** project has demonstrated the power of **SQL-driven analytics** in extracting meaningful insights from sales transactions, customer behaviors, and product performance. Through **data cleaning, exploratory analysis, segmentation, and advanced reporting**, we have uncovered patterns that can inform business strategy, improve customer engagement, and optimize product offerings.

### **Key Takeaways**  
1. **Customer Behavior Analysis**  
   - Segmented customers into **VIP, Regular, and New categories** based on spending and purchase history.  
   - Identified key metrics such as **recency, lifespan, and monthly spending trends**, helping businesses enhance customer retention strategies.  

2. **Product Performance Evaluation**  
   - Categorized products based on **sales performance** into **high-performing, mid-range, and low-performing products**.  
   - Conducted **cost segmentation analysis**, helping in pricing strategies and inventory management.  

3. **Revenue & Sales Trends**  
   - Explored **total revenue** from different customer demographics and product categories.  
   - Used **window functions** to analyze **monthly moving averages and cumulative revenue**, aiding in financial forecasting.  

4. **Geographic Insights & Market Trends**  
   - Mapped sales distribution **across countries**, identifying regions with high purchasing activity.  
   - Used aggregated reports to **track overall business performance**.  

### **Impact & Business Applications**  
The findings from this analysis enable businesses to:  
✅ **Optimize pricing strategies** based on cost segmentation and revenue contribution.  
✅ **Enhance customer loyalty programs** by identifying high-value customers.  
✅ **Improve inventory planning** based on product performance and sales trends.  
✅ **Refine targeted marketing strategies** for different customer segments.  

### **Future Enhancements**  
🚀 To further enrich this project, future iterations could incorporate:  
- **Data visualization** using Tableau or Power BI to enhance the presentation of insights.  
- **Predictive analytics models** to forecast future sales trends.  
- **Real-time analytics integration** for dynamic business monitoring.  
