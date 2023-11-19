# Online Commerce Company
## Introduction
Project made for my Database class at Instituto Superior Técnico. The project consists of a database for an online commerce company. The database is made in PostgreSQL and the application is made in Python using the Flask framework. The database runs on a docker container inside the workspace folder.

## Application Development

### Explanation of the web application architecture.

The app starts with an initial page (index.html) with a button that redirects to the login page. In the login page (login_page.html), the user can log in as a manager or with one of the customer accounts.

In the customer app part, the user can only create and pay for their corresponding orders (orders_index_private.html, order_payment, add_order.html).

In the manager app part, there is a navbar with options: Customers, Orders, Products, Suppliers (navbar.html). In the customers section, all customers can be viewed, deleted, or new ones can be created (/customer). In the orders section, the orders of each customer can be viewed, and it can be checked whether they have been paid or not (order_customer.html, orders_index.html). In the products section, all products can be viewed and new ones can be created. It is possible to edit or view all details. Deletion is also possible (/products). In the suppliers section, all suppliers can be viewed and deleted or new ones can be created (/supplier).

In addition to this, we implemented pagination, input verification, and tried to ensure the security of the app through the implementation of transactions.

To customize the app, we used bootstrap.


# Database

## Database Initialization

### Initialize tables:
```sql
DROP TABLE IF EXISTS customer CASCADE;
DROP TABLE IF EXISTS orders CASCADE;
DROP TABLE IF EXISTS pay CASCADE;
DROP TABLE IF EXISTS employee CASCADE;
DROP TABLE IF EXISTS process CASCADE;
DROP TABLE IF EXISTS department CASCADE;
DROP TABLE IF EXISTS workplace CASCADE;
DROP TABLE IF EXISTS works CASCADE;
DROP TABLE IF EXISTS office CASCADE;
DROP TABLE IF EXISTS warehouse CASCADE;
DROP TABLE IF EXISTS product CASCADE;
DROP TABLE IF EXISTS contains CASCADE;
DROP TABLE IF EXISTS supplier CASCADE;
DROP TABLE IF EXISTS delivery CASCADE;

CREATE TABLE customer(
  cust_no INTEGER PRIMARY KEY,
  name VARCHAR(80) NOT NULL,
  email VARCHAR(254) UNIQUE NOT NULL,
  phone VARCHAR(15),
  address VARCHAR(255)
);

CREATE TABLE orders(
  order_no INTEGER PRIMARY KEY,
  cust_no INTEGER NOT NULL REFERENCES customer,
  date DATE NOT NULL
  --order_no must exist in contains
);

CREATE TABLE pay(
  order_no INTEGER PRIMARY KEY REFERENCES orders,
  cust_no INTEGER NOT NULL REFERENCES customer
);

CREATE TABLE employee(
  ssn VARCHAR(20) PRIMARY KEY,
  TIN VARCHAR(20) UNIQUE NOT NULL,
  bdate DATE,
  name VARCHAR NOT NULL
  --age must be >=18
);

CREATE TABLE process(
  ssn VARCHAR(20) REFERENCES employee,
  order_no INTEGER REFERENCES orders,
  PRIMARY KEY (ssn, order_no)
);

CREATE TABLE department(
  name VARCHAR PRIMARY KEY
);

CREATE TABLE workplace(
  address VARCHAR PRIMARY KEY,
  lat NUMERIC(8, 6) NOT NULL,
  long NUMERIC(9, 6) NOT NULL,
  UNIQUE(lat, long)
  --address must be in warehouse or office but not both
);

CREATE TABLE office(
  address VARCHAR(255) PRIMARY KEY REFERENCES workplace
);

CREATE TABLE warehouse(
  address VARCHAR(255) PRIMARY KEY REFERENCES workplace
);

CREATE TABLE works(
  ssn VARCHAR(20) REFERENCES employee,
  name VARCHAR(200) REFERENCES department,
  address VARCHAR(255) REFERENCES workplace,
  PRIMARY KEY (ssn, name, address)
);

CREATE TABLE product(
  SKU VARCHAR(25) PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  description VARCHAR,
  price NUMERIC(10, 2) NOT NULL,
  ean NUMERIC(13) UNIQUE
);

CREATE TABLE contains(
  order_no INTEGER REFERENCES orders,
  SKU VARCHAR(25) REFERENCES product,
  qty INTEGER,
  PRIMARY KEY (order_no, SKU)
);

CREATE TABLE supplier(
  TIN VARCHAR(20) PRIMARY KEY,
  name VARCHAR(200),
  address VARCHAR(255),
  SKU VARCHAR(25) REFERENCES product,
  date DATE
);

CREATE TABLE delivery(
  address VARCHAR(255) REFERENCES warehouse ,
  TIN VARCHAR(20) REFERENCES supplier,
  PRIMARY KEY (address, TIN)
);
```

### Example initial data:
```sql
insert into customer (cust_no, name, email, phone, address) VALUES (-1,'Deleted_User', '', '', '');
insert into customer (cust_no, name, email, phone, address) VALUES (1,'Belchior', 'belchiorlindo@gmail.com', '912345678', 'Avenida da Liberdade, 2000-002 Póvoa de Varzim');
insert into customer (cust_no, name, email, phone, address) VALUES (2,'Nunes', 'danielnunes@gmail.com', '939393939', 'Travessa dos Pescadores, 3000-003 Coimbra');
insert into customer (cust_no, name, email, phone, address) VALUES (3,'Costa', 'joaolscosta@gmail.com', '947854684', 'Rua das Flores, 1000-001 Lisboa');


insert into orders (order_no, date, cust_no) VALUES (1,'2017-01-01', (select cust_no from customer where name = 'Belchior'));
insert into orders (order_no, date, cust_no) VALUES (2,'2018-01-01', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (3,'2019-01-01', (select cust_no from customer where name = 'Costa'));
insert into orders (order_no, date, cust_no) VALUES (4,'2014-01-01', (select cust_no from customer where name = 'Costa'));
insert into orders (order_no, date, cust_no) VALUES (5,'2023-01-02', (select cust_no from customer where name = 'Costa'));
insert into orders (order_no, date, cust_no) VALUES (6,'2023-02-02', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (7,'2023-01-03', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (8,'2022-01-03', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (9,'2022-01-04', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (10,'2022-01-04', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (11,'2022-03-04', (select cust_no from customer where name = 'Nunes'));
insert into orders (order_no, date, cust_no) VALUES (12,'2022-10-04', (select cust_no from customer where name = 'Nunes'));




insert into pay (order_no, cust_no) VALUES ((select order_no from orders where date = '2017-01-01'), (select cust_no from customer where name = 'Belchior'));
insert into pay (order_no, cust_no) VALUES ((select order_no from orders where date = '2018-01-01'), (select cust_no from customer where name = 'Nunes'));
insert into pay (order_no, cust_no) VALUES (8, 2);
insert into pay (order_no, cust_no) VALUES (9, 2);

insert into employee (ssn, TIN, bdate, name) VALUES ('546654654', '123123123', '2002-04-10', 'Pedrinho');
insert into employee (ssn, TIN, bdate, name) VALUES ('546654655', '123123124', '2002-02-14', 'Gui');
insert into employee (ssn, TIN, bdate, name) VALUES ('546654656', '123123125', '2002-01-01', 'Luana');

insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Pedrinho'), (select order_no from orders where date = '2017-01-01'));
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Luana'), (select order_no from orders where date = '2023-01-02'));
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Gui'), (select order_no from orders where date = '2019-01-01'));
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Gui'), (select order_no from orders where date = '2023-01-03'));
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Pedrinho'), (select order_no from orders where date = '2014-01-01'));
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Gui'), 11);
insert into process (ssn, order_no) VALUES ((select ssn from employee where name = 'Gui'), 12);


insert into department (name) VALUES ('Marketing');
insert into department (name) VALUES ('Production');

insert into workplace (address, lat, long) VALUES ('Rua das Flores, 1000-001 Lisboa', '12.000000', '45.000000');
insert into workplace (address, lat, long) VALUES ('Rua do Comércio, 4000-004 Porto', '78.000000', '10.000000');
insert into workplace (address, lat, long) VALUES ('Praça da República, 5000-005 Vila Real', '78.000000', '13.000000');
insert into workplace (address, lat, long) VALUES ('Avenida Central, 6000-006 Castelo Branco', '78.000000', '11.000000');


insert into office (address) VALUES ((select address from workplace where address = 'Praça da República, 5000-005 Vila Real'));
insert into office (address) VALUES ((select address from workplace where address = 'Avenida Central, 6000-006 Castelo Branco'));

insert into warehouse (address) VALUES ((select address from workplace where address = 'Rua das Flores, 1000-001 Lisboa'));
insert into warehouse (address) VALUES ((select address from workplace where address = 'Rua do Comércio, 4000-004 Porto'));


insert into works (ssn, name, address) VALUES ((select ssn from employee where name = 'Pedrinho'), (select name from department where name = 'Marketing'), (select address from workplace where address = 'Praça da República, 5000-005 Vila Real'));
insert into works (ssn, name, address) VALUES ((select ssn from employee where name = 'Luana'), (select name from department where name = 'Marketing'), (select address from workplace where address = 'Rua das Flores, 1000-001 Lisboa'));
insert into works (ssn, name, address) VALUES ((select ssn from employee where name = 'Gui'), (select name from department where name = 'Production'), (select address from workplace where address = 'Rua das Flores, 1000-001 Lisboa'));
insert into works (ssn, name, address) VALUES ((select ssn from employee where name = 'Gui'), (select name from department where name = 'Production'), (select address from workplace where address = 'Avenida Central, 6000-006 Castelo Branco'));


insert into product (SKU, name, description, price, ean) VALUES ('-1', '', '',0, '-1');
insert into product (SKU, name, description, price, ean) VALUES ('EI01TS01EI01TS01EI01TS011', 'Gaming Chair', 'Chair made with love and kindness', 5.35, '1234567891234');
insert into product (SKU, name, description, price, ean) VALUES ('EI01TS01EI01TS01EI01TS012', 'Cake', 'Cake made by our amazing chef Luana', 10.35, '1234567891235');
insert into product (SKU, name, description, price, ean) VALUES ('EI01TS01EI01TS01EI01TS013', 'TV', 'LG TV 4K', 500.35, '1234567891236');
insert into product (SKU, name, description, price, ean) VALUES ('EI01TS01EI01TS01EI01TS014', 'Table', 'Table made with paus and cola', 50.35, '1234567891237');



insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'Gaming Chair'), (select order_no from orders where date = '2017-01-01'), 2);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'Cake'), (select order_no from orders where date = '2018-01-01'), 10);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), (select order_no from orders where date = '2019-01-01'), 3);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), (select order_no from orders where date = '2023-01-02'), 14);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), (select order_no from orders where date = '2023-01-03'), 3);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 8, 14);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 9, 20);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 10, 2);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 11, 2);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 12, 2);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 4, 14);
insert into contains (SKU, order_no, qty) VALUES ((select SKU from product where name = 'TV'), 6, 14);


insert into supplier (TIN, name, address, SKU, date) VALUES ('123456789', 'Goncalo', 'Rua das Palmeiras, 7000-007 Évora', (select SKU from product where name = 'Gaming Chair'), '2017-01-01');
insert into supplier (TIN, name, address, SKU, date) VALUES ('123456788', 'Rafael', 'Avenida do Mar, 8000-008 Faro', (select SKU from product where name = 'Cake'), '2018-01-01');
insert into supplier (TIN, name, address, SKU, date) VALUES ('123456787', 'Luana', 'Travessa das Oliveiras, 9000-009 Funchal', (select SKU from product where name = 'TV'), '2019-01-01');
insert into supplier (TIN, name, address, SKU, date) VALUES ('123456786', 'Ricky', 'Rua dos Castanheiros, 1000-010 Guarda', (select SKU from product where name = 'TV'), '2023-01-02');



insert into delivery (address, TIN) VALUES ((select address from warehouse where address = 'Rua das Flores, 1000-001 Lisboa'), (select TIN from supplier where name = 'Goncalo'));
insert into delivery (address, TIN) VALUES ((select address from warehouse where address = 'Rua do Comércio, 4000-004 Porto'), (select TIN from supplier where name = 'Luana'));
insert into delivery (address, TIN) VALUES ((select address from warehouse where address = 'Rua das Flores, 1000-001 Lisboa'), (select TIN from supplier where name = 'Rafael'));
insert into delivery (address, TIN) VALUES ((select address from warehouse where address = 'Rua do Comércio, 4000-004 Porto'), (select TIN from supplier where name = 'Ricky'));
```

## Integrity Constraints

No employee can be younger than 18 years old.
```sql
CREATE OR REPLACE FUNCTION ValidAge() RETURNS TRIGGER AS
$$
  BEGIN
    IF ((CURRENT_DATE - 6570) < NEW.bdate ) THEN
      RAISE EXCEPTION 'Employees must be 18 years old';
    END IF;
    RETURN NEW;
  END
$$ LANGUAGE plpgsql;

CREATE TRIGGER CheckValidAge
BEFORE INSERT OR UPDATE ON employee
FOR EACH ROW EXECUTE PROCEDURE ValidAge();
```

A 'Workplace' must be either an 'Office' or 'Warehouse' but not both.
```sql
CREATE OR REPLACE FUNCTION CheckValidWorkplace() RETURNS TRIGGER AS
$$
  BEGIN
    IF (NEW.address NOT IN (SELECT address FROM Office) AND
        NEW.address NOT IN (SELECT address FROM Warehouse)) 
      OR
       ( NEW.address IN (SELECT address FROM Office) AND
         NEW.address IN (SELECT address FROM Warehouse))  
      THEN
      RAISE EXCEPTION 'Workplace must be either an Office or a Warehouse and not Both!';
    END IF;
    RETURN NEW;
  END
$$ LANGUAGE plpgsql;

CREATE TRIGGER CheckValidWorkplace
BEFORE INSERT OR UPDATE ON Workplace
FOR EACH ROW EXECUTE FUNCTION CheckValidWorkplace();
```

An 'Order' must appear in 'Contains'.
```sql
CREATE OR REPLACE FUNCTION CheckValidOrder() RETURNS TRIGGER AS
$$
  BEGIN
    IF (NEW.order_no NOT IN (SELECT order_no FROM contains)) THEN
      RAISE EXCEPTION 'Orders must have at least one product!';
    END IF;
    RETURN NEW;
  END
$$ LANGUAGE plpgsql;

CREATE CONSTRAINT TRIGGER CheckValidOrder
AFTER INSERT OR UPDATE ON orders 
DEFERRABLE 
INITIALLY DEFERRED
FOR EACH ROW EXECUTE PROCEDURE CheckValidOrder();
```

## Some example queries

What is the number and name(s) of the customer(s) with the highest total value of paid orders?
```sql
SELECT c.name, c.cust_no
FROM customer c
JOIN pay p ON c.cust_no = p.cust_no
GROUP BY c.cust_no, name HAVING COUNT(*) >= ALL (
    SELECT COUNT(*)
    FROM customer c
    JOIN pay p ON c.cust_no = p.cust_no
    GROUP BY c.cust_no, name
);
```

What are the names of the employees who processed orders on every day in 2022 when there were orders?
```sql
SELECT DISTINCT name
FROM Employee e1
JOIN process USING (ssn)
JOIN pay p1 USING (order_no)
JOIN orders o1 USING (order_no)
WHERE NOT EXISTS (
    SELECT DISTINCT date
      FROM orders o2
      JOIN pay p2 USING(order_no)
      WHERE o1.date = o2.date
    EXCEPT
    SELECT DISTINCT date
      FROM orders
      WHERE date < '2023-01-01'
      AND date > '2021-12-31'
);
```

How many orders were placed but not paid in each month of 2022?
```sql
SELECT 
  EXTRACT(MONTH FROM date) as Month, COUNT(*) as to_pay
FROM orders o
WHERE o.order_no NOT IN (
  SELECT order_no
  FROM pay
)
AND date < '2023-01-01'
AND date > '2021-12-31'
GROUP BY
  EXTRACT(MONTH FROM date)
;
```

## Views

```sql
CREATE VIEW product_sales AS
SELECT 
  c.SKU, 
  c.order_no, 
  c.qty, 
  c.qty * p.price AS total_price, 
  EXTRACT(YEAR FROM o.date) AS year,
  EXTRACT(MONTH FROM o.date) AS month,
  EXTRACT(DAY FROM o.date) AS day_of_month,
  EXTRACT(DOW FROM o.date) AS day_of_week,
  SUBSTRING(SUBSTRING(cst.address FROM '[0-9]{3} .+') FROM '[[:<:]][A-Z].+') AS city
  FROM contains c
  JOIN orders o USING (order_no)
  JOIN product p USING (SKU)
  JOIN customer cst USING (cust_no)
  ;
```

## OLAP Queries

Auxiliary table:
```sql
CREATE TABLE data AS
SELECT 
  DATE '2022-01-01' + SEQUENCE.DAY AS date
FROM 
  GENERATE_SERIES(0, 365) AS SEQUENCE(DAY);
```

The total quantity of products sold for each product and for each week of 2022.
```sql
SELECT 
  p.name AS product_name,
  ps.city,
  EXTRACT(MONTH FROM d.date) AS month,
  EXTRACT(DAY FROM d.date) AS day_of_month,
  EXTRACT(DOW FROM d.date) AS day_of_week,
  SUM(ps.qty) AS total_quantity,
  SUM(ps.total_price) AS total_value
FROM 
  product_sales ps
JOIN 
  product p ON ps.SKU = p.SKU
RIGHT JOIN 
  data d ON (ps.year = EXTRACT(YEAR FROM d.date) AND ps.month = EXTRACT(MONTH FROM d.date) AND ps.day_of_month = EXTRACT(DAY FROM d.date))
WHERE 
  EXTRACT(YEAR FROM d.date) = 2022 AND TOTAL_PRICE IS NOT NULL
GROUP BY 
  GROUPING SETS ((p.name, ps.city, EXTRACT(MONTH FROM d.date), EXTRACT(DAY FROM d.date), EXTRACT(DOW FROM d.date)), ())
ORDER BY 
  p.name, ps.city, EXTRACT(MONTH FROM d.date), EXTRACT(DAY FROM d.date), EXTRACT(DOW FROM d.date);
```

The average total price of products sold per day of the week for each month of 2022.
```sql
SELECT 
  EXTRACT(MONTH FROM d.date) AS month,
  EXTRACT(DOW FROM d.date) AS day_of_week,
  AVG(ps.total_price) AS average_daily_sales
FROM 
  product_sales ps
RIGHT JOIN 
  data d ON (ps.year = EXTRACT(YEAR FROM d.date) AND ps.month = EXTRACT(MONTH FROM d.date) AND ps.day_of_month = EXTRACT(DAY FROM d.date))
WHERE 
  EXTRACT(YEAR FROM d.date) = 2022 AND ps.total_price IS NOT NULL
GROUP BY 
  GROUPING SETS ((EXTRACT(MONTH FROM d.date), EXTRACT(DOW FROM d.date)), ())
ORDER BY 
  EXTRACT(MONTH FROM d.date), EXTRACT(DOW FROM d.date);
```
