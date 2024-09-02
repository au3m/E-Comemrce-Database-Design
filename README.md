# __db-mentorship-assignment 1__
### We will introduce solving for following tasks:
- Create the DB schema script with the following entities
![](https://github.com/au3m/db-mentorship-assignment/blob/main/assets/entities.png?raw=true)
- Identify the relationships between entities
- Draw the ERD diagram of this sample schema
- Write an SQL query to generate a daily report of the total revenue for a specific date.
- Write an SQL query to generate a monthly report of the top-selling products in a given month.
- Write a SQL query to retrieve a list of customers who have placed orders totaling more than $500 in the past month.
Include customer names and their total order amounts. [Complex query].
- How we can apply a denormalization mechanism on customer and order entities.
## DB schema script
```sql
CREATE DATABASE IF NOT EXISTS mentorship_ecommerce;
USE mentorship_ecommerce;

DROP TABLE IF EXISTS order_details;
DROP TABLE IF EXISTS product;
DROP TABLE IF EXISTS category;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS customer;

CREATE TABLE category (
	category_id INTEGER PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL
);
CREATE TABLE product (
	product_id INTEGER PRIMARY KEY,
    category_id INTEGER,
    product_name VARCHAR(50) NOT NULL,
    description TEXT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INTEGER NOT NULL CHECK (stock_quantity > 0),
    FOREIGN KEY (category_id) REFERENCES category(category_id)
    
);
CREATE TABLE customer (
	customer_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(50) NOT NULL
);
CREATE TABLE orders (
	order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_date DATETIME NOT NULL,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customer(customer_id)
);
CREATE TABLE order_details (
	order_detail_id INTEGER PRIMARY KEY,
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES product(product_id)
);
```

---
## ER diagram showing relationships
![](https://github.com/au3m/db-mentorship-assignment/blob/main/assets/mentorship-ecommerce%20erd.png?raw=true)

---
## Revenue - daily report
```sql
SELECT DATE(order_date), SUM(total_amount) AS total_revenue FROM orders
WHERE DATE(order_date) = '2024-08-02'
GROUP BY DATE(order_date)
ORDER BY DATE(order_date);
```
## Top Selling Product - monthly report
```sql
SELECT product.product_name, COUNT(order_details.product_id) AS top_selling_product, Month(orders.order_date) AS month
FROM
product JOIN order_details ON product.product_id = order_details.product_id
JOIN orders ON order_details.order_id = orders.order_id
WHERE
MONTH(orders.order_date) = '02'
GROUP BY month, product.product_name
ORDER BY top_selling_product DESC;
```
## List of Customers With Orders Totaling More Than $500 In the Past Month
```sql
SELECT CONCAT(customer.first_name + ' ' + customer.last_name) AS top_customer, sum(orders.total_amount) AS amount
FROM
customer JOIN orders ON customer.customer_id = orders.customer_id
WHERE YEAR(orders.order_date) = YEAR(NOW() - INTERVAL 1 MONTH) AND MONTH(orders.order_date) = MONTH(NOW() - INTERVAL 1 MONTH)
GROUP BY top_customer, orders.total_amount
HAVING amount > 500;
```
---

## Denormalization Mechanism on Customer and Order Entities
###### Achieving better performance for data which retrives using join operation between customer and order entities like our last example __"List of Customers With Orders Totaling More Than $500 In the Past Month"__ we can create new table with attributes (customer_name and spending_amount) and triger db to update this entity within specific interval of time to achieve reading like this report more effiecient without the burdens of joining process.
