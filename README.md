#  DataWave Industries – SQL Federation Architecture

## Table of Contents

- Problem Summary
- Architecture Overview
- Core Components
- Component Interaction Flow
- End-to-End Data Flow
- Project Structure
- Directory Explanation
- Docker Networking and Container Communication in SQL Federation Architecture
- Project Execution
  - Setup Instructions & User Guide
- SSO Integration
- Troubleshooting Guide
- Improvements and Conclusion

---
## Problem Summary
DataWave currently runs analytics on an old on-prem Hadoop cluster that has several issues:

**Key Problems**
- Scalability limits – Cannot scale easily as data grows.
- Performance bottlenecks – Queries are slow with increasing workloads.
- Availability issues – Cluster instability and downtime.
- Operational overhead – Requires significant maintenance.
Because of this, the company wants to modernize their data platform in the cloud.
---
## Architecture Overview

The DataWave SQL Federation Architecture enables unified querying across multiple heterogeneous data sources using Trino as the federation engine. It integrates governance, authentication, analytics, and auditing components to provide a secure and scalable data platform.

<img width="1000" height="350" alt="image" src="https://github.com/user-attachments/assets/71a8577e-188a-48ac-ba8f-f89895d57fb0" />

## Core Components
### 1. Data Sources (Heterogeneous Databases)
- Data sources are the systems where the actual data is stored. In this architecture, multiple types of databases are connected to the federation engine.
- Examples of data sources include:
  ```text
  PostgreSQL – stores structured relational data
  MySQL – stores operational data
  Object storage (MinIO / S3) – stores large datasets or files
  Other databases – such as Hive, Elasticsearch, or analytics systems
  ```
- Purpose
  - These systems hold the raw data used by applications and business teams.
- Role in the Architecture
  - Instead of copying or moving the data into one centralized database, the architecture allows querying data directly from these systems using the federation engine.
-  Benefit
  - This reduces data duplication and allows real-time access to multiple data sources.


### 2. Trino (SQL Federation Engine)
- Trino is the central query engine in this architecture. It allows users to run SQL queries across multiple databases as if they were a single system.
- Responsibilities
    - Connects to different data sources using connectors
    - Executes distributed SQL queries
    - Combines results from multiple systems
    - Provides a unified query interface
- Example:
  - A query can combine data from MySQL and PostgreSQL:
    ```yml
    SELECT s.shipment_id, l.location
    FROM mysql.shipments.shipments s
    JOIN postgres.logistics.locations l
    ON s.location_id = l.id;
    ```
- Role in Architecture
  - Trino sits between the users and the data sources and acts as a federation layer.
- Benefit
    - Users can query multiple systems using one SQL interface.

### 3. Apache Ranger (Data Governance and Access Control) 
- Apache Ranger is responsible for security and data governance in the system.
- Responsibilities
    - Manage access control policies
    - Provide role-based access control
    - Enforce data security policies
    - Manage fine-grained permissions
- Example Policies
  ```text
  Allow analysts to read only specific tables
  Restrict access to sensitive columns
  Limit data access based on user roles
  ```
- Role in Architecture
  - Ranger integrates with Trino to check whether a user is allowed to access certain data before executing the query.
- Benefit
  - Ensures secure and controlled access to sensitive data

ℹ️ **Note:** Apache Ranger is **not implemented in this project** because an official and lightweight Docker image is not readily available in public repositories for simple local deployment. However, 
in a production-grade architecture, Apache Ranger can be integrated with Trino to enforce access control policies.

### 4. Keycloak (Authentication and Single Sign-On)
- Keycloak provides identity and access management.
- Responsibilities
    - User authentication
    - Identity management
    - Single Sign-On (SSO)
    - Role and group management
- Workflow
    - A user logs in through Keycloak
    - Keycloak authenticates the user credentials
    - User roles and groups are provided to the system
    - Ranger uses this information to enforce access policies
- Role in Architecture
  - Keycloak ensures that only authenticated users can access the data platform.
- Benefit
  - Provides centralized and secure user authentication.

ℹ️ **Note:** Since Metabase Community Edition does not support OIDC authentication and an enterprise license was not available, Keycloak was implemented using OAuth2 through a reverse proxy to enable secure SSO access.

### 5. Metabase (Business Intelligence and Analytics)
- Metabase is a web-based analytics and visualization platform.
- Responsibilities
  - Provide dashboards and reports
  - Allow users to run SQL queries
  - Create charts and visualizations
  - Enable business intelligence analytics
- Role in Architecture
  - Metabase connects to Trino as a data source and allows users to analyze data from multiple databases.
- Benefit
  - Business users can analyze data without needing deep technical knowledge.

### 6. Elasticsearch (Audit Logging and Monitoring)
- Elasticsearch is used for storing and analyzing audit logs generated by the system.
- Responsibilities
  - Store access logs
  - Track user queries
  - Monitor system activity
  - Support compliance and auditing
- Example Audit Information
  - Username
  - Executed query
  - Database accessed
  - Timestamp
  - Access decision (allowed or denied)  
- Role in Architecture
  - When a user queries data, Ranger generates audit logs which are stored in Elasticsearch.
- Benefit
  - Provides transparency and helps detect unauthorized data access.

ℹ️ **Note:** Elasticsearch was considered as a potential solution for centralized logging and observability in this architecture. However, a full logging stack such as ELK (Elasticsearch, Logstash, Kibana) was not implemented in this project, as the primary focus of this case study was to demonstrate the SQL federation architecture using Trino and data visualization through Metabase.

## Component Interaction Flow
The following sequence describes how the components interact.
- Step 1 — User Authentication
  - Users authenticate through Keycloak using SSO.
- Step 2 — Authorization
  - User roles and permissions are verified by Apache Ranger.
- Step 3 — Query Execution
  - Users or applications submit queries through Metabase.
- Step 4 — Federated Query Processing
  - Metabase sends the query to Trino.
  - Trino:
    - Connects to multiple databases
    - Executes distributed queries
    - Aggregates results
- Step 5 — Data Access Enforcement
  - Ranger validates that the user has permission to access requested data.
- Step 6 — Audit Logging
  - All query activity is logged into Elasticsearch.
- Step 7 — Visualization
  - Metabase displays query results in dashboards for business users.

## End-to-End Data Flow
<img width="713" height="553" alt="image" src="https://github.com/user-attachments/assets/b59ed971-d233-4d68-bd68-bf2c5f6629ad" />



## Project Structure
```
sql_federation_architecture
│
├── docker-compose.yml
│
├── init-db
│   ├── mysql-init.sql
│   └── postgres-init.sql
│
├── trino
│   ├── config.properties
│   ├── jvm.config
│   ├── node.properties 
│   └── catalog
│       ├── mysql.properties
│       └── postgresql.properties
│
└── README.md
```

## Directory Explanation
### docker-compose.yml
- Defines all services required for the architecture.
- Services include:
  - Trino (SQL federation engine)
  - PostgreSQL
  - MySQL
  - Metabase
  - Keycloak
- Docker Compose allows the entire architecture to start with a single command:
  ```yml
  docker-compose up -d
  ```
### init-db/
This directory contains SQL scripts used to initialize the databases when containers start.
- **mysql-init.sql**
  - Creates tables and sample data for the MySQL database.
  - Example use case:
    -  shipment data
    - operational datasets
- **postgres-init.sql**
  - Creates tables and sample data for the PostgreSQL database.
  - Example use case:
    - logistics data
    - location datasets
  - These scripts run automatically when the database container starts. 

### trino/
- This directory contains the configuration files for the Trino SQL federation engine.
- Trino uses these files to define how it runs and how it connects to external data sources.

  **config.properties**
  - Defines the Trino server configuration.
  - Example parameters:
    ```yml
    coordinator=true
    node-scheduler.include-coordinator=true
    http-server.http.port=8080
    discovery-server.enabled=true
    ```
  - Purpose:
    - Configures the Trino coordinator
    - Defines the HTTP port
    - Enables cluster discovery

  **jvm.config**
  - Defines Java Virtual Machine settings for Trino.
  - Example settings:
    ```YML
    -Xmx2G
    -XX:+UseG1GC
    ```
  - Purpose:
    - Controls memory allocation
    - Improves performance and garbage collection

  **node.properties**
  - Defines node-specific configuration.
  - Example:
    ```YML
    node.environment=production
    node.id=trino-node-1
    node.data-dir=/var/trino/data
    ```
  - Purpose:
    - Identifies the node in the cluster
    - Defines the data directory

### catalog/
- The catalog directory defines connectors to external data sources.
- Each .properties file represents a data source connection.

  **mysql.properties**
  - Configures the MySQL connector.
  - Example:
    ```YML
    connector.name=mysql
    connection-url=jdbc:mysql://mysql:3306/shipments
    connection-user=trino
    connection-password=trino
    ```
  - Purpose:
    - Allows Trino to query MySQL databases.
  
  **postgresql.properties**
  - Configures the PostgreSQL connector.
  - Example:
    ```YML
    connector.name=postgresql
    connection-url=jdbc:postgresql://postgres:5432/logistics
    connection-user=trino
    connection-password=trino
    ```
  - Purpose:
    - Enables Trino to access PostgreSQL tables.

## Docker Networking and Container Communication in SQL Federation Architecture
Docker Compose automatically creates an **isolated virtual network** for all services defined in the `docker-compose.yml` file.\ This allows containers to communicate with each other using **service
names instead of IP addresses**.
### Docker Container Architecture
```
Docker Host (EC2 Instance)
│
├── trino container
│      Port: 8080
│
├── metabase container
│      Port: 3000
│
├── mysql container
│      Port: 3306
│
├── postgres container
│      Port: 5432
│
└── docker network
       sql_federation_architecture_default
```
- When the platform is started with:
  ```
  docker-compose up -d
  ```
- Docker automatically creates a network similar to:
  ```
  sql_federation_architecture_default
  ```
- All containers join this network.

### How Containers Communicate
- Inside a Docker network, containers communicate using **DNS-based service discovery**.
- Instead of container IP addresses (which may change), services use the **service name defined in docker-compose.yml**.
- Example:
  - Trino connects to MySQL using:
    ```
    jdbc:mysql://mysql:3306/shipments
    ```
    - Where:
      - mysql → Docker service name
      - 3306 → MySQL container port

- Docker automatically resolves the hostname `mysql` to the correct container.

### Service Communication
```
Metabase
   │
   ▼
Trino
   │
   ├── MySQL
   └── PostgreSQL
```
### Host vs Container Networking

### 1. Container-to-Container Communication
- Uses **service names**.
- Example:
  ```
  http://trino:8080
  ```
- Used by:
  ```
  Metabase → Trino
  Trino → MySQL
  Trino → PostgreSQL
  ```

### 2. Host-to-Container Communication
- Uses **mapped ports**.
- Example:
  ```
  http://<EC2_PUBLIC_IP>:8080
  ```
- Used by:
  ```
  User Browser → Trino\
  User Browser → Metabase\
  User Browser → Keycloak
  ```
- Example port mappings:
  | Service      | Container Port | Host Port |
  | ------------ | -------------- | --------- |
  | Trino        | 8080           | 8080      |
  | Metabase     | 3000           | 3000      |
  | Keycloak     | 8080           | 8081      |
  | OAuth2 Proxy | 4180           | 4180      |


###  Docker Compose Networking
- services:
  ```
  trino: image: trinodb/trino ports: - "8080:8080"
  mysql: image: mysql:8 ports: - "3306:3306"
  postgres: image: postgres:14 ports: - "5432:5432"
  metabase: image: metabase/metabase ports: - "3000:3000"
  ```
- All services automatically join the same Docker network.

### Inspecting Docker Networks
- To view Docker networks:
  ```
  docker network ls
  ```
- Example output:
  ```
  bridge\
  host\
  none\
  sql_federation_architecture_default
  ```
- To inspect the project network:
  ```
  docker network inspect sql_federation_architecture_default
  ```
- This command shows:
  - Connected containers
  - Container IP addresses
  - Network configuration

ℹ️ **Note:** This command can be useful for troubleshooting network-related issues between services running in the Docker Compose environment.


## Project Execution
### Setup Instructions & User Guide
### Architecture Diagram – SQL Federation Platform (Trino, Metabase, Postgres and Mysql)

```
        
                                             ┌──────────────────────────────┐
                                             │            User              │
                                             │  (Browser / BI Analyst)      │
                                             └───────────────┬──────────────┘
                                                             │
                                                             │ HTTP
                                                             ▼
                                             ┌──────────────────────────────┐
                                             │           Metabase           │
                                             │     BI / Analytics Layer     │
                                             │        Port: 3000            │
                                             └───────────────┬──────────────┘
                                                             │
                                                             │ SQL Queries
                                                             ▼
                                             ┌──────────────────────────────┐
                                             │            Trino             │
                                             │      SQL Federation Engine   │
                                             │        Port: 8080            │
                                             └───────────────┬──────────────┘
                                                             │
                                             ┌───────────────┴───────────────┐
                                             │                               │
                                             ▼                               ▼
                               ┌──────────────────────────┐     ┌──────────────────────────┐
                               │          MySQL           │     │        PostgreSQL        │
                               │   Operational Database   │     │    Relational Database   │
                               │        Port: 3306        │     │        Port: 5432        │
                               └──────────────────────────┘     └──────────────────────────┘
                    
        
                        Docker Network
                sql_federation_architecture_default
```

**Step 1 — How to install dependencies**
- Prerequisites
- Before starting, ensure:
  -   Amazon Linux instance (EC2 or local VM)
  -   `sudo` privileges
  -   Internet connectivity
  -   `dnf` package manager available
```
## Install Docker

sudo dnf update -y             # Update all installed packages to the latest version.
sudo dnf install docker -y     # Install Docker from the Amazon Linux repository.
sudo systemctl start docker    # Start Docker Service
sudo systemctl enable docker   # Enable Docker to start automatically at boot.
sudo systemctl status docker   # Check Docker status.
sudo usermod -aG docker $USER  # Add your user to the Docker group.
newgrp docker                  # Apply the group change.
docker --version               # Verify Docker Installation: Output --> Docker version 25.x

## Install Docker Compose

sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose # Download Docker Compose.
sudo chmod +x /usr/local/bin/docker-compose # Provide execution permission.
docker-compose --version # Verify Docker Compose Installation:  Output --> Docker Compose version v2.x.x


## Install & Verify Git

sudo dnf install git -y # Install Git using DNF
git --version git #Output --> Check Git version. version 2.x.x

sudo dnf install java-17-amazon-corretto-devel -y
java -version
```

**Step 2 — How to Clone the Repository**
```
git clone https://gitlab.com/satishdevops777/sql_federation_architecture.git && cd sql_federation_architecture
```
<img width="1731" height="272" alt="image" src="https://github.com/user-attachments/assets/ad58f9db-3903-487b-868f-f8c29b54dba1" />



**Step 3 — How to Start the Platform**
Start all services using Docker Compose.
```bash
docker-compose up -d
```
<img width="1918" height="395" alt="image" src="https://github.com/user-attachments/assets/778122b2-7583-49d3-a861-a2cd2e0342ad" />




**Step 4 - How to Verify that all containers are running and inspect logs if any service is not functioning correctly.**
```
# List all running containers
docker ps

# Check logs of a specific container
docker logs <container_name> --tail 50
```
<img width="1918" height="838" alt="image" src="https://github.com/user-attachments/assets/34990ebd-9e6a-405c-861f-e7a3b5520b5b" />

```
trino   Up (unhealthy)
```
- This usually happens because Trino requires some time (10–20 seconds) to fully initialize its connectors and internal services during startup. During this initialization phase, the Docker healthcheck endpoint may temporarily return HTTP 503, which causes Docker to mark the container as unhealthy.

**Step 5 - How to Access Service URLs**
- Service Access
- After deployment, the services can be accessed at:

| Service  | URL                                            | Description         |
| -------- | ---------------------------------------------- | ------------------- |
| Trino    | [http://localhost:8080](http://localhost:8080) | SQL query engine    |
| Metabase | [http://localhost:3000](http://localhost:3000) | BI dashboards       |
| Keycloak | [http://localhost:8081](http://localhost:8081) | Authentication      |

**Note: When deploying this setup on an AWS EC2 instance, replace localhost with the EC2 instance's public IP address to access the services from your browser or external systems.**

**Step 6 – How to access the SQL Federation Engine CLI OR UI and Run Queries**
- Run below command to access SQL Federation Engine CLI (trino)
  ```
  docker exec -it trino trino
  ```
  <img width="1073" height="69" alt="image" src="https://github.com/user-attachments/assets/d55a90ca-2d3e-47e0-9017-5f587f0a1da7" />
- Run below example SQL Queries

**Show connected data sources**
  ```sql
  SHOW CATALOGS; 
  ````
  <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/92b60b81-2f8e-46d2-a435-439d491d8b5f" />


**Query MySQL**
  ```sql
  SELECT * FROM mysql.shipments.shipments;
  ```
  <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/fe4d40a1-4a13-48f1-9e26-dce687b000b7" />


**Query PostgreSQL**
  ```sql
  SELECT * FROM postgresql.public.customers;
  ```
  <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/d3b34d9f-08fc-4bb7-a9e5-759f932a6c97" />

    

**Cross-database federation query with join**
  ```sql
  SELECT
      s.shipment_id,
      s.product_name,
      c.customer_name
  FROM mysql.shipments.shipments s
  JOIN postgresql.public.customers c
  ON s.customer_id = c.id;
  ```
  <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/69e8a3ed-e1dc-45a5-9fba-0ca3491505f7" />

    

**Analytical query**
  ```sql
  SELECT
      c.customer_name,
  COUNT(s.shipment_id) AS total_shipments
  FROM mysql.shipments.shipments s
  JOIN postgresql.public.customers c
  ON s.customer_id = c.id
  GROUP BY c.customer_name
  ORDER BY total_shipments DESC;
  ```
  <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/59a38236-7ee0-4233-9c02-4db368342df1" />


- Run below command to access SQL Federation Engine UI (trino)
- Open the URL in browser
  ```MARKDOWN
  Note: Replace <EC2_PUBLIC_IP> with your EC2 instance's public IP address. 
  In this setup, Trino is running on an EC2 instance and can be accessed using:
  
  http://<EC2_PUBLIC_IP>:8080
  ```
- URL
  ```
  http://174.129.146.225:8080 
  ```
- Trino requires a user name to identify the query executor. In this setup authentication is not enabled, so any username can be provided to access the Trino UI.
  
  <img width="1091" height="300" alt="image" src="https://github.com/user-attachments/assets/12609155-02d8-48ab-8e77-fc97372881de" />

  Note: In this setup, the EC2 instance public IP is 174.129.146.225 and the Trino SQL Federation service is accessible on port 8080. The Trino Web UI can be accessed using http://174.129.146.225:8080

- The Trino UI is mainly used to monitor query execution, cluster status, and query history, while SQL queries are executed through external tools such as Metabase, which connects to Trino and allows users to run queries and visualize federated data

  <img width="1714" height="901" alt="image" src="https://github.com/user-attachments/assets/e19e6577-71db-4387-bd65-37485059b044" />


**Step 7 – How to access Metabase and Run Queries**
- Metabase is used as a BI and query interface to execute SQL queries against the Trino SQL Federation engine.
- Access Metabase using the following URL:
  ```text
  http://174.129.146.225:3000
  ```

  <img width="1548" height="941" alt="image" src="https://github.com/user-attachments/assets/5b3630ef-399d-49f6-ada9-5013a72ce071" />

  Note: In this setup, the EC2 instance public IP is 174.129.146.225 and the Metabase UI is accessible on port 3000. The Metabase Web UI can be accessed using http://174.129.146.225:3000

- When adding Trino in Metabase, you need to fill the connection details so Metabase can send queries to Trino.
- To connect both MySQL and PostgreSQL through Trino in Metabase, you do not need two separate database connections. You connect Metabase to Trino once, and Trino exposes both catalogs.
  ***Metabase → Trino Connection***: Use these values:
  | Field                 | Value                                 |
  | --------------------- | ------------------------------------- |
  | **Database Type**     | Trino                                 |
  | **Host**              | any_name                      |
  | **Port**              | `8080`                                |
  | **Catalog**           | `postgres` (default starting catalog) |
  | **Schema (optional)** | `public`                              |
  | **Username**          | `admin`                               |

- Now we can run queries through metabase UI under NEW --> Select SQL_query
Note: At the end of the query do not use `;` semicolon to run queries through metabase

  <img width="1000" height="200" alt="image" src="https://github.com/user-attachments/assets/65874de5-8164-4819-be25-252bf292f93e" />



- Run below example SQL Queries
  - Show connected data sources
    ```sql
    SHOW CATALOGS
    ````
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/2363bbb7-6802-4392-b4e0-fe19469503e2" />

  - Query MySQL
    ```sql
    SELECT * FROM mysql.shipments.shipments
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/2f23f388-5b86-4050-8972-a20cec898e41" />


  - Query PostgreSQL
    ```sql
    SELECT * FROM postgresql.public.customers
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/6c62e533-00ef-434c-a617-8a46e43e5457" />

    
    
  - Cross-database federation query with join
    ```sql
    SELECT
        s.shipment_id,
        s.product_name,
        c.customer_name
    FROM mysql.shipments.shipments s
    JOIN postgresql.public.customers c
    ON s.customer_id = c.id
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/d1460676-ffe8-4b07-a1c5-a31dff9af26d" />


    
  - Analytical query
    ```sql
    SELECT
        c.customer_name,
        COUNT(s.shipment_id) AS total_shipments
    FROM mysql.shipments.shipments s
    JOIN postgresql.public.customers c
    ON s.customer_id = c.id
    GROUP BY c.customer_name
    ORDER BY total_shipments DESC
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/55a48ccc-e8e6-42f1-8531-b2fba28ea58f" />


##  SSO implementation
- Keycloak is used to provide centralized authentication and Single Sign-On (SSO) for the data platform.
- In the SQL Federation architecture, multiple services such as Metabase, Trino, and other applications may need secure user access. Instead of each service managing its own user accounts, Keycloak acts as a central identity provider.

### Architecture with SSO

```
User
  │
  ▼
Keycloak (SSO Authentication with Oauth2)
  │
  ├──► Metabase (BI Tool)
  │         │
  │         ▼
  │      Trino
  │
  └──► Trino UI
            │
            ▼
     PostgreSQL / MySQL / Hive
```
**Step 1 – Running Keycloak in Development Mode (Disable HTTPS)**
- In this setup, Keycloak is started in development mode, which disables the strict HTTPS requirement. This allows services such as Trino and Metabase to communicate with Keycloak using HTTP during testing.
- In the docker-compose.yml file, the Keycloak configuration is present but currently commented out. Please uncomment the relevant code to enable it.
  ```yml
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    container_name: keycloak
    command: start-dev
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: admin
      KC_HTTP_ENABLED: true
      KC_HOSTNAME_STRICT: false
      KC_HOSTNAME_STRICT_HTTPS: false

    ports:
      - "8081:8080"
  ```
- The start-dev command runs Keycloak in development mode, However, the Keycloak UI may still display "HTTPS Required" depending on the realm configuration.
- Start keyclock using below command
```
git pull
docker-compose up -d keycloak
```


<img width="1603" height="851" alt="image" src="https://github.com/user-attachments/assets/6f9bb762-67ae-4917-8afb-4b27985749b8" />

**Note: In this setup, the EC2 instance public IP is 174.129.146.225 and the Keycloak UI is accessible on port 8081. The Keycloak Web UI can be accessed using http://174.129.146.225:8081**

**Step 2 – Run the following configuration to disable HTTPS and access the Keycloak UI**
- Enter the Keycloak Container and Run the following command to access the Keycloak container shell. This allows you to inspect configurations, run commands, or troubleshoot issues inside the container.
  ```bash
  docker exec -it keycloak /bin/bash
  ```
- Login to Keycloak Admin CLI
  - After entering the Keycloak container, use the Keycloak Admin CLI (kcadm.sh) to authenticate and manage Keycloak configurations such as realms, users, and clients.
  - Run the following command to log in to the Keycloak admin CLI:
    ```
    /opt/keycloak/bin/kcadm.sh config credentials \
    --server http://localhost:8080 \
    --realm master \
    --user admin \
    --password admin
    ```
  - This command authenticates to the Keycloak master realm using the admin credentials and allows you to perform administrative operations using the CLI.
  - Once authenticated, you can use the Keycloak Admin CLI to create realms, users, roles, and configure authentication settings.
-  Disable HTTPS Requirement in Keycloak
  -  By default, Keycloak may enforce HTTPS for secure communication. For development or testing environments, the HTTPS requirement can be disabled to allow HTTP access.
  -  Run the following command inside the Keycloak container:
     ```
     /opt/keycloak/bin/kcadm.sh update realms/master -s sslRequired=NONE
     ```
  - This command updates the master realm configuration and disables the mandatory HTTPS requirement.
  - To verify that sslRequired=NONE is applied in Keycloak after running
    ```
    /opt/keycloak/bin/kcadm.sh get realms/master | grep sslRequired
    ```

    | Value      | Meaning                   |
    | ---------- | ------------------------- |
    | `NONE`     | HTTP allowed              |
    | `EXTERNAL` | HTTPS required externally |
    | `ALL`      | HTTPS required everywhere |

  - Expected output:
    ```
    "sslRequired" : "NONE"
    ```
    <img width="896" height="129" alt="image" src="https://github.com/user-attachments/assets/2e25b96a-65ce-46d5-ac44-b2f75ab52b87" />


**⚠️ Note: Disabling HTTPS should only be done in development or testing environments. In production deployments, HTTPS must be enabled to ensure secure authentication and communication.**
  - Restart Keycloak 
    ```
    docker restart keycloak
    ```

    <img width="1199" height="525" alt="image" src="https://github.com/user-attachments/assets/ccc2f91c-8826-4647-a5e0-200089fe8fe3" />

  - After disabling the HTTPS requirement, the Keycloak Admin Console can be accessed using HTTP.
    - Use the following URL and default credentials to log in:
      ```text
      # Keycloak Web UI URL
      http://174.129.146.225:8081 

      # Username and Password
      Username: admin
      Password: admin
      ```
      

**Step 3 – How to access Trino through Keycloak (SSO)**
- To access Trino through Keycloak (SSO), the next steps are to create a realm, create a client for Trino in Keycloak, and configure Trino to use Keycloak as an OpenID Connect provider.
- Create a New Realm: Click Manage Realm → Create Realm
- In Datawave Realm Settings set SSL to "None"
  ```
  Realm Name: datawave # You can use any name
  ```
  <img width="1613" height="858" alt="image" src="https://github.com/user-attachments/assets/d78556ba-041c-4d01-bba1-8fe33806d496" />

- Create a Client for Trino: Clients → Create Client
  - Use the following values: Click Save.
    | Field       | Value                                                      |
    | ----------- | ---------------------------------------------------------- |
    | Client ID   | trino                                                      |
    | Client Type | OpenID Connect                                             |
    | Root URL    | https://174.129.146.225:8443                                |

    <img width="1499" height="666" alt="image" src="https://github.com/user-attachments/assets/d406f9ad-bdd8-44e6-9dea-37e2adc95c87" />

- Configure Client Capability Settings
  - Use the same options shown and Click Save
    ```
    | Setting               | Value   |
    | --------------------- | ------- |
    | Client Authentication | ON      |
    | Authorization         | OFF     |
    | Standard Flow         | ENABLED |
    | Direct Access Grants  | OFF     |
    | Implicit Flow         | OFF     |
    | Service Accounts      | OFF     |
    ```
- Configure Redirect URI
  
  | Field               | Value                                                         |
  | ------------------- | ------------------------------------------------------------- |
  | Valid Redirect URIs |  https://174.129.146.225:8443/ *                              |
  | Web Origins         | *                                                             |

- Copy Client Secret
  - Go to:
    ```
    Clients → trino → Credentials
    ```
  - Copy the Client Secret. We will use this in trino/config.properties
 
- Configure Trino using HTTPS
  - Genarate an SSL Self signed Certificate in EC2 with below command
    ```
    keytool -genkeypair \
    -alias trino \
    -keyalg RSA \
    -keysize 2048 \
    -validity 3650 \
    -keystore trino-keystore.jks \
    -storepass changeit \
    -keypass changeit \
    -dname "CN=174.129.146.225"
    ```
  - Run below command to move the SSL
    ```
    mv trino-keystore.jks ~/sql_federation_architecture/trino/
    ```
  - In below config.properties replace REPLACE_WITH_CLIENT_SECRET with credential-secret value and Append this code to trino/config.properties.
    ```
    # Query limits
  
    query.max-memory=2GB
    query.max-memory-per-node=512MB
    
    
    # Discovery service
    
    discovery.uri=https://174.129.146.225:8443
    
    
    # OAuth2 authentication
    
    http-server.authentication.type=oauth2
    web-ui.authentication.type=oauth2
    
    
    # HTTPS server
    
    http-server.https.enabled=true
    http-server.https.port=8443
    http-server.https.keystore.path=/etc/trino/trino-keystore.jks
    http-server.https.keystore.key=changeit
    
    
    # Keycloak
    
    http-server.authentication.oauth2.issuer=http://174.129.146.225:8081/realms/datawave
    http-server.authentication.oauth2.client-id=trino
    http-server.authentication.oauth2.client-secret= REPLACE_WITH_CLIENT_SECRET
    http-server.authentication.oauth2.scopes=openid
    http-server.authentication.oauth2.principal-field=preferred_username
    
    
    # internal communication
    
    internal-communication.shared-secret=trino-secret-123
    ```
  - Run Below commands
    ```
    git pull
    docker-compose down trino
    docker-compose up -d trino
    docker ps #Trino should up
    ```
  - 
  
  - Open Trino and It will redirect to keycloak for authentication: 
    ```
    https://174.129.146.225:8443
    ```
  - Create User and set password in Datawave realm and use those credentials to access trino.




    <img width="1420" height="947" alt="image" src="https://github.com/user-attachments/assets/9f23a853-af5e-4241-b7f1-f84cabf42caa" />

  - Metabase → Trino Connection Settings
    | Field        | Value            |
    | ------------ | ---------------- |
    | Display Name | Trino Federation |
    | Host         | trino            |
    | Port         | 8080             |
    | Catalog      | postgres         |
    | Username     | admin            |


    <img width="1457" height="727" alt="image" src="https://github.com/user-attachments/assets/be7f1db5-de18-446a-bde1-a895ace99ce3" />

    - Go to:
      ```
      Admin Settings → Authentication → OpenID Connect
      ```
      <img width="1902" height="861" alt="image" src="https://github.com/user-attachments/assets/78dd57e2-fff9-432a-b801-0584bf92a4db" />

      
    - Fill In and save
      ```
      | Field             | Value                                                                                                                                                        |
      | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
      | Client ID         | metabase                                                                                                                                                     |
      | Client Secret     | (paste from Keycloak)                                                                                                                                        |
      | Authorization URL | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/auth](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/auth)         |
      | Token URL         | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/token](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/token)       |
      | User Info URL     | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/userinfo](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/userinfo) |
      ```
    - Logout from Metabase
      - Open again:
        ```
        http://174.129.146.225:3000
        ```














**Step 3 – How to access Metabase through Keycloak (SSO)**
- To access Metabase through Keycloak (SSO), the next steps are to create a realm, create a client for Metabase in Keycloak, and configure Metabase to use Keycloak as an OpenID Connect provider.
- Create a New Realm: Click Manage Realm → Create Realm
  ```
  Realm Name: datawave # You can use any name
  ```
  <img width="1613" height="858" alt="image" src="https://github.com/user-attachments/assets/d78556ba-041c-4d01-bba1-8fe33806d496" />

- Create a Client for Metabase: Clients → Create Client
  - Use the following values: Click Save.
    | Field       | Value                                                      |
    | ----------- | ---------------------------------------------------------- |
    | Client ID   | metabase                                                   |
    | Client Type | OpenID Connect                                             |
    | Root URL    | [http://174.129.146.225:3000](http://174.129.146.225:3000) |

    <img width="1499" height="666" alt="image" src="https://github.com/user-attachments/assets/d406f9ad-bdd8-44e6-9dea-37e2adc95c87" />

- Configure Client Capability Settings
  - Use the same options shown and Click Save
    ```
    | Setting               | Value   |
    | --------------------- | ------- |
    | Client Authentication | ON      |
    | Authorization         | OFF     |
    | Standard Flow         | ENABLED |
    | Direct Access Grants  | OFF     |
    | Implicit Flow         | OFF     |
    | Service Accounts      | OFF     |
    ```
- Configure Redirect URI
  
  | Field               | Value                                                         |
  | ------------------- | ------------------------------------------------------------- |
  | Valid Redirect URIs | [http://174.129.146.225:3000/](http://174.129.146.225:3000/)* |
  | Web Origins         | *                                                             |

- Copy Client Secret
  - Go to:
    ```
    Clients → metabase → Credentials
    ```
  - Copy the Client Secret. We will use this in Metabase.
 
- Configure Metabase SSO
  - Open Metabase: 
    ```
    http://174.129.146.225:3000
    ```

    <img width="1420" height="947" alt="image" src="https://github.com/user-attachments/assets/9f23a853-af5e-4241-b7f1-f84cabf42caa" />

  - Metabase → Trino Connection Settings
    | Field        | Value            |
    | ------------ | ---------------- |
    | Display Name | Trino Federation |
    | Host         | trino            |
    | Port         | 8080             |
    | Catalog      | postgres         |
    | Username     | admin            |


    <img width="1457" height="727" alt="image" src="https://github.com/user-attachments/assets/be7f1db5-de18-446a-bde1-a895ace99ce3" />

    - Go to:
      ```
      Admin Settings → Authentication → OpenID Connect
      ```
      <img width="1902" height="861" alt="image" src="https://github.com/user-attachments/assets/78dd57e2-fff9-432a-b801-0584bf92a4db" />

      
    - Fill In and save
      ```
      | Field             | Value                                                                                                                                                        |
      | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
      | Client ID         | metabase                                                                                                                                                     |
      | Client Secret     | (paste from Keycloak)                                                                                                                                        |
      | Authorization URL | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/auth](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/auth)         |
      | Token URL         | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/token](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/token)       |
      | User Info URL     | [http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/userinfo](http://174.129.146.225:8081/realms/datawave/protocol/openid-connect/userinfo) |
      ```
    - Logout from Metabase
      - Open again:
        ```
        http://174.129.146.225:3000
        ```
      - Now you should see: Sign in with Keycloak

    - Since Metabase Community Edition does not support OpenID Connect natively, OAuth2 Proxy is deployed as an authentication gateway to integrate Keycloak SSO. OAuth2 Proxy handles authentication with Keycloak and     forwards authenticated users to Metabase.
      - Add OAuth2 Proxy Container: Update docker-compose.yml.
        ```yml
        oauth2-proxy:
          image: quay.io/oauth2-proxy/oauth2-proxy:v7.6.0
          container_name: oauth2-proxy
          ports:
            - "4180:4180"
          environment:
            OAUTH2_PROXY_PROVIDER: oidc
            OAUTH2_PROXY_CLIENT_ID: metabase
            OAUTH2_PROXY_CLIENT_SECRET: IPlMbZopeh7VMDCR7tZ0ENveFQShPWsZ
            OAUTH2_PROXY_COOKIE_SECRET: e168b6db76fdffc9d9eced4412709443
            OAUTH2_PROXY_COOKIE_SECURE: "false"
            OAUTH2_PROXY_EMAIL_DOMAINS: "*"
            OAUTH2_PROXY_OIDC_ISSUER_URL: http://174.129.146.225:8081/realms/datawave
            OAUTH2_PROXY_INSECURE_OIDC_SKIP_ISSUER_VERIFICATION: "true"
            OAUTH2_PROXY_REDIRECT_URL: http://174.129.146.225:4180/oauth2/callback
            OAUTH2_PROXY_UPSTREAMS: http://metabase:3000
            OAUTH2_PROXY_HTTP_ADDRESS: 0.0.0.0:4180
          depends_on:
            - keycloak
            - metabase
        ```
    - Access Metabase through OAuth Proxy
      - Instead of opening:
        ```
        http://174.129.146.225:3000
        ```
      - Open:
        ```
        http://174.129.146.225:4180
        ```
      - Flow:
        ```
        User → OAuth2 Proxy → Keycloak Login → Metabase
        ```


## Troubleshooting Guide
During the implementation of the SQL Federation Architecture several issues were encountered while configuring Trino, connecting Metabase to the federation engine, integrating Keycloak for authentication, and configuring OAuth2 Proxy for SSO. This section documents common problems and their solutions.

### 1. Trino Container Shows Unhealthy Status
Running:
```
docker ps
```
may show:
```
trino   Up (unhealthy)
```
**Cause**: Trino health checks may fail temporarily during startup while connectors initialize.

**Solution**

- Check Trino logs:```docker logs trino```
- If logs contain: ```SERVER STARTED```
then Trino started successfully.



### 2. Trino Cannot Connect to MySQL or PostgreSQL

**Error**

    Connection refused

**Cause**

Incorrect host configuration in Trino catalog.

**Solution**

Check configuration inside:

    trino/catalog/

Example MySQL:

    connector.name=mysql
    connection-url=jdbc:mysql://mysql:3306
    connection-user=trino
    connection-password=trino

Example PostgreSQL:

    connector.name=postgresql
    connection-url=jdbc:postgresql://postgres:5432/logistics
    connection-user=trino
    connection-password=trino

Restart Trino:

    docker restart trino

------------------------------------------------------------------------

### 3. Trino Catalog Not Visible

**Symptom**

    SHOW CATALOGS

does not show expected catalogs.

**Solution**

Ensure catalog files exist:

    mysql.properties
    postgresql.properties
    hive.properties

Restart Trino.

------------------------------------------------------------------------

### 4. Metabase Cannot Connect to Trino

**Cause**

Incorrect database connection configuration.

**Solution**

Use:

  Field           Value
  --------------- ----------
  Database Type   Trino
  Host            trino
  Port            8080
  Catalog         postgres
  Username        admin

------------------------------------------------------------------------

### 5. Metabase Cannot See Tables

Use full catalog names.

Example:

    SELECT * FROM mysql.shipments.shipments

    SELECT * FROM postgresql.public.customers

------------------------------------------------------------------------

### 6. Semicolon Not Required in Metabase SQL Editor

Incorrect:

    SELECT * FROM mysql.shipments.shipments;

Correct:

    SELECT * FROM mysql.shipments.shipments

Metabase automatically executes queries and does not require `;`.

------------------------------------------------------------------------

### 7. OAuth2 Proxy Cookie Secret Error

**Error**

    cookie_secret must be 16, 24, or 32 bytes

**Solution**

Generate:

    openssl rand -hex 16

Example:

    e168b6db76fdffc9d9eced4412709443

------------------------------------------------------------------------

### 8. OAuth2 Proxy Cannot Reach Keycloak

**Error**

    failed to discover OIDC configuration
    connection refused

Restart services:

    docker restart keycloak
    docker restart oauth2-proxy

------------------------------------------------------------------------

### 9. Invalid Client Credentials

**Error**

    unauthorized_client
    Invalid client credentials

**Solution**

Retrieve secret from:

    Keycloak → Clients → metabase → Credentials

Update docker-compose and restart oauth2-proxy.

------------------------------------------------------------------------

### 10. Keycloak Realm Does Not Exist

List realms:

    docker exec -it keycloak /opt/keycloak/bin/kcadm.sh get realms

Ensure realm name matches configuration.

Example:

    datawave

------------------------------------------------------------------------

### 11. HTTPS Required Error in Keycloak

Enter container:

    docker exec -it keycloak /bin/bash

Login CLI:

    /opt/keycloak/bin/kcadm.sh config credentials --server http://localhost:8080 --realm master --user admin --password admin

Disable HTTPS:

    /opt/keycloak/bin/kcadm.sh update realms/datawave -s sslRequired=NONE

Restart:

    docker restart keycloak

------------------------------------------------------------------------

### 12. Authentication Loop

Clear browser cookies or open incognito window.

Ensure:

    OAUTH2_PROXY_COOKIE_SECURE=false

------------------------------------------------------------------------

### 13. Docker Troubleshooting Commands

Check containers:

    docker ps

Check all containers:

    docker ps -a

View logs:

    docker logs trino
    docker logs metabase
    docker logs keycloak
    docker logs oauth2-proxy

Recent logs:

    docker logs trino --tail 50

Restart services:

    docker restart trino
    docker restart metabase
    docker restart keycloak
    docker restart oauth2-proxy

Restart stack:

    docker-compose restart

Stop stack:

    docker-compose down

Start stack:

    docker-compose up -d

------------------------------------------------------------------------

### 14. Verify Trino CLI

    docker exec -it trino trino

Run:

    SHOW CATALOGS
    SHOW SCHEMAS FROM mysql
    SHOW TABLES FROM postgresql.public

------------------------------------------------------------------------

### 15. Verify Container Networking

    docker exec -it trino ping mysql
    docker exec -it trino ping postgres
    docker exec -it trino ping keycloak

------------------------------------------------------------------------

### Summary

These troubleshooting steps help diagnose:

-   Trino federation issues
-   Metabase query problems
-   Keycloak authentication configuration
-   OAuth2 Proxy SSO integration
-   Docker container networking


## Improvements and Conclusion
While the current SQL Federation architecture successfully demonstrates unified querying across heterogeneous data sources, authentication with Keycloak SSO, and analytics through Metabase, several improvements can
further enhance scalability, security, and operational reliability for production deployments.

### Future Improvements

### 1. CI/CD Pipeline Implementation
- Currently the platform is deployed manually using Docker Compose. In a production environment, deployment should be automated using a **CI/CD pipeline**.
- Possible CI/CD tools:
  -   GitLab CI/CD
  -   GitHub Actions
  -   Jenkins
- Example CI/CD workflow:
    ```
    Developer Commit → Git Repository → CI/CD Pipeline → Build Docker Images
    → Run Tests → Security Scans → Push Images → Deploy to Kubernetes
    ```
- Benefits:
  -   Automated deployments
  -   Faster release cycles
  -   Reduced manual errors
  -   Continuous testing and validation

### 2. Migration to Kubernetes
The current deployment runs all services on a single host using Docker Compose. For production workloads, the platform should be migrated to **Kubernetes**.
- Benefits:
  -   Auto‑scaling of services
  -   Self‑healing containers
  -   Rolling updates
  -   High availability
  -   Better resource management
- Suggested Kubernetes architecture:
  ```
  Kubernetes Cluster - Trino Deployment - MySQL StatefulSet - PostgreSQL
  StatefulSet - Metabase Deployment - Keycloak Deployment - OAuth2 Proxy
  Deployment
  ```
- Databases should run as **StatefulSets** while application services run as **Deployments**.

### 3. Secure Secret Management (AWS Secrets Manager)
Currently credentials such as database passwords and client secrets are stored in configuration files or environment variables.
- In production environments, secrets should be managed using a secure secret management system such as:
  -   AWS Secrets Manager
  -   HashiCorp Vault
  -   Kubernetes Secrets
-   Secrets that should be protected:
  -   Database credentials
  -   OAuth client secrets
  -   Keycloak admin credentials
  -   API keys
-   Benefits:
  -   Secure storage of credentials
  -   Centralized secret management
  -   Secret rotation support
  -   Reduced risk of credential exposure

### 4. SSL/TLS Security
Currently services communicate over HTTP for development purposes. Production systems should enable **HTTPS with SSL/TLS encryption**.
- Recommended improvements:
  -   Use AWS ACM or Let's Encrypt certificates
  -   Enable HTTPS for Keycloak authentication endpoints
  -   Secure Metabase dashboards
  -   Protect Trino APIs
- Example secure architecture:
    ```
    User → HTTPS Load Balancer → Metabase / Keycloak / Trino
    ```
- Bnefits:
  -   Secure authentication
  -   Encrypted communication
  -   Protection against man‑in‑the‑middle attacks


### 5. Monitoring and Observability
Operational visibility is critical in production environments.
- Recommended tools:
  -   Prometheus -- metrics collection
  -   Grafana -- monitoring dashboards
  -   Elasticsearch -- centralized logging
  -   Alertmanager -- alert notifications
-   Key metrics to monitor:
  -   Query performance
  -   Trino cluster health
  -   Database connectivity
  -   Authentication failures
  -   Resource usage
-   Benefits:
  -   Real‑time system visibility
  -   Faster troubleshooting
  -   Performance optimization

### 6. Trino Cluster Scaling
Currently Trino runs as a single node. Production deployments should run **distributed Trino clusters**.
- Example architecture:
  - Trino Coordinator 
    - Worker Node 1
    - Worker Node 2
    - Worker Node 3
- Benefits:
  - Parallel query execution
  - Higher performance
  - Horizontal scalability

## Conclusion
This project demonstrates the design and implementation of a **modern SQL Federation architecture** capable of querying multiple heterogeneous data sources through a unified interface.
**Key capabilities implemented:**
  -   SQL federation using Trino
  -   Integration of multiple databases such as MySQL, PostgreSQL, Minio and S3
  -   Centralized authentication and Single Sign‑On using Keycloak
  -   Business intelligence dashboards using Metabase
  -   Containerized deployment using Docker Compose

