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



### How trino connetcs do databases?
- Trino connects to databases using something called connectors. A connector is like a plugin that allows Trino to talk to a specific data source.
- A connector is a module that enables Trino to communicate with external systems such as: MySQL, PostgreSQL, Apache Hive, Apache Kafka
- Each connector knows how to query that specific system
- Connector configuration files are placed in:
```
/etc/trino/catalog/
    mysql.properties
    postgresql.properties
```

### What happens if Trino has 100 data sources?
- In Trino, each data source is configured as a catalog using a connector. If there are 100 data sources, there will be 100 catalog configurations in /etc/trino/catalog. When a query references a catalog, Trino uses the corresponding connector to fetch data from that system. The coordinator plans the query and workers execute it in parallel.

### Connectors
```
mysql.properties
postgresql.properties
```

```yml
connector.name=mysql
connection-url=jdbc:mysql://mysql-host:3306
connection-user=trino
connection-password=trino

connector.name=postgresql
connection-url=jdbc:postgresql://postgres:5432/logistics
connection-user=trino
connection-password=trino
```
- logistics → Database name
- public → Default schema inside that database
- tables → Stored inside the schema

```
Docker starts container
        │
PostgreSQL server starts
        │
Create user = trino
Create database = logistics
        │
Run init.sql script
        │
Database ready
```
- In Docker Compose, the PostgreSQL container is created using the official image. Environment variables create the database, user, and password automatically during startup. The port mapping exposes PostgreSQL on localhost:5432, and the initialization SQL script runs automatically to create tables or seed data.

```
/docker-entrypoint-initdb.d/init.sql # This is a special directory inside the PostgreSQL Docker image.


Docker starts container
        │
PostgreSQL initializes database
        │
Check /docker-entrypoint-initdb.d/
        │
Run init.sql automatically
        │
Tables / data created
```

```
Host machine
--------------------------------
./init-db/postgres-init.sql
          │
          │ volume mount
          ▼
Container
--------------------------------
/docker-entrypoint-initdb.d/init.sql
```
- The name inside the container does not have to match the host name.
  ```
    - volumes:
      - ./init-db/postgres-init.sql:/docker-entrypoint-initdb.d/setup.sql
  ```
- Because PostgreSQL runs all .sql files in that directory.
- Docker mounts ./init-db/postgres-init.sql from the host and exposes it inside the container as /docker-entrypoint-initdb.d/init.sql, which PostgreSQL automatically executes during initialization

### config.properties
- In Trino, the config.properties file is the main server configuration file. It defines how the Trino node behaves and how the cluster works.

```
############################################
# TRINO SERVER CONFIGURATION
# Main configuration file for Trino nodes
############################################


############################################
# NODE ROLE CONFIGURATION
############################################

# Defines whether this node acts as the coordinator.
# The coordinator receives queries, plans execution,
# schedules tasks on workers, and returns results.
coordinator=true

# Allows the coordinator to also execute query tasks.
# Usually true for single-node or development setups.
# In production clusters this is typically set to false.
node-scheduler.include-coordinator=true


############################################
# HTTP SERVER CONFIGURATION
############################################

# Port on which the Trino server listens for requests.
# Clients, CLI, and UI connect to this port.
http-server.http.port=8080

# Enable HTTPS communication (optional).
# http-server.https.enabled=false

# HTTPS port if HTTPS is enabled.
# http-server.https.port=8443


############################################
# CLUSTER DISCOVERY CONFIGURATION
############################################

# Enables the discovery service.
# Workers use this service to register with the coordinator.
discovery-server.enabled=true

# URI where the discovery service is available.
# Workers use this endpoint to locate the coordinator.
# In Kubernetes this would typically be the coordinator service.
discovery.uri=http://localhost:8080


############################################
# QUERY MEMORY LIMITS
############################################

# Maximum memory a query can use across the entire cluster.
# If exceeded, the query will fail.
query.max-memory=2GB

# Maximum memory a query can use on a single node.
# Prevents a single worker from running out of memory.
query.max-memory-per-node=512MB

# Maximum total memory including revocable memory.
# Helps protect the cluster from excessive memory usage.
# query.max-total-memory=4GB


############################################
# QUERY EXECUTION LIMITS
############################################

# Maximum runtime allowed for a query.
# Queries running longer than this will be terminated.
# query.max-run-time=1h

# Maximum number of stages allowed in a query plan.
# Helps prevent overly complex queries.
# query.max-stage-count=150

# Number of completed queries kept in history.
# query.max-history=100


############################################
# TASK EXECUTION SETTINGS
############################################

# Number of concurrent drivers per task.
# Controls parallelism during execution.
# task.concurrency=4

# Maximum worker threads used for task execution.
# task.max-worker-threads=16


############################################
# DATA EXCHANGE SETTINGS
############################################

# Number of threads used for exchanging data
# between worker nodes during query execution.
# exchange.client-threads=25

# Maximum buffer size used during data exchange.
# exchange.max-buffer-size=32MB


############################################
# NODE SCHEDULER SETTINGS
############################################

# Maximum number of splits (data partitions)
# that can be scheduled on a node.
# node-scheduler.max-splits-per-node=100

# Maximum number of pending splits per task.
# node-scheduler.max-pending-splits-per-task=10


############################################
# SPILL TO DISK SETTINGS
############################################

# Enables spilling intermediate data to disk
# when queries exceed available memory.
# spill-enabled=true

# Directory where spilled data is stored.
# spiller-spill-path=/data/trino/spill


############################################
# FAULT TOLERANCE / RETRY SETTINGS
############################################

# Retry policy for failed tasks.
# TASK retry allows retrying individual failed tasks.
# retry-policy=TASK
```

### Worker Node - config.properties
```
############################################
# TRINO WORKER CONFIGURATION
# Configuration for worker nodes
############################################


############################################
# NODE ROLE
############################################

# This node is a worker, not a coordinator.
# Workers execute query tasks assigned by the coordinator.
coordinator=false


############################################
# HTTP SERVER CONFIGURATION
############################################

# Port where the Trino worker listens.
# The coordinator communicates with workers using this port.
http-server.http.port=8080


############################################
# CLUSTER DISCOVERY
############################################

# URI of the coordinator discovery service.
# Workers use this endpoint to register themselves with the coordinator.
# In Kubernetes, this would typically be the coordinator service name.
discovery.uri=http://trino-coordinator:8080


############################################
# QUERY MEMORY LIMITS
############################################

# Maximum memory a query can use on this worker node.
# Helps protect the node from running out of memory.
query.max-memory-per-node=512MB

# Maximum total memory a query can use on this worker node,
# including revocable memory.
query.max-total-memory-per-node=1GB


############################################
# TASK EXECUTION SETTINGS
############################################

# Number of parallel drivers per task.
# Controls how much parallelism happens within each task.
task.concurrency=4

# Maximum number of worker threads used to process tasks.
task.max-worker-threads=16


############################################
# DATA EXCHANGE SETTINGS
############################################

# Threads used to exchange data between workers.
exchange.client-threads=25

# Maximum buffer size for data exchange between nodes.
exchange.max-buffer-size=32MB


############################################
# SPILL TO DISK SETTINGS (Optional)
############################################

# Allows queries to spill data to disk if memory is insufficient.
spill-enabled=true

# Directory where spilled data will be stored.
spiller-spill-path=/data/trino/spill


############################################
# NODE SCHEDULER SETTINGS
############################################

# Maximum number of data splits processed by this node.
node-scheduler.max-splits-per-node=100

# Maximum pending splits allowed per task.
node-scheduler.max-pending-splits-per-task=10
```

- By default, Trino does not enforce authentication. The username entered in the UI is only used to identify the query user. Real authentication must be configured explicitly using mechanisms such as password authentication, OAuth, LDAP, or Kerberos.
```
http-server.authentication.type=PASSWORD
password-authenticator.name=file

/etc/trino/password-authenticator.properties
    password-authenticator.name=file
    file.password-file=/etc/trino/password.db

/etc/trino/password.db and hashed password like below
    admin:$2y$10$2JYy6YH9Yb9hO6KX1CqM6eM8kS4mZ0yA4x6b7K3N4fT5v8g9q1aBC
    user1:$2y$10$A9uYp7X4J6Wb9LQ1s3d6cZ2eN0rV8P5K7tY4qF1s2b6D9gH3mC8pO

htpasswd -nbB admin password

docker restart trino

```
- Yes. Trino supports password authentication using a file-based password authenticator. Users and hashed passwords are stored in a password file, and the coordinator validates credentials during login.
- Applications like Trino and Metabase redirect users to Keycloak for authentication. After the user logs in, Keycloak creates a browser session. When the user accesses another application, Keycloak detects the existing session and issues a token without prompting for login again.

### What jvm.config Controls
- This file defines:
    - JVM memory allocation
    - garbage collection settings
    - JVM performance tuning
    - debugging options
It directly affects Trino performance and stability.
- We need jvm.config in Trino because Trino runs on the Java Virtual Machine (JVM). This file tells the JVM how much memory to use and how to manage resources when running Trino. Without proper JVM settings, Trino may run slowly, crash, or run out of memory.
- Heap memory stores Java objects created during query execution, such as query plans, task objects, metadata, intermediate results, and buffers used to process data.

```
########################################################
# TRINO JVM CONFIGURATION
# This file contains JVM startup parameters used to run
# the Trino server. These settings control memory usage,
# garbage collection, and JVM performance.
########################################################

# Run JVM in server mode.
# This mode is optimized for long-running server applications
# like Trino and provides better performance.
-server


########################################################
# HEAP MEMORY CONFIGURATION
########################################################

# Maximum heap memory size.
# This defines the maximum amount of RAM the JVM can use
# to store Java objects such as query execution data,
# intermediate results, metadata objects, and buffers.
#
# Example:
# -Xmx16G means JVM can use up to 16 GB RAM.
#
# Important rule:
# Heap size should typically be 70–80% of system memory.
-Xmx16G


########################################################
# GARBAGE COLLECTION SETTINGS
########################################################

# Use the G1 Garbage Collector.
# G1GC is recommended for large heap sizes and
# low pause time applications like Trino.
#
# It helps reduce long garbage collection pauses
# during heavy query workloads.
-XX:+UseG1GC


# Define the heap region size used by G1GC.
# The heap is divided into regions for efficient memory
# management. Larger heaps benefit from larger regions.
#
# Example:
# 32M means each region is 32 MB.
-XX:G1HeapRegionSize=32M


########################################################
# MEMORY MANAGEMENT SAFETY OPTIONS
########################################################

# Prevents the JVM from spending too much time
# doing garbage collection. If GC overhead becomes
# too high, the JVM may terminate the process.
-XX:+UseGCOverheadLimit


# Allows explicit garbage collection calls to run concurrently
# instead of stopping the entire application.
-XX:+ExplicitGCInvokesConcurrent


########################################################
# ERROR HANDLING OPTIONS
########################################################

# If the JVM runs out of memory, create a heap dump.
# Heap dumps help engineers debug memory leaks
# or excessive memory usage.
-XX:+HeapDumpOnOutOfMemoryError


# Immediately terminate the JVM if an OutOfMemoryError occurs.
# This prevents the server from running in an unstable state.
-XX:+ExitOnOutOfMemoryError


########################################################
# OPTIONAL DEBUGGING OPTIONS (NOT ALWAYS USED)
########################################################

# Enable detailed GC logging for troubleshooting memory issues.
# Uncomment if you need to debug garbage collection behavior.
#
# -Xlog:gc


# Specify heap dump file location.
# Example:
# -XX:HeapDumpPath=/var/log/trino/heapdump.hprof
```
- jvm.config defines the JVM runtime parameters used to start the Trino server. It controls heap memory size, garbage collection settings, and error handling to ensure efficient execution of queries.

### Node properties (node.properties)
- Each Trino node must have a unique node.id defined in node.properties. When deploying multiple coordinators, they are usually named coordinator-1, coordinator-2, etc., or derived from pod hostnames in Kubernetes. In production, one coordinator is typically active while the other acts as a standby behind a load balancer.
- In a Trino cluster, workers typically share the same configuration while the coordinator has a slightly different configuration. Each node still requires a unique node ID, which is often derived from the hostname in Kubernetes

```
trino-cluster/
│
├── docker-compose.yml
│
├── coordinator/
│   ├── config.properties
│   ├── node.properties
│   └── jvm.config
│
├── worker/
│   ├── config.properties
│   ├── node.properties
│   └── jvm.config
│
└── catalog/
    └── postgres.properties
```


### Keycloak



| Mode      | Purpose          |
| --------- | ---------------- |
| start-dev | development mode |
| start     | production mode  |

start-dev:
- no TLS required
- easier configuration
- auto database setup
- Used for local testing.
- 

A realm is a security domain
- Client ID: trino or Client ID: trino
- Redirect URL: http://174.129.146.225:8080/ui/* OR http://174.129.146.225:3000/*
- Trino Configuration Example
  ```
  http-server.authentication.type=oauth2
  http-server.authentication.oauth2.issuer=http://174.129.146.225:8081/realms/datawave
  http-server.authentication.oauth2.client-id=trino
  http-server.authentication.oauth2.client-secret=<secret>
  http-server.authentication.oauth2.principal-field=preferred_username
  ```
  - How Applications Connect to Keycloak
    ```
    http://174.129.146.225:8081/realms/datawave/.well-known/openid-configuration #Applications use the OpenID discovery endpoint
    ```
  - This provides:
      - authorization endpoint
      - token endpoint
      - public keys
    ```
            User
             │
             ▼
            Metabase / Trino
             │
             ▼
            Keycloak (SSO)
             │
             ▼
            Identity verification
    ```
- Keycloak acts as an identity provider. Applications like Trino or Metabase redirect users to Keycloak for authentication using OAuth2 or OpenID Connect. After login, Keycloak returns an access token that the application validates to establish the user session.
```
        User Browser
              │
              ▼
        Trino UI
              │
        Redirect
              ▼
        Keycloak Login
              │
        User Authentication
              │
        Keycloak returns Authorization Code
              │
        Trino exchanges code for token
              │
        Token validated
              │
        Session created
              ▼
        User accesses Trino
```
- When a user accesses Trino, the server redirects the browser to Keycloak for authentication. After successful login, Keycloak returns an authorization code. Trino exchanges this code for an access token, validates the token, extracts the username from the token claims, and creates a session for the authenticated user.
- The client secret is a credential used by the application to authenticate itself with the identity provider. During the OAuth2 token exchange, Trino sends the client ID and client secret to Keycloak to securely obtain an access token.
