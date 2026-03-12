# QA

## 1. docker exec -it trino /bin/bash and docker exec -it trino trino?
```
docker exec → Run a command in a running container
-it → Interactive terminal
trino → Container name
/bin/bash → Start Bash shell

docker exec → Execute command in container
-it → Interactive terminal
trino → Container name
trino → The Trino CLI program
```


## 2. Trino
```
It is a distributed SQL query engine, and the Trino CLI is just a tool to send SQL queries to the Trino server.
Trino acts like a single SQL layer over multiple data systems.
A Trino server is simply the running Trino service that processes queries.

1️⃣ Coordinator
The brain of the cluster.
Responsibilities:
  Accepts queries from CLI / JDBC / BI tools
  Parses SQL - Parsing a query means analyzing the SQL syntax and converting it into an internal structure so the query engine can understand and execute it.
  Creates execution plan
  Distributes tasks to workers # Trino distributes work by breaking a query into stages and tasks. The coordinator schedules these tasks across worker nodes, which process data in parallel and return results back to the coordinator.
  Collects results

2️⃣ Worker Nodes
Workers actually process the data.

Responsibilities:
  Execute query tasks
  Read data from sources (Hive, MySQL, S3 etc.)
  Send results back to coordinator
```

## 3. Command to check trino cluster status
```
docker exec -it trino trino
SELECT * FROM system.runtime.nodes;
curl http://localhost:8080/v1/info
or we check through UI
```
Why trino requires username why it is not asking password ?

why we have to use trino what about ETL Pipelines?

How trino will connect to multiple DB's?

Why we have to choose trino over other SQL Federation tools?

Advantages of trino and disadvantages of trino

any reliability concerns related trino?

where they are using trino and why?

Use of trino UI?

Why do we need metabase?

How metabase connects to trino?

How trino filter DB queries for cross databases?

for the first time if we run metabase its asking me details like mail name etc?

In metabase setup at the start in add data it is showing me trino and athena, MySQL etc why do i need choose trino?

connection string, display name, schema username, password host port catalog what is the use of filling this details?

Use a secure connection (SSL)
Use an SSH-tunnel what why do we need?

How to build metabase ?

Use of SSL in metabase?


Explain me below
CREATE TABLE shipments (
    shipment_id INT,
    customer_id INT,
    product_name VARCHAR(100)
);

INSERT INTO shipments VALUES
(1001,1,'Laptop'),
(1002,2,'Phone'),
(1003,3,'Tablet');


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



Explain me below?
services:

  trino:
    image: trinodb/trino
    container_name: trino
    ports:
      - "8080:8080"
    volumes:
      - ./trino:/etc/trino

  postgres:
    image: postgres:14
    container_name: postgres
    environment:
      POSTGRES_USER: trino
      POSTGRES_PASSWORD: trino
      POSTGRES_DB: logistics
    ports:
      - "5432:5432"
    volumes:
      - ./init-db/postgres-init.sql:/docker-entrypoint-initdb.d/init.sql

  mysql:
    image: mysql:8
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: shipments
      MYSQL_USER: trino
      MYSQL_PASSWORD: trino
    ports:
      - "3306:3306"
    volumes:
      - ./init-db/mysql-init.sql:/docker-entrypoint-initdb.d/init.sql

  metabase:
    image: metabase/metabase
    container_name: metabase
    ports:
      - "3000:3000"



