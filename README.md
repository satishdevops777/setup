# Pre-requisites Docker, Docker Compose, and Git Installation Guide

This guide explains how to install **Docker**, **Docker Compose**, and **Git** on **Red Hat Enterprise Linux (RHEL)** or RHEL-based
distributions using the **DNF package manager**.

------------------------------------------------------------------------

## Prerequisites

Before installing ensure the following:

-   RHEL 8 / RHEL 9 or compatible distribution
-   `sudo` access
-   Internet connectivity
-   `dnf` package manager available

------------------------------------------------------------------------

## 1. Update System Packages

Update all system packages.

``` bash
sudo dnf update -y
```

------------------------------------------------------------------------

## 2. Install Required Utilities

Install repository management tools.

``` bash
sudo dnf install -y dnf-plugins-core
```

------------------------------------------------------------------------

## 3. Install Docker

Install Docker Engine.

``` bash
sudo dnf install docker -y
```

------------------------------------------------------------------------

## 4. Start and Enable Docker

Start Docker service.

``` bash
sudo systemctl start docker
```

Enable Docker on system boot.

``` bash
sudo systemctl enable docker
```

Check Docker service status.

``` bash
sudo systemctl status docker
```

Expected output:

    Active: active (running)

------------------------------------------------------------------------

## 5. Run Docker Without sudo (Optional but Recommended)

Add your user to the Docker group.

``` bash
sudo usermod -aG docker $USER
```

Apply the new group permissions.

``` bash
newgrp docker
```

------------------------------------------------------------------------

## 6. Verify Docker Installation

Check Docker version.

``` bash
docker --version
```

Example:

    Docker version 25.x.x

------------------------------------------------------------------------

## 7. Install and Verify Docker Compose

Download the latest Docker Compose binary.

``` bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

Provide execute permission.

``` bash
sudo chmod +x /usr/local/bin/docker-compose
```
Check version.

``` bash
docker-compose --version
```

Example output:

    Docker Compose version v2.x.x

------------------------------------------------------------------------

## 8. Install Git & Verify Git Installation

Install Git using DNF.

``` bash
sudo dnf install git -y
```

Check Git version.

``` bash
git --version
```

Example output:

    git version 2.x.x

Configure Git user details.

``` bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Verify configuration.

``` bash
git config --list
```

------------------------------------------------------------------------

## Installation Verification Summary

  Component        Verification Command
  ---------------- ----------------------------
  Docker           `docker --version`
  Docker Compose   `docker-compose --version`
  Git              `git --version`

------------------------------------------------------------------------


You have successfully installed:

-   Docker Engine
-   Docker Compose
-   Git

Your system is now ready to run **containerized applications, CI/CD
pipelines, and DevOps workflows**.
