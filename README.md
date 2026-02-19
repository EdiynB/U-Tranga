# Tranga Unified Project

This repository contains both the Backend and Frontend components of the Tranga project.
Run on an Ubuntu 22.04 server

- **Backend**: [tranga-fork-main](./tranga-fork-main)
- **Frontend**: [tranga-yacobGUI-fork-main](./tranga-yacobGUI-fork-main)

---

# Deployment Guide (Ubuntu Server)

Follow these steps to deploy your unified Tranga project on a headless Ubuntu server using Docker.

## 1. Prerequisites
Ensure your server has Git, Docker, and Docker Compose installed.

```bash
# Update package list
sudo apt update

# Install Git, Docker, and Docker Compose
sudo apt install git docker.io docker-compose-v2 -y

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

## 2. Clone the Repository
Clone this repository to your server.

```bash
git clone https://github.com/EdiynB/U-Tranga.git
cd U-Tranga
```

## 3. Configuration
The project is split into `tranga-fork-main` (Backend) and `tranga-yacobGUI-fork-main` (Frontend). To run your specific code, we will use a unified Docker Compose file that builds from source.

Create a `docker-compose.yml` file in the root of the `U-Tranga` directory:

```yaml
services:
  tranga-api:
    build:
      context: ./tranga-fork-main
      dockerfile: Dockerfile
    container_name: tranga-api
    volumes:
      - ./Manga:/Manga
      - ./settings:/usr/share/tranga-api
    ports:
      - "6531:6531"
    depends_on:
      tranga-pg:
        condition: service_healthy
    environment:
      - POSTGRES_HOST=tranga-pg
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    restart: unless-stopped

  tranga-website:
    build:
      context: ./tranga-yacobGUI-fork-main
      dockerfile: Dockerfile
    container_name: tranga-website
    ports:
      - "9555:80"
    environment:
      - API_HOST=http://<YOUR_SERVER_IP>:6531
    depends_on: 
      - tranga-api
    restart: unless-stopped

  tranga-pg:
    image: postgres:17
    container_name: tranga-pg
    environment:
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_USER=postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready", "-U", "postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
```

> [!IMPORTANT]
> Replace `<YOUR_SERVER_IP>` with the actual IP address of your Ubuntu server so the frontend can communicate with the API.

## 4. Run the Application
Start the containers in detached mode. This will build the images from your source code.

```bash
sudo docker compose up -d --build
```

## 5. Accessing the App
- **Frontend**: `http://<YOUR_SERVER_IP>:9555`
- **API Swagger**: `http://<YOUR_SERVER_IP>:6531/swagger`

## 6. Maintenance
To stop the app:
```bash
sudo docker compose down
```

To see logs:
```bash
sudo docker compose logs -f
```
