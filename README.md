#

















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
