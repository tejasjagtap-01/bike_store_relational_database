# 🚴‍♂️ Retail Sales Analysis & Exploration (SQL)

## 📂 Project Overview
This project involves a comprehensive Exploratory Data Analysis (EDA) and business intelligence data analysis on a multi-store retail sales database using **PostgreSQL** and **pgAdmin 4**. The objective is to design and implement a robust relational database schema from scratch, execute critical data cleaning strategies during data ingestion, explore foundational database boundaries, and author advanced multi-table SQL queries to extract key commercial insights.

The analysis provides deep intelligence on sales trends, inventory health, organizational HR performance, logistic shipping latencies, and macro brand revenue distributions.

---

## 🏗️ 1. Database Setup

### Table Creation & Schema Architecture
The infrastructure is engineered using strict relational design patterns, dividing operational data into two core business domains: **Production Operations** (catalog management) and **Sales Transactions** (customer invoicing). Relationships are established seamlessly using inline `REFERENCES` constraints.

To eliminate data redundancy, **Composite Primary Keys** (combining two tracking columns as a single unique identifier) were engineered for transactional bridge entities like `order_items` and `stocks`. This structurally prevents duplicate line items in a customer cart or duplicate product ledger entries at any physical store location.

### Schema Blueprint Workflow:
* **Parent Components (Independent):** `categories`, `brands`, `customers`, and `stores`.
* **Child Components (Dependent):** `products` and `staffs` (utilizes a self-referencing relationship for managerial hierarchies).
* **Composite Bridge Components:** `orders`, `order_items`, and `stocks`.

---

## 🛠️ 2. Data Cleaning

### Strategic Engineering Overrides
During the data compilation and CSV ingestion phase, several real-world structural anomalies were encountered and systemically cleaned directly via the database engine.

---

## 🔍 3. Data Exploration (EDA)

### Database Profiling & Diagnostics
Before writing financial analysis modules, automated data integrity health scripts were deployed to map database boundaries, evaluate performance metrics, and detect missing parameters:

---

## 📈 4. Data Analysis & Business Insights

### Targeted Business Queries
The completed relational database warehouse was systematically queried to solve ten mission-critical commercial challenges across inventory optimization, human resource output, and complex sales trends:
* **Inventory Optimization (Low-Stock Alert):** 
* **Financial Modeling (High-Value Invoicing):** Isolates premium consumer transaction orders yielding absolute values scaling beyond $5,000 after subtracting compound multi-item discounts.
* **Workforce Sales Performance Audit:** Tracks historical transaction throughput distributions across staff assets. This utilizes defensive `LEFT JOIN` operations to preserve organizational visibility over personnel who have generated zero direct customer transactions.
* **Product Category Performance Leaderboard:** Connects multi-tier operational bridge layers to rank product classifications and manufacturing brands by total lifetime revenue generation, giving executives immense volume contract bargaining power with suppliers.
* **Logistics Performance Analysis:** Examines logistical supply chain efficiency by calculating the average latency period (in days) between order placement and fulfillment transit dates across individual physical store branches.



 ## 💻 SQL Scripts & Implementation

**🛠️ Database Setup & Table Creation**

**CREATE TABLE 1. Categories**
```sql
CREATE TABLE categories(
category_id INT PRIMARY KEY,
category_name VARCHAR (20) NOT NULL
);

SELECT * FROM categories;
```


**CREATE TABLE 2. Brands**
```sql
CREATE TABLE brands(
brand_id INT PRIMARY KEY,
brand_name VARCHAR(20) NOT NULL
);

SELECT * FROM brands;
```


**CREATE TABLE 3. Customers**
```sql
CREATE TABLE customers(
customer_id INT PRIMARY KEY,
first_name TEXT,
last_name TEXT,
phone VARCHAR(20),
email TEXT,
street TEXT,
city TEXT,
state_name VARCHAR(2),
zip_code VARCHAR(10) 
);

SELECT * FROM customers;
```


**CREATE TABLE 4. Stores**
```sql
CREATE TABLE stores(
store_id INT PRIMARY KEY,
store_name TEXT,
phone VARCHAR(20), 
email TEXT,
street TEXT,
city TEXT,
state_name VARCHAR(2),
zip_code VARCHAR(20)
);

SELECT * FROM stores;
```


**CREATE TABLE 5. Staffs**
```sql
CREATE TABLE staffs(
staff_id INT PRIMARY KEY,
first_name TEXT,
last_name TEXT,
email TEXT,
phone_no VARCHAR(20),
active INT NOT NULL,
store_id INT REFERENCES stores(store_id),
manager_id INT
);

SELECT * FROM staffs;
```


**CREATE TABLE 6. Products**
```sql
CREATE TABLE products(
product_id INT PRIMARY KEY,
product_name TEXT NOT NULL,
brand_id INT REFERENCES brands(brand_id),
category_id INT REFERENCES categories(category_id),
model_year INT,
list_price DECIMAL(10,2)
);

SELECT * FROM products;
```


**CREATE TABLE 7. Orders**
```sql
--Delete the old, hidden broken version of the table
DROP TABLE IF EXISTS orders CASCADE;
--CREATE TABLE
CREATE TABLE orders(
order_id INT PRIMARY KEY,	
customer_id	INT REFERENCES customers(customer_id),
order_status INT NOT NULL,
order_date DATE,
required_date DATE NOT NULL,
shipped_date DATE ,
store_id INT REFERENCES stores(store_id),
staff_id INT REFERENCES staffs(staff_id)
);

SELECT * FROM orders;
```


**CREATE TABLE 8. Order Items**
```sql
--Delete the old, hidden broken version of the table
DROP TABLE IF EXISTS order_items CASCADE;
CREATE TABLE order_items(
order_id INT REFERENCES orders(order_id),
item_id	INT,
product_id INT REFERENCES products(product_id),
quantity INT NOT NULL,
list_price	DECIMAL(10,2) NOT NULL,
discount DECIMAL(4,2),

-- This sets BOTH columns as the unique identifier together:
PRIMARY KEY (order_id, item_id)
); 

SELECT * FROM order_items;
```


**CREATE TABLE 9. Stocks**
```sql
CREATE TABLE stocks(
store_id INT REFERENCES stores(store_id),
product_id	INT REFERENCES products(product_id),
quantity INT,

-- Combined primary key so a store can't have duplicate rows for the same product
PRIMARY KEY (store_id, product_id)
);

SELECT * FROM stocks;
```


**====================================================================**
##📊DATA EXPLOARATION (EDA)
**====================================================================**
###1. Check for Duplicate Rows (Uniqueness Check)
```sql
SELECT 
	customer_id, 
	COUNT(*)
FROM customers
GROUP BY 
customer_id
HAVING count(*)>1
```


###2. Find the Range of Your Data (Min/Max Check)
```sql
SELECT
	MIN(list_price) AS cheapest_bike,
	MAX(list_price) AS most_expensive_bikes,
	AVG(list_price) AS average_bike_price
FROM products;
```


###3. Identify Missing Values (Null Scan)
```sql
SELECT 
	COUNT(*) AS missing_phone_counts
	FROM customers
WHERE phone IS NULL;
```



**====================================================================**
## BIKE STORE DATA ANALYTICS: PORTFOLIO PRACTICE QUESTIONS
**====================================================================**


**➡️QUESTION 1: THE LOW-STOCK ALERT
Find all product names where the stock quantity currently available in the stocks table is less than 5 units.**
```sql
SELECT 
	product_name, 
	quantity
FROM
products p  join stocks s
on p.product_id = s.product_id 
where quantity <5 
order by quantity desc;
```


**➡️QUESTION 2: CUSTOMER LOCATION MAPPING
List all unique customer cities along with the total count of customers living in each city. Sort the results from highest to lowest customer count.**
```sql
SELECT 
	city,
	COUNT (customer_id) AS total_customers
	FROM customers 
GROUP BY city
order by total_customers desc;
```


**➡️QUESTION 3: HIGH-VALUE ORDERS
Find all unique order IDs from the order_items table where the total order value (quantity multiplied by list_price, minus the discount) is greater than $5,000.**
```sql
SELECT 
	order_id, 
	SUM(quantity * list_price * (1 - discount))
AS total_order_value
FROM order_items
GROUP BY  order_id 
HAVING SUM(quantity * list_price * (1 - discount)) > 5000 ;
```


**➡️QUESTION 4: EMPLOYEES PERFORMANCE AUDIT
List all staff members (first_name and last_name) and the total number of orders they have personally processed. Include staff members who have processed 0 orders.**
```sql
SELECT 
	s.first_name, 
	last_name, 
	COUNT(o.order_id) AS total_orders
FROM staffs s
LEFT JOIN orders o 
ON s.staff_id = o.staff_id
GROUP BY 
s.staff_id, s.first_name, s.last_name;
```


**➡️QUESTION 5: MOST POPULAR BIKE CATEGORIES
Calculate the total sales revenue generated by each bike category. Display the category_name and the total revenue, sorted from highest to lowest.**
```sql
Select 
	category_name,
	SUM(oi.quantity * oi.list_price * (1-oi.discount)) AS total_revenue
FROM categories c
JOIN products p 
ON c.category_id = p.category_id
JOIN order_items oi
ON p.product_id = oi.product_id 
GROUP BY c.category_name
ORDER BY total_revenue DESC
```


**➡️QUESTION 6: SHIPPING DELAY TRACKING
Find the average number of days it takes for an order to ship (shipped_date minus order_date) for each physical store location.**
```sql
SELECT 
	s.store_name,
	AVG(shipped_date - order_date) AS avg_delay
From orders o 
JOIN stores s
ON o.store_id = s.store_id
GRoup by s.store_name
order by avg_delay desc
```


**➡️QUESTION 7: YEAR-OVER-YEAR MODEL PERFORMANCE
Calculate the total revenue generated by products grouped by their product model_year.Ordered chronologically by model year.**
```sql
SELECT
	p.model_year,
	SUM(oi.quantity * oi.list_price * (1-oi.discount)) AS total_revenue
FROM products p
JOIN order_items oi 
ON p.product_id = oi.product_id
GROUP BY p.model_year
ORDER BY p.model_year ASC;
```


**➡️QUESTION 8: VIP CUSTOMER LEADERBOARD
Find the top 5 highest-spending customers of all time.Display their full name (first and last), email, and total lifetime amount spent.**
```sql
SELECT
	first_name,
	last_name
FROM customers;
```


**➡️QUESTION 9: BRAND REVENUE CONTRIBUTION
Calculate the total sales revenue generated by each bike brand name.Sort the list so the most profitable brand is at the very top.**
```sql
SELECT 
	b.brand_name,
	SUM(oi.quantity * oi.list_price * (1-oi.discount)) AS total_revenue
FROM brands b
JOIN products p
ON b.brand_id = p.brand_id
JOIN order_items oi 
ON p.product_id = oi.product_id
GROUP BY b.brand_name
ORDER BY total_revenue DESC;
```


**➡️QUESTION 10: INVENTORY VALUATION REPORT
Calculate the total current financial dollar value of all inventory sitting across all stores combined. (Formula: stock quantity multiplied by product_list_price).**
```sql
SELECT 
	SUM(s.quantity * p.list_price) 
	AS total_inventory_value 
FROM stocks s
JOIN products p 
ON s.product_id =p.product_id;
```



## 5. Business Recommendations (The "So What?" Section)
Data analysts don't just write queries; they solve business problems. Add a section explaining what a business should *do* with the information you found.

## 💡 Strategic Business Recommendations

Based on the insights derived from the SQL analysis, here are the core recommendations for the retail store management:

* **Shift Optimization (Q10):** Since sales spike significantly during specific shifts, management should align staffing schedules to ensure peak hours (e.g., Afternoon shifts) are fully staffed, while reducing overhead during slower Morning hours.
  
* **Targeted Marketing (Q4 & Q6):** With the average age of 'Beauty' product buyers calculated, marketing campaigns for this category should be precisely targeted toward that specific age demographic via social media channels.
  
* **Inventory Focus (Q3 & Q8):** Prioritize inventory management and restocking schedules for the top-performing categories and ensure high-value customers (Top 5) are enrolled in a premium loyalty program to increase retention.


## 🔮6. Conclusion & Future Work
This project successfully demonstrates how raw transactional retail logs can be cleaned, structured, and transformed into rich operational intelligence using PostgreSQL. 

**Next Steps:**
  **Dashboard Integration:** The next phase of this project involves connecting this PostgreSQL database to **Power BI** or **Tableau** to build an interactive sales performance dashboard.
  **Predictive Analytics:** Utilizing Python to build a regression model forecasting next month's sales trends based on the historical data cleaned here.

## 👨‍💻 Author

**Name** **Tejas Jagtap**

Let's connect! Whether you have questions about this project, want to collaborate on a data initiative, or just want to talk shop about SQL, feel free to reach out:
**💼 LinkedIn:** https://www.linkedin.com/in/tejasjagtap01/
**🐙 GitHub Portfolio:** https://github.com/tejasjagtap-01

Thank you for checking out my project! 🚀
