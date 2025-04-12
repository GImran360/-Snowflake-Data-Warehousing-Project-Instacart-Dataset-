# 🧠 Snowflake Data Warehousing Project | Instacart Dataset 🛒

# Overview / Introduction 
This project demonstrates how to build a modern data warehouse pipeline using Snowflake and AWS S3. It follows the classic ETL approach (Extract → Transform → Load), converting raw CSV data into meaningful insights by creating dimensional models and performing deep data analysis

# 📦 Data Source & Storage 
# Staging Setup in Snowflake
```
CREATE STAGE my_stage 
URL="s3://dw-snowflake-course-imran/instacart/"
CREDENTIALS=(AWS_KEY_ID='XXXX' AWS_SECRET_KEY='XXXX');
```
# 🧾 File Format Configuration
```
CREATE OR REPLACE FILE FORMAT csv_file_format 
TYPE='CSV'
FIELD_DELIMITER=',' 
SKIP_HEADER=1 
FIELD_OPTIONALLY_ENCLOSED_BY='"';
```
# 🏗️ Raw Data Tables (Staging Area)
## We load the following CSVs from S3 into Snowflake raw tables:

* Aisles

* Departments

* Products

* Orders

```
CREATE TABLE aisles (
  aisle_id INTEGER PRIMARY KEY,
  aisle VARCHAR
);
COPY INTO aisles FROM @my_stage/aisles.csv FILE_FORMAT=(FORMAT_NAME='csv_file_format');
```
# 🧱 Dimensional Modeling (Star Schema)
# Creating meaningful dimension and fact tables for optimized analytics:

# 📘 Dimensions:
* dim_users

* dim_products

* dim_departments

* dim_orders

* dim_aisles

* 📊 Fact Table:
# dim_fact_order_product – Combines products, users, departments, and orders into a powerful analytical hub.

# Order_Products

```
CREATE OR REPLACE TABLE dim_fact_order_product AS
SELECT 
  op.order_id, 
  p.product_id, 
  o.user_id, 
  p.aisle_id, 
  p.department_id, 
  op.add_to_cart_order, 
  op.reordered
FROM order_products op
JOIN orders o ON op.order_id = o.order_id
JOIN products p ON op.product_id = p.product_id;
```
# 📈 Business Insights & Analytics
# 🧮 Total Number of Products Ordered Per Department

```
SELECT COUNT(*) AS total_no_products_order, d.deparment 
FROM dim_fact_order_product fop 
JOIN dim_departments d ON fop.department_id = d.department_id
GROUP BY d.deparment;
```
![image](https://github.com/user-attachments/assets/9d87461e-b293-4126-ba23-034d04a5ce29)

# 🔁 Top 5 Aisles with Highest Reordered Products 

#  Query to find the top 5 aisles with the highest number of reordered products:
```

select * from dim_aisles

select * from dim_fact_order_product

select asi.aisle, count(*) as higjest_no_re_order  from dim_fact_order_product fop join dim_aisles asi on fop.aisle_id=asi.aisle_id where fop.reordered=TRUE group by asi.aisle order by higjest_no_re_order desc limit 5 ;
```
![image](https://github.com/user-attachments/assets/f8f440b6-8b2b-4020-bbbf-f4d7cd80456b)

# Query to calculate the average number of products added to the cart per order by day of the week:

```
select o.order_dow , avg(fop.add_to_cart_order) as avg_prd_add_cart from dim_fact_order_product fop join dim_orders o on 
fop.order_id = o.order_id group by o.order_dow

```
![image](https://github.com/user-attachments/assets/690f75f5-838a-4f80-84a8-d62094981b77)

# Query to identify the top 10 users with the highest number of unique products ordered:
![image](https://github.com/user-attachments/assets/505ba0bd-c375-4cc0-8830-d69a1c82c829)


## Codes 

```
Orders Table:

Column Name	Data Type	Description
order_id	integer	Unique identifier for an order
user_id	integer	Unique identifier for a user
order_number	integer	A counter for the orders of a user
order_dow	integer	The day of the week the order was placed
order_hour_of_day	integer	The hour of the day the order was placed
days_since_prior_order	integer	Number of days since the previous order
Products Table:

Column Name	Data Type	Description
product_id	integer	Unique identifier for a product
product_name	varchar	Name of the product
aisle_id	integer	Unique identifier for an aisle
department_id	integer	Unique identifier for a department
Order Products Table:

Column Name	Data Type	Description
order_id	integer	Unique identifier for an order
product_id	integer	Unique identifier for a product
add_to_cart_order	integer	The order in which the product was added to the cart
reordered	boolean	Has the product been ordered by this user in the past?
Column Name	Data Type	Description
aisle_id	integer	Unique identifier for an aisle
aisle	varchar	Name of the aisle
Aisles Table:

Departments Table:

Column Name	Data Type	Description
department_id	integer	Unique identifier for a department
department	varchar	Name of the department
SQL Statements 
Building Stage and File Format

CREATE STAGE my_stage
URL = 's3://dw-snowflake-course-darshil/instacart/'
CREDENTIALS = (AWS_KEY_ID = '' AWS_SECRET_KEY = '');

CREATE OR REPLACE FILE FORMAT csv_file_format
TYPE = 'CSV'
FIELD_DELIMITER = ','
SKIP_HEADER = 1
FIELD_OPTIONALLY_ENCLOSED_BY='"'; -- If the CSV file has a header row, skip it

Aisles Table:


CREATE TABLE aisles (
        aisle_id INTEGER PRIMARY KEY,
        aisle VARCHAR
    );

COPY INTO aisles (aisle_id, aisle)
FROM @my_stage/aisles.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
Departments Table:

CREATE TABLE departments (
        department_id INTEGER PRIMARY KEY,
        department VARCHAR
    );

COPY INTO departments (department_id, department)
FROM @my_stage/departments.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Products Table:

CREATE OR REPLACE TABLE products (
        product_id INTEGER PRIMARY KEY,
        product_name VARCHAR,
        aisle_id INTEGER,
        department_id INTEGER
    );

COPY INTO products (product_id, product_name, aisle_id, department_id)
FROM @my_stage/products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Orders Table:

CREATE OR REPLACE TABLE orders (
        order_id INTEGER PRIMARY KEY,
        user_id INTEGER,
        eval_set STRING,
        order_number INTEGER,
        order_dow INTEGER,
        order_hour_of_day INTEGER,
        days_since_prior_order INTEGER
    );

COPY INTO orders (order_id, user_id, eval_set, order_number, order_dow, order_hour_of_day, days_since_prior_order)
FROM @my_stage/orders.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');

Order Products Table:


CREATE OR REPLACE TABLE order_products (
        order_id INTEGER,
        product_id INTEGER,
        add_to_cart_order INTEGER,
        reordered INTEGER,
        PRIMARY KEY (order_id, product_id)
    );
    
COPY INTO order_products (order_id, product_id, add_to_cart_order, reordered)
FROM @my_stage/order_products.csv
FILE_FORMAT = (FORMAT_NAME = 'csv_file_format');
Building Dim Fact Tables 
CREATE OR REPLACE TABLE dim_users AS (
  SELECT
    user_id
  FROM
    orders
);

CREATE OR REPLACE TABLE dim_products AS (
  SELECT
    product_id,
    product_name
  FROM
    products
);

CREATE OR REPLACE TABLE dim_aisles AS (
  SELECT
    aisle_id,
    aisle
  FROM
    aisles
);

CREATE OR REPLACE TABLE dim_departments AS (
  SELECT
    department_id,
    department
  FROM
    departments
);

CREATE OR REPLACE TABLE dim_orders AS (
  SELECT
    order_id,
    order_number,
    order_dow,
    order_hour_of_day,
    days_since_prior_order
  FROM
    orders
);

CREATE TABLE fact_order_products AS (
  SELECT
    op.order_id,
    op.product_id,
    o.user_id,
    p.department_id,
    p.aisle_id,
    op.add_to_cart_order,
    op.reordered
  FROM
    order_products op
  JOIN
    orders o ON op.order_id = o.order_id
  JOIN
    products p ON op.product_id = p.product_id
);
Analytics (change tables names) 
-- Query to calculate the total number of products ordered per department:
SELECT
  d.department,
  COUNT(*) AS total_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_departments d ON fop.department_id = d.department_id
GROUP BY
  d.department;

-- Query to find the top 5 aisles with the highest number of reordered products:
SELECT
  a.aisle,
  COUNT(*) AS total_reordered
FROM
  fact_order_products fop
JOIN
  dim_aisles a ON fop.aisle_id = a.aisle_id
WHERE
  fop.reordered = TRUE
GROUP BY
  a.aisle
ORDER BY
  total_reordered DESC
LIMIT 5;

-- Query to calculate the average number of products added to the cart per order by day of the week:
SELECT
  o.order_dow,
  AVG(fop.add_to_cart_order) AS avg_products_per_order
FROM
  fact_order_products fop
JOIN
  dim_orders o ON fop.order_id = o.order_id
GROUP BY
  o.order_dow;

-- Query to identify the top 10 users with the highest number of unique products ordered:
SELECT
  u.user_id,
  COUNT(DISTINCT fop.product_id) AS unique_products_ordered
FROM
  fact_order_products fop
JOIN
  dim_users u ON fop.user_id = u.user_id
GROUP BY
  u.user_id
ORDER BY
  unique_products_ordered DESC
LIMIT 10;

```
🙌 Contact

📧 Email: gandooriimran@gmail.com

WhatsApp: +91 9381838092



