# DATAWAVE INDUSTRIES - SQL FEDERATION ARCHITECTURE 
---
## Problem Summary
DataWave currently runs analytics on an old on-prem Hadoop cluster that has several issues:

**Key Problems**
- Scalability limits – Cannot scale easily as data grows.
- Performance bottlenecks – Queries are slow with increasing workloads.
- Availability issues – Cluster instability and downtime.
- Operational overhead – Requires significant maintenance.
Because of this, the company wants to modernize their data platform in the cloud.

## Architecture Overview

The DataWave SQL Federation Architecture enables unified querying across multiple heterogeneous data sources using Trino as the federation engine. It integrates governance, authentication, analytics, and auditing components to provide a secure and scalable data platform.

<img width="644" height="324" alt="image" src="https://github.com/user-attachments/assets/71a8577e-188a-48ac-ba8f-f89895d57fb0" />

### Core Components
**1. Data Sources (Heterogeneous Databases)**
- Data sources are the systems where the actual data is stored. In this architecture, multiple types of databases are connected to the federation engine.
- Examples of data sources include:
  ```
    PostgreSQL – stores structured relational data
    MySQL – stores operational data
    Object storage (MinIO / S3) – stores large datasets or files
    Other databases – such as Hive, Elasticsearch, or analytics systems
    ```
    **Purpose**
    - These systems hold the raw data used by applications and business teams.
    
    **Role in the Architecture**
    - Instead of copying or moving the data into one centralized database, the architecture allows querying data directly from these systems using the federation engine.
    
    **Benefit**
    - This reduces data duplication and allows real-time access to multiple data sources.


**2. Trino (SQL Federation Engine)**
- Trino is the central query engine in this architecture. It allows users to run SQL queries across multiple databases as if they were a single system.
- Responsibilities
    - Connects to different data sources using connectors
    - Executes distributed SQL queries
    - Combines results from multiple systems
    - Provides a unified query interface
- Example:
    - A query can combine data from MySQL and PostgreSQL:
    ```
    SELECT s.shipment_id, l.location
    FROM mysql.shipments.shipments s
    JOIN postgres.logistics.locations l
    ON s.location_id = l.id;
    ```
    **Role in Architecture**
    - Trino sits between the users and the data sources and acts as a federation layer.
    
    **Benefit**
    - Users can query multiple systems using one SQL interface.

**3. Apache Ranger (Data Governance and Access Control)**
- Apache Ranger is responsible for security and data governance in the system.
- Responsibilities
    - Manage access control policies
    - Provide role-based access control
    - Enforce data security policies
    - Manage fine-grained permissions
- Example Policies
    ```
    Allow analysts to read only specific tables
    Restrict access to sensitive columns
    Limit data access based on user roles
    ```
    **Role in Architecture**
    - Ranger integrates with Trino to check whether a user is allowed to access certain data before executing the query.
    
    **Benefit**
    - Ensures secure and controlled access to sensitive data


**4. Keycloak (Authentication and Single Sign-On)**
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
      
    **Role in Architecture**
        - Keycloak ensures that only authenticated users can access the data platform.
        
    **Benefit**
        - Provides centralized and secure user authentication.















## Pre-requisites Docker, Docker Compose, and Git Installation Guide (Amazon Linux)
This guide explains how to install **Docker**, **Docker Compose**, and **Git** on **Amazon Linux (Amazon Linux 2023 / Amazon Linux 2)** using the `dnf` package manager.

------------------------------------------------------------------------
### Prerequisites
Before starting, ensure:
-   Amazon Linux instance (EC2 or local VM)
-   `sudo` privileges
-   Internet connectivity
-   `dnf` package manager available
------------------------------------------------------------------------
### 1. Update System Packages
Update all installed packages to the latest version.
``` bash
sudo dnf update -y
```
------------------------------------------------------------------------

### 2. Install Docker
Install Docker from the Amazon Linux repository.
``` bash
sudo dnf install docker -y
```
------------------------------------------------------------------------
### 3. Start Docker Service
Start Docker.
``` bash
sudo systemctl start docker
```
Enable Docker to start automatically at boot.
``` bash
sudo systemctl enable docker
```
Check Docker status.
``` bash
sudo systemctl status docker
```
Expected output:
```bash
Active: active (running)
```
------------------------------------------------------------------------

### 4. Run Docker Without sudo (Optional)
Add your user to the Docker group.
``` bash
sudo usermod -aG docker $USER
```
Apply the group change.
``` bash
newgrp docker
```
------------------------------------------------------------------------

### 5. Verify Docker Installation
Check Docker version.
``` bash
docker --version
```
Example:
```bash
Docker version 25.x
```
------------------------------------------------------------------------

### 6. Install Docker Compose
Amazon Linux may not include Docker Compose by default, so install it
manually.
Download Docker Compose.
``` bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
Provide execution permission.
``` bash
sudo chmod +x /usr/local/bin/docker-compose
```
------------------------------------------------------------------------
### 7. Verify Docker Compose Installation
Check Docker Compose version.
``` bash
docker-compose --version
```
Example:
``` bash
    Docker Compose version v2.x.x
```
------------------------------------------------------------------------
### 8. Install & Verify Git
Install Git using DNF.
``` bash
sudo dnf install git -y
```
Check Git version.
``` bash
git --version
```
Example:
```bash
    git version 2.x.x
```
------------------------------------------------------------------------

### Conclusion
You have successfully installed:
-   Docker
-   Docker Compose
-   Git
Your system is now ready to run **containerized applications, CI/CD
pipelines, and DevOps workflows on Amazon Linux**.
