### mysql-init.sq
```sql
CREATE TABLE shipments (
    shipment_id INT,
    customer_id INT,
    product_name VARCHAR(100)
);

INSERT INTO shipments VALUES
(1001,1,'Laptop'),
(1002,2,'Phone'),
(1003,3,'Tablet');
```
- shipments → The name of the table where shipment data will be stored.
- Data Types
  - INT → Stores numbers (1, 2, 1001 etc.)
  - VARCHAR(100) → Stores text up to 100 characters

### Important
- CREATE TABLE → Defines table structure
- INSERT INTO → Adds records into table
- VARCHAR(100) → Variable string up to 100 characters
- Multiple rows can be inserted in one INSERT statement
- Number of inserted values must match the number of table columns unless columns are explicitly specified.
  ```sql
  INSERT INTO shipments (customer_id, shipment_id, product_name, location)
  VALUES (1,1001,'Laptop','Chennai');
  ```
### postgres-init.sql

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100),
    country VARCHAR(50)
);

INSERT INTO customers (customer_name, country)
VALUES
('Amazon', 'USA'),
('Flipkart', 'India'),
('DHL', 'Germany');
```

| Column        | Data Type    | Meaning                      |
| ------------- | ------------ | ---------------------------- |
| id            | SERIAL       | Auto-increment customer ID   |
| customer_name | VARCHAR(100) | Name of the customer/company |
| country       | VARCHAR(50)  | Customer's country           |

- SERIAL means the database automatically generates numbers.

### What is PRIMARY KEY?
- PRIMARY KEY means:
  - Values must be unique
  - Values cannot be NULL
  - Used to identify each row uniquely
