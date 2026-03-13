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
 

### Docker Compose configuration
- It defines multiple containers (services) that will run together to build a data platform stack
  ```bash
  docker-compose up -d #Docker will create and start all these services
  ```

### Architecture Overview

- Your stack contains:
    - Trino → Distributed SQL query engine
    - PostgreSQL → Database
    - MySQL → Database
    - Metabase → BI / dashboard tool
    - Keycloak → Identity & authentication server
    - OAuth2 Proxy → Authentication gateway for Metabase

    ```
    User → OAuth2 Proxy → Keycloak (login)
                ↓
              Metabase
                ↓
              Trino
         ↙             ↘
      Postgres        MySQL
    ```

  ### Trino Service
  ```yml
  trino:
  image: trinodb/trino
  container_name: trino
  ports:
    - "8080:8080" #HOST_PORT : CONTAINER_PORT
  volumes:
    - ./trino:/etc/trino #HOST_PATH : CONTAINER_PATH
  ```
- Purpose: SQL query engine
- What it does:
    - Connects to multiple databases
    - Allows federated queries

- ./trino → Folder in your host system (current directory)
- /etc/trino → Folder inside the container
  ```
  Host: ./trino/config.properties
            ↓
  Container: /etc/trino/config.properties
  ```
- The container will read configuration files from your host machine.
- If you edit a file:
  ```
  ./trino/config.properties
  ```
  - the change immediately reflects inside the container.

1️⃣ Trino Architecture

- Trino has two main components:
    - Coordinator
    - Workers

- Coordinator
    - Accepts client queries
    - Parses and plans the query
    - Splits the query into tasks
    - Sends tasks to workers
    - Collects final results
- Workers
    - Execute the tasks
    - Process data
    - Send results back to coordinator

Simple Architecture
```
Client
   │
   ▼
Coordinator
   │
   ├── Worker 1
   ├── Worker 2
   ├── Worker 3
   └── Worker N
```

### Example Flow
***Query Sent to Coordinator***
- Client sends query using:
    - Trino CLI
    - JDBC
    - BI tool

- Example flow:
  ```
      Client
       │
       ▼
    Trino Coordinator
  ```
- The Coordinator receives the SQL.
***Parsing and Planning***
- Coordinator does:
    - Parsing
        - Checks SQL syntax
    - Semantic analysis
        - Identifies catalogs
            - mysql.shipments.shipments
            - postgresql.public.customers
    - Logical Plan
        - Example logical plan:
          ```
          Scan MySQL shipments
          Scan PostgreSQL customers
          Join on customer_id = id
          Return columns
          ```
  ***Query Split into Stages***
  - Coordinator divides the query into execution stages.
  - Example:
    ```
    Stage 1 → Scan MySQL table
    Stage 2 → Scan PostgreSQL table
    Stage 3 → Perform JOIN
    Stage 4 → Return results
    ```
  ***Tasks Distributed to Workers***
    - Coordinator sends tasks to worker nodes
    - Workers run the tasks
    
 ***Data Fetch from MySQL and PostgreSQL***
 - Workers connect to both databases using connectors.
 - Example
   ```
   Worker 1 → fetch MySQL shipments data
   Worker 2 → fetch PostgreSQL customers data
   Worker 3 → assist with join processing
   ```

- Trino connectors handle this:
  ```
  mysql connector
  postgresql connector
  ```

***Data Processing in Workers***
- Workers process data in parallel.
- Example:
```
MySQL shipments table
---------------------
Split 1 → Worker 1
Split 2 → Worker 2

PostgreSQL customers table
--------------------------
Split 1 → Worker 2
Split 2 → Worker 3
```
- Each worker processes part of the dataset
***Join Execution***
- Workers perform the join condition:
```
s.customer_id = c.id
```
***Partial Results Sent to Coordinator***
- Workers send partial results:
  ```
  Worker 1 → partial rows
  Worker 2 → partial rows
  Worker 3 → partial rows
  ```
- Coordinator merges them

***Final Result Returned***
- Coordinator sends final result to client.
```
Coordinator
   │
   ▼
Client (CLI / BI tool)
```
- This query is called Federated Query.

- Meaning: One query accessing multiple data sources.
- Example sources Trino supports: MySQL, PostgreSQL, Hive, Iceberg, Kafka, MongoDB

***Trino coordinator parses the query and creates a distributed execution plan. Workers fetch data from MySQL and PostgreSQL using connectors, process data in parallel, perform the join in memory, and send results back to the coordinator.***

### What is Coordinator Failure?
- Coordinator failure means:
    - Coordinator process crashes
    - Server goes down
    - Network becomes unreachable
    - JVM crash
    - Out of memory
    - Host machine failure
Example errors:
```
Query failed: coordinator unreachable
Connection refused :8080
```
### What Happens When Coordinator Fails

- Running queries fail: All active queries stop.
- New queries cannot be submitted: Clients cannot connect
- Workers become idle
- Workers:
    - Do not automatically elect a new coordinator
    - Wait for coordinator to return
    - Trino does not support automatic leader election.
    ```
    Query aborted: coordinator lost
    http://coordinator:8080 unreachable
    Worker 1 → idle
    Worker 2 → idle
    Worker 3 → idle
    ```

### How to Detect Coordinator Failure?
```
systemctl status trino
curl http://localhost:8080/v1/info
http://coordinator:8080/ui
/var/log/trino/server.log
```
### How to Fix Coordinator Failure?
```
systemctl restart trino #Check service
Coordinator often fails due to JVM memory issues.

/etc/trino/jvm.config
-Xmx16G

Check disk
If disk is full:
df -h

tail -f /var/log/trino/server.log #Check Logs

```

### Standby Coordinator

```
           Clients
              │
        Load Balancer
          /        \
Active Coordinator   Standby Coordinator
        │
   Worker Nodes

```
- Active coordinator → Handles queries
- Standby coordinator → Idle / waiting
- Workers → Execute tasks


Even with HA:

- Running queries fail if the active coordinator crashes.
- Only new queries succeed after failover.
- This is a limitation of Trino’s architecture.

### How Trino handles fault tolerance?
- Trino provides fault tolerance mainly at the worker level. The coordinator monitors worker nodes using heartbeats. If a worker fails during query execution, the coordinator reschedules the failed tasks on other workers using data splits. However, if the coordinator fails, running queries fail because the coordinator manages query planning and scheduling.

### Why is Trino not a database?
- Trino is not a database because it does not store data. It is a distributed SQL query engine that queries data from external systems like MySQL, PostgreSQL, Hive, and Iceberg using connectors. Trino processes the data in memory and returns results, but the actual data remains in the underlying storage systems.

### Difference between Trino and Presto?
- Trino is a fork of Presto created by the original developers after they left Facebook. Both are distributed SQL query engines with similar architecture, but Trino has faster development, more community contributions, and more modern features compared to Presto.
