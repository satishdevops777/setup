# Project Execution

## Setup Instructions

**Step 1 — How to Clone the Repository**
```
git clone https://gitlab.com/satishdevops777/sql_federation_architecture.git && cd sql_federation_architecture
```
<img width="1725" height="288" alt="image" src="https://github.com/user-attachments/assets/68a33999-6f01-449f-9837-fd5575dc07b7" />


**Step 2 — How to Start the Platform**
Start all services using Docker Compose.
```bash
docker-compose up -d
```
<img width="1908" height="223" alt="image" src="https://github.com/user-attachments/assets/d56e1d10-995c-4a72-bd10-fc3f3ab60215" />



**Step 3 - How to Verify that all containers are running and inspect logs if any service is not functioning correctly.**
```
# List all running containers
docker ps

# Check logs of a specific container
docker logs <container_name> --tail 50
```
<img width="1919" height="511" alt="image" src="https://github.com/user-attachments/assets/ad9bef7f-dce4-422d-9aaa-2a62064d9530" />


**Step 4 - How to Access Service URLs**
- Service Access
- After deployment, the services can be accessed at:

| Service  | URL                                            | Description         |
| -------- | ---------------------------------------------- | ------------------- |
| Trino    | [http://localhost:8080](http://localhost:8080) | SQL query engine    |
| Metabase | [http://localhost:3000](http://localhost:3000) | BI dashboards       |
| Keycloak | [http://localhost:8081](http://localhost:8081) | Authentication      |

**Note: Note: When deploying this setup on an AWS EC2 instance, replace localhost with the EC2 instance's public IP address to access the services from your browser or external systems.**
