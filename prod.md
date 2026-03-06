## Project Execution
- After the Prerequisites section, add:

Step 1 — Clone the Repository
```
git clone https://github.com/your-repository/sql-federation-project.git
cd sql-federation-project
```

Step 2 — Start the Platform
Start all services using Docker Compose.
```bash
docker-compose up -d
```

Step 3 - Verify Containers
```
docker ps
```
Expected running services:
```yml
trino
postgres
mysql
minio
hive-metastore
metabase
keycloak
ranger
```
Step 4 - Service Access URLs
- Service Access
- After deployment, the services can be accessed at:

| Service  | URL                                            | Description         |
| -------- | ---------------------------------------------- | ------------------- |
| Trino    | [http://localhost:8080](http://localhost:8080) | SQL query engine    |
| Metabase | [http://localhost:3000](http://localhost:3000) | BI dashboards       |
| MinIO    | [http://localhost:9001](http://localhost:9001) | Object storage UI   |
| Keycloak | [http://localhost:8081](http://localhost:8081) | Authentication      |
| Ranger   | [http://localhost:6080](http://localhost:6080) | Governance policies |


