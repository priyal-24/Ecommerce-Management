# 🛒 E-Commerce Order Management System (MySQL Database Project)

[![Database: MySQL](https://img.shields.io/badge/Database-MySQL_8.0+-4479A1?style=for-the-badge&logo=mysql&logoColor=white)](https://www.mysql.com/)
[![Environment: phpMyAdmin](https://img.shields.io/badge/Interface-phpMyAdmin-orange?style=for-the-badge&logo=phpmyadmin&logoColor=white)](https://www.phpmyadmin.net/)
[![Language: SQL](https://img.shields.io/badge/Language-SQL-CC292B?style=for-the-badge&logo=sqlite&logoColor=white)](https://en.wikipedia.org/wiki/SQL)
[![Status: Completed](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)](https://github.com/)

---

## 📌 Project Overview

The **E-Commerce Order Management System** is an enterprise-grade relational database project built using **MySQL**. It is engineered to simulate the complete backend operations of a modern online shopping platform.

The system manages the entire transactional lifecycle of an e-commerce platform—from **catalog and inventory indexing** to **customer acquisition, multi-item basket ordering, payment processing (UPI, Card, PayPal)**, and **last-mile logistics delivery tracking**.

### 🎯 Key Objectives
- Design a normalized (3NF) relational database schema across 7 core entities.
- Enforce strict data integrity using **Primary Keys**, **Foreign Keys**, `ENUM` types, and `NOT NULL` constraints.
- Implement comprehensive **CRUD** transactions.
- Solve critical business queries using **Aggregation**, **Multi-Table Joins (INNER, LEFT, FULL OUTER simulation)**, **Correlated Subqueries**, and **Temporal / String Transformations**.
- Implement advanced business intelligence analytics using **SQL Window Functions (`ROW_NUMBER`, `RANK`, Running Totals)** and conditional customer tiering via **SQL `CASE` Expressions**.

---

## 🏗 Database Schema & Entity Relationships

The relational architecture models a complete order lifecycle through 7 normalized tables:

```
[ Categories ]
      │ (1:N)
      ▼
 [ Products ] ◄─────┐ (1:N)
                    │
[ Customers ]       │
      │ (1:N)       │
      ▼             │
   [ Orders ]       │
      │ (1:N)       │
      ├────────► [ Order_Items ]
      │
      ├────────► [ Payments ] (1:N / 1:1)
      │
      └────────► [ Shipping ] (1:1 / 1:N)
```

### Table Metadata & Constraints Summary

| # | Entity / Table Name | Primary Key | Foreign Key Dependencies | Description |
|---|---------------------|-------------|--------------------------|-------------|
| 1 | `Categories` | `category_id` | *None* | Product classification taxonomy. |
| 2 | `Products` | `product_id` | `category_id` $\rightarrow$ `Categories` | Inventory catalog with unit price and stock. |
| 3 | `Customers` | `customer_id` | *None* | Registered customer profiles & contact details. |
| 4 | `Orders` | `order_id` | `customer_id` $\rightarrow$ `Customers` | Transaction headers recording timestamps & status. |
| 5 | `Order_Items` | `order_item_id` | `order_id` $\rightarrow$ `Orders`, `product_id` $\rightarrow$ `Products` | Line-item junction table storing units & subtotals. |
| 6 | `Payments` | `payment_id` | `order_id` $\rightarrow$ `Orders` | Financial transactions, gateway type, and status. |
| 7 | `Shipping` | `shipping_id` | `order_id` $\rightarrow$ `Orders` | Logistics tracking, dispatch/delivery dates, and status. |

---

## ⚙️ Database Definition Language (DDL)

```sql
-- Create Database
CREATE DATABASE IF NOT EXISTS ecommerce_management;
USE ecommerce_management;

-- 1. Categories Table
CREATE TABLE Categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL
);

-- 2. Products Table
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category_id INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT NOT NULL,
    added_date DATE NOT NULL,
    FOREIGN KEY (category_id) REFERENCES Categories(category_id)
);

-- 3. Customers Table
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150),
    phone_number VARCHAR(20),
    address VARCHAR(255),
    registration_date DATE NOT NULL
);

-- 4. Orders Table
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(12,2) NOT NULL,
    status ENUM('Pending', 'Shipped', 'Delivered', 'Cancelled') NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

-- 5. Order_Items Table
CREATE TABLE Order_Items (
    order_item_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    subtotal DECIMAL(12,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (product_id) REFERENCES Products(product_id)
);

-- 6. Payments Table
CREATE TABLE Payments (
    payment_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    payment_date DATE NOT NULL,
    payment_method ENUM('Credit Card', 'PayPal', 'UPI') NOT NULL,
    payment_status ENUM('Paid', 'Pending', 'Failed') NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- 7. Shipping Table
CREATE TABLE Shipping (
    shipping_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    shipping_date DATE,
    delivery_date DATE,
    shipping_status ENUM('Dispatched', 'In Transit', 'Delivered') NOT NULL,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);
```

---

## 📥 Data Seeding (DML)

```sql
-- 1. Categories
INSERT INTO Categories (category_id, category_name) VALUES
(1, 'Electronics'),
(2, 'Fashion'),
(3, 'Home Appliances'),
(4, 'Books'),
(5, 'Beauty');

-- 2. Products
INSERT INTO Products (product_id, name, category_id, price, stock_quantity, added_date) VALUES
(1, 'Laptop', 1, 65000.00, 20, '2026-01-10'),
(2, 'Smartphone', 1, 35000.00, 15, '2026-02-05'),
(3, 'Headphones', 1, 2500.00, 600, '2026-03-12'),
(4, 'T-Shirt', 2, 999.00, 120, '2026-01-20'),
(5, 'Jeans', 2, 2499.00, 80, '2026-02-15'),
(6, 'Air Fryer', 3, 8500.00, 35, '2026-03-01'),
(7, 'SQL Mastery Book', 4, 1800.00, 200, '2026-04-10'),
(8, 'Face Serum', 5, 2200.00, 75, '2026-05-05');

-- 3. Customers
INSERT INTO Customers (customer_id, name, email, phone_number, address, registration_date) VALUES
(1, 'Priya Shah', 'priya@example.com', '9876500001', 'Ahmedabad', '2021-08-15'),
(2, 'Rahul Patel', 'rahul@example.com', '9876500002', 'Ahmedabad', '2023-02-10'),
(3, 'Neha Mehta', NULL, '9876500003', 'Surat', '2024-06-20'),
(4, 'Amit Desai', 'amit@example.com', '9876500004', 'Vadodara', '2022-11-05'),
(5, 'Karan Joshi', 'karan@example.com', '9876500005', 'Rajkot', '2025-01-15'),
(6, 'Riya Shah', 'riya@example.com', '9876500006', 'Ahmedabad', '2022-04-18'),
(7, 'Dev Patel', 'dev@example.com', '9876500007', 'Gandhinagar', '2023-09-12'),
(8, 'Maya Trivedi', NULL, '9876500008', 'Anand', '2025-07-01');

-- 4. Orders
INSERT INTO Orders (order_id, customer_id, order_date, total_amount, status) VALUES
(101, 1, '2026-04-10', 65000.00, 'Delivered'),
(102, 1, '2026-05-15', 35000.00, 'Delivered'),
(103, 1, '2026-06-20', 8500.00, 'Shipped'),
(104, 1, '2026-08-05', 1800.00, 'Pending'),
(105, 2, '2026-04-25', 25000.00, 'Delivered'),
(106, 3, '2026-05-30', 999.00, 'Delivered'),
(107, 4, '2026-07-12', 17000.00, 'Shipped'),
(108, 5, '2026-08-18', 7000.00, 'Pending'),
(109, 6, '2026-09-01', 22000.00, 'Delivered'),
(110, 7, '2026-09-15', 5000.00, 'Cancelled'),
(111, 8, '2026-01-01', 12000.00, 'Cancelled'),
(112, 2, '2026-10-01', 4000.00, 'Pending');

-- 5. Order_Items
INSERT INTO Order_Items (order_item_id, order_id, product_id, quantity, subtotal) VALUES
(1, 101, 1, 1, 65000.00),
(2, 102, 2, 1, 35000.00),
(3, 103, 6, 1, 8500.00),
(4, 104, 7, 1, 1800.00),
(5, 105, 3, 10, 25000.00),
(6, 106, 4, 1, 999.00),
(7, 107, 6, 2, 17000.00),
(8, 108, 5, 2, 4998.00),
(9, 108, 3, 1, 2500.00),
(10, 109, 2, 1, 22000.00),
(11, 110, 3, 2, 5000.00),
(12, 111, 8, 1, 2200.00),
(13, 112, 7, 1, 1800.00),
(14, 112, 4, 2, 1998.00);

-- 6. Payments
INSERT INTO Payments (payment_id, order_id, payment_date, payment_method, payment_status) VALUES
(1, 101, '2026-04-10', 'Credit Card', 'Paid'),
(2, 102, '2026-05-15', 'UPI', 'Paid'),
(3, 103, '2026-06-20', 'UPI', 'Paid'),
(4, 104, '2026-08-05', 'Credit Card', 'Pending'),
(5, 105, '2026-04-25', 'PayPal', 'Paid'),
(6, 106, '2026-05-30', 'UPI', 'Paid'),
(7, 107, '2026-07-12', 'Credit Card', 'Paid'),
(8, 108, '2026-08-18', 'UPI', 'Pending'),
(9, 109, '2026-09-01', 'Credit Card', 'Paid'),
(10, 110, '2026-09-15', 'UPI', 'Failed'),
(11, 111, '2026-01-01', 'PayPal', 'Failed'),
(12, 112, '2026-10-01', 'UPI', 'Pending');

-- 7. Shipping
INSERT INTO Shipping (shipping_id, order_id, shipping_date, delivery_date, shipping_status) VALUES
(1, 101, '2026-04-11', '2026-04-15', 'Delivered'),
(2, 102, '2026-05-16', '2026-05-20', 'Delivered'),
(3, 103, '2026-06-21', NULL, 'In Transit'),
(4, 104, NULL, NULL, 'Dispatched'),
(5, 105, '2026-04-26', '2026-04-30', 'Delivered'),
(6, 106, '2026-05-31', '2026-06-03', 'Delivered'),
(7, 107, '2026-07-13', NULL, 'In Transit'),
(8, 108, NULL, NULL, 'Dispatched'),
(9, 109, '2026-09-02', '2026-09-06', 'Delivered'),
(10, 110, NULL, NULL, 'Dispatched');
```

---

## 📊 Practical Exam Tasks & Solution Queries

### Task 1: CRUD Operations
- **Insert New Product:**
  ```sql
  INSERT INTO Products (product_id, name, category_id, price, stock_quantity, added_date)
  VALUES (9, 'Smart Watch', 1, 5500.00, 50, '2026-10-06');
  ```
- **Update Stock / Order Status:**
  ```sql
  UPDATE Orders
  SET status = 'Shipped'
  WHERE order_id = 104;
  ```
- **Delete Cancelled Orders Older than 30 Days:**
  ```sql
  DELETE FROM Order_Items
  WHERE order_id IN (
      SELECT order_id
      FROM Orders
      WHERE status = 'Cancelled'
      AND order_date < CURRENT_DATE - INTERVAL 30 DAY
  );
  ```

---

### Task 2: SQL Clauses (`WHERE`, `HAVING`, `LIMIT`)
- **Find all orders placed in the last 6 months:**
  ```sql
  SELECT *
  FROM Orders
  WHERE order_date >= CURRENT_DATE - INTERVAL 6 MONTH;
  ```
- **Get the top 5 highest-priced products:**
  ```sql
  SELECT *
  FROM Products
  ORDER BY price DESC
  LIMIT 5;
  ```
- **Find customers who have placed more than 3 orders:**
  ```sql
  SELECT
      c.customer_id,
      c.name,
      COUNT(o.order_id) AS order_count
  FROM Customers c
  JOIN Orders o
      ON c.customer_id = o.customer_id
  GROUP BY c.customer_id, c.name
  HAVING COUNT(o.order_id) > 3;
  ```

---

### Task 3: SQL Operators (`AND`, `OR`, `NOT`)
- **Get all orders where status = 'Pending' AND payment_status = 'Paid':**
  ```sql
  SELECT
      o.order_id,
      o.status,
      p.payment_status
  FROM Orders o
  JOIN Payments p
      ON o.order_id = p.order_id
  WHERE o.status = 'Pending'
  AND p.payment_status = 'Paid';
  ```
- **Find all products that are NOT out of stock:**
  ```sql
  SELECT *
  FROM Products
  WHERE NOT stock_quantity = 0;
  ```
- **Retrieve customers who registered after 2022 OR have spent above ₹10,000:**
  ```sql
  SELECT
      c.customer_id,
      c.name,
      c.registration_date,
      COALESCE(SUM(o.total_amount), 0) AS total_spent
  FROM Customers c
  LEFT JOIN Orders o
      ON c.customer_id = o.customer_id
  GROUP BY
      c.customer_id,
      c.name,
      c.registration_date
  HAVING c.registration_date > '2022-12-31'
      OR COALESCE(SUM(o.total_amount), 0) > 10000;
  ```

---

### Task 4: Sorting & Grouping (`ORDER BY`, `GROUP BY`)
- **List all products sorted by price in descending order:**
  ```sql
  SELECT *
  FROM Products
  ORDER BY price DESC;
  ```
- **Display the number of orders placed by each customer:**
  ```sql
  SELECT
      c.customer_id,
      c.name,
      COUNT(o.order_id) AS total_orders
  FROM Customers c
  LEFT JOIN Orders o
      ON c.customer_id = o.customer_id
  GROUP BY c.customer_id, c.name;
  ```
- **Show total revenue generated per category:**
  ```sql
  SELECT
      cat.category_name,
      SUM(oi.subtotal) AS total_revenue
  FROM Categories cat
  JOIN Products p
      ON cat.category_id = p.category_id
  JOIN Order_Items oi
      ON p.product_id = oi.product_id
  JOIN Orders o
      ON oi.order_id = o.order_id
  WHERE o.status <> 'Cancelled'
  GROUP BY cat.category_id, cat.category_name;
  ```

---

### Task 5: Aggregate Functions (`SUM`, `AVG`, `MAX`, `MIN`, `COUNT`)
- **Find total platform revenue:**
  ```sql
  SELECT SUM(total_amount) AS total_revenue
  FROM Orders
  WHERE status <> 'Cancelled';
  ```
- **Identify the most purchased product (by quantity):**
  ```sql
  SELECT
      p.product_id,
      p.name,
      SUM(oi.quantity) AS units_sold
  FROM Products p
  JOIN Order_Items oi
      ON p.product_id = oi.product_id
  JOIN Orders o
      ON oi.order_id = o.order_id
  WHERE o.status <> 'Cancelled'
  GROUP BY p.product_id, p.name
  ORDER BY units_sold DESC
  LIMIT 1;
  ```
- **Calculate average order value (AOV):**
  ```sql
  SELECT AVG(total_amount) AS average_order_value
  FROM Orders
  WHERE status <> 'Cancelled';
  ```

---

### Task 6: Primary & Foreign Key Constraint Verification
- **Verify Primary Keys in Schema:**
  ```sql
  SELECT
      TABLE_NAME,
      COLUMN_NAME,
      CONSTRAINT_NAME
  FROM information_schema.KEY_COLUMN_USAGE
  WHERE TABLE_SCHEMA = 'ecommerce_management'
  AND CONSTRAINT_NAME = 'PRIMARY';
  ```
- **Verify Foreign Key Referential Integrity:**
  ```sql
  SELECT
      TABLE_NAME,
      COLUMN_NAME,
      CONSTRAINT_NAME,
      REFERENCED_TABLE_NAME,
      REFERENCED_COLUMN_NAME
  FROM information_schema.KEY_COLUMN_USAGE
  WHERE TABLE_SCHEMA = 'ecommerce_management'
  AND REFERENCED_TABLE_NAME IS NOT NULL;
  ```

---

### Task 7: Relational Joins (`INNER`, `LEFT`, Multi-Table)
- **INNER JOIN: Orders with customer details:**
  ```sql
  SELECT
      o.order_id,
      c.name AS customer_name,
      o.order_date,
      o.total_amount
  FROM Orders o
  INNER JOIN Customers c
      ON o.customer_id = c.customer_id;
  ```
- **LEFT JOIN: All customers and their order status (including non-buyers):**
  ```sql
  SELECT
      c.customer_id,
      c.name AS customer_name,
      o.order_id,
      o.status
  FROM Customers c
  LEFT JOIN Orders o
      ON c.customer_id = o.customer_id;
  ```
- **Multi-Table JOIN: Order ledger with full product line-items:**
  ```sql
  SELECT
      o.order_id,
      c.name AS customer_name,
      p.name AS product_name,
      oi.quantity,
      oi.subtotal
  FROM Orders o
  JOIN Customers c
      ON o.customer_id = c.customer_id
  JOIN Order_Items oi
      ON o.order_id = oi.order_id
  JOIN Products p
      ON oi.product_id = p.product_id;
  ```
- **INNER JOIN: Products with Category classification:**
  ```sql
  SELECT
      p.product_id,
      p.name AS product_name,
      cat.category_name,
      p.price
  FROM Products p
  JOIN Categories cat
      ON p.category_id = cat.category_id;
  ```

---

### Task 8: Subqueries
- **Find products priced above the catalog average price:**
  ```sql
  SELECT product_id, name, price
  FROM Products
  WHERE price > (
      SELECT AVG(price)
      FROM Products
  );
  ```
- **Identify the customer who has placed the highest number of orders:**
  ```sql
  SELECT customer_id, name
  FROM Customers
  WHERE customer_id = (
      SELECT customer_id
      FROM Orders
      GROUP BY customer_id
      ORDER BY COUNT(order_id) DESC
      LIMIT 1
  );
  ```
- **Retrieve orders greater than overall average order amount:**
  ```sql
  SELECT order_id, customer_id, total_amount
  FROM Orders
  WHERE total_amount > (
      SELECT AVG(total_amount)
      FROM Orders
  );
  ```

---

### Task 9: Date & Time Functions
- **System runtime date and timestamp:**
  ```sql
  SELECT
      CURRENT_DATE AS today_date,
      CURRENT_TIME AS current_time_value;
  ```
- **Extract Year, Month, and Day from `order_date`:**
  ```sql
  SELECT
      order_id,
      order_date,
      YEAR(order_date) AS order_year,
      MONTH(order_date) AS order_month,
      DAY(order_date) AS order_day
  FROM Orders;
  ```
- **Calculate elapsed days since transaction placement:**
  ```sql
  SELECT
      order_id,
      order_date,
      DATEDIFF(CURRENT_DATE, order_date) AS days_since_order
  FROM Orders;
  ```

---

### Task 10: String Manipulation Functions
- **Convert customer names to uppercase:**
  ```sql
  SELECT customer_id, name, UPPER(name) AS uppercase_name
  FROM Customers;
  ```
- **Calculate character length of product titles:**
  ```sql
  SELECT product_id, name, LENGTH(name) AS name_length
  FROM Products;
  ```
- **Concatenate customer name and address:**
  ```sql
  SELECT customer_id, CONCAT(name, ' - ', address) AS customer_details
  FROM Customers;
  ```

---

### Task 11: SQL Window Functions
- **Rank orders based on total transaction amount (`ROW_NUMBER`):**
  ```sql
  SELECT
      order_id,
      customer_id,
      total_amount,
      ROW_NUMBER() OVER (ORDER BY total_amount DESC) AS order_rank
  FROM Orders;
  ```
- **Rank catalog merchandise by unit price (`RANK`):**
  ```sql
  SELECT
      product_id,
      name,
      price,
      RANK() OVER (ORDER BY price DESC) AS price_rank
  FROM Products;
  ```
- **Calculate chronological running total of revenue (`SUM OVER`):**
  ```sql
  SELECT
      order_id,
      order_date,
      total_amount,
      SUM(total_amount) OVER (
          ORDER BY order_date
      ) AS running_total
  FROM Orders;
  ```

---

### Task 12: Conditional `CASE` Expressions
- **Assign Customer Loyalty Status based on lifetime spend:**
  - **Gold:** $> ₹50,000$
  - **Silver:** Between $₹20,000$ and $₹50,000$
  - **Bronze:** $< ₹20,000$
  ```sql
  SELECT
      c.customer_id,
      c.name,
      COALESCE(SUM(o.total_amount), 0) AS total_spent,
      CASE
          WHEN COALESCE(SUM(o.total_amount), 0) > 50000 THEN 'Gold'
          WHEN COALESCE(SUM(o.total_amount), 0) BETWEEN 20000 AND 50000 THEN 'Silver'
          ELSE 'Bronze'
      END AS Loyalty_Status
  FROM Customers c
  LEFT JOIN Orders o
      ON c.customer_id = o.customer_id
      AND o.status <> 'Cancelled'
  GROUP BY c.customer_id, c.name;
  ```
- **Categorize products based on sales velocity:**
  - **Best Seller:** $> 500$ units sold
  - **Popular:** Between $200$ and $500$ units sold
  - **Regular:** $< 200$ units sold
  ```sql
  SELECT
      p.product_id,
      p.name,
      COALESCE(SUM(oi.quantity), 0) AS units_sold,
      CASE
          WHEN COALESCE(SUM(oi.quantity), 0) > 500 THEN 'Best Seller'
          WHEN COALESCE(SUM(oi.quantity), 0) BETWEEN 200 AND 500 THEN 'Popular'
          ELSE 'Regular'
      END AS Product_Category
  FROM Products p
  LEFT JOIN Order_Items oi
      ON p.product_id = oi.product_id
  LEFT JOIN Orders o
      ON oi.order_id = o.order_id
      AND o.status <> 'Cancelled'
  GROUP BY p.product_id, p.name;
  ```

---

## 💻 How to Run the Project Locally

1. **Clone the repository:**
   ```bash
   git clone https://github.com/priyal-24/Ecommerce-Management.git
   cd Ecommerce-Management
   ```

2. **Start MySQL Server:**
   - Launch your **XAMPP / WAMP** control panel and start the **Apache** and **MySQL** modules.

3. **Import Database:**
   - Open your browser and navigate to `http://localhost/phpmyadmin`.
   - Click on the **Import** tab.
   - Choose `Ecommerce.sql` (or `ecommerce_management.sql`) from this repository.
   - Click **Go** to build the database schema and seed the initial dataset.

4. **Execute Analytics Queries:**
   - Open the **SQL** console inside `ecommerce_management` and run any of the Task 1 to Task 12 queries directly.

---

## 🏆 Project Achievements & Learnings
- **Schema Normalization:** Designed a clean, non-redundant database architecture adhering to 3NF standards.
- **Relational Integrity:** Implemented foreign keys across 6 relationships preventing orphaned transaction entries.
- **Analytical Competence:** Successfully orchestrated window functions (`SUM OVER`, `RANK`), multi-condition `CASE` tiering, and subquery logic.
- **Complete Test Coverage:** Every query has been validated on MySQL 8.0 / phpMyAdmin with documented output logs.