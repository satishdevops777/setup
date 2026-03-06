# Project Execution

## Setup Instructions

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
```

**Step 2 — How to Clone the Repository**
```
git clone https://gitlab.com/satishdevops777/sql_federation_architecture.git && cd sql_federation_architecture
```
<img width="1725" height="288" alt="image" src="https://github.com/user-attachments/assets/68a33999-6f01-449f-9837-fd5575dc07b7" />


**Step 3 — How to Start the Platform**
Start all services using Docker Compose.
```bash
docker-compose up -d
```
<img width="1908" height="223" alt="image" src="https://github.com/user-attachments/assets/d56e1d10-995c-4a72-bd10-fc3f3ab60215" />



**Step 4 - How to Verify that all containers are running and inspect logs if any service is not functioning correctly.**
```
# List all running containers
docker ps

# Check logs of a specific container
docker logs <container_name> --tail 50
```
<img width="1919" height="511" alt="image" src="https://github.com/user-attachments/assets/ad9bef7f-dce4-422d-9aaa-2a62064d9530" />


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
  - Show connected data sources
    ```sql
    SHOW CATALOGS; 
    ````
    <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/92b60b81-2f8e-46d2-a435-439d491d8b5f" />

  - Query MySQL
    ```sql
    SELECT * FROM mysql.shipments.shipments;
    ```
    <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/fe4d40a1-4a13-48f1-9e26-dce687b000b7" />

  - Query PostgreSQL
    ```sql
    SELECT * FROM postgresql.public.customers;
    ```
    <img width="948" height="250" alt="image" src="https://github.com/user-attachments/assets/d3b34d9f-08fc-4bb7-a9e5-759f932a6c97" />

    
    
  - Cross-database federation query with join
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

    
  - Analytical query
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
  ```test
  http://18.232.99.36:8080/
  ```
- Trino requires a user name to identify the query executor. In this setup authentication is not enabled, so any username can be provided to access the Trino UI.
  
  <img width="1091" height="300" alt="image" src="https://github.com/user-attachments/assets/12609155-02d8-48ab-8e77-fc97372881de" />

  Note: Note: In this setup, the EC2 instance public IP is 18.232.99.36 and the Trino SQL Federation service is accessible on port 8080. The Trino Web UI can be accessed using http://18.232.99.36:8080

- The Trino UI is mainly used to monitor query execution, cluster status, and query history, while SQL queries are executed through external tools such as Metabase, which connects to Trino and allows users to run queries and visualize federated data

  <img width="1714" height="901" alt="image" src="https://github.com/user-attachments/assets/e19e6577-71db-4387-bd65-37485059b044" />


**Step 7 – How to access Metabase and Run Queries**
- Metabase is used as a BI and query interface to execute SQL queries against the Trino SQL Federation engine.
- Access Metabase using the following URL:
  ```text
  http://18.232.99.36:3000
  ```

  <img width="1548" height="941" alt="image" src="https://github.com/user-attachments/assets/5b3630ef-399d-49f6-ada9-5013a72ce071" />

  Note: Note: In this setup, the EC2 instance public IP is 18.232.99.36 and the Metabase UI is accessible on port 3030. The Metabase Web UI can be accessed using http://18.232.99.36:3030

- When adding Trino in Metabase, you need to fill the connection details so Metabase can send queries to Trino.
- To connect both MySQL and PostgreSQL through Trino in Metabase, you do not need two separate database connections. You connect Metabase to Trino once, and Trino exposes both catalogs.
  ***Metabase → Trino Connection***: Use these values:
  | Field                 | Value                                 |
  | --------------------- | ------------------------------------- |
  | **Database Type**     | Trino                                 |
  | **Host**              | `18.232.99.36`                        |
  | **Port**              | `8080`                                |
  | **Catalog**           | `postgres` (default starting catalog) |
  | **Schema (optional)** | `public`                              |
  | **Username**          | `admin`                               |

- Now we can run queries through metabase under NEW --> Select SQL_query

  <img width="1775" height="405" alt="image" src="https://github.com/user-attachments/assets/65874de5-8164-4819-be25-252bf292f93e" />




- Run below example SQL Queries
  - Show connected data sources
    ```sql
    SHOW CATALOGS; 
    ````
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/2363bbb7-6802-4392-b4e0-fe19469503e2" />

  - Query MySQL
    ```sql
    SELECT * FROM mysql.shipments.shipments;
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/2f23f388-5b86-4050-8972-a20cec898e41" />


  - Query PostgreSQL
    ```sql
    SELECT * FROM postgresql.public.customers;
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
    ON s.customer_id = c.id;
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
    ORDER BY total_shipments DESC;
    ```
    <img width="589" height="580" alt="image" src="https://github.com/user-attachments/assets/55a48ccc-e8e6-42f1-8531-b2fba28ea58f" />



