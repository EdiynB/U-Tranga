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

On a headless server, you'll need to create a `docker-compose.yml` file in the root of the `U-Tranga` directory. This file tells Docker how to build and connect your backend and frontend.

### A. Find your Server's IP
You need your server's local IP address so the frontend (running in your browser) knows where to find the backend API.
```bash
hostname -I | awk '{print $1}'
```
*Note the IP address that appears (e.g., 192.168.1.50).*

### B. Create the Compose File
Use a text editor like `nano` to create the file:
```bash
nano docker-compose.yml
```

Paste the following content into the editor:

```yaml
services:
  tranga-api:
    build:
      context: ./tranga-fork-main
      dockerfile: Dockerfile
    container_name: tranga-api
    volumes:
      # Stores your downloaded manga
      - ./Manga:/Manga
      # Stores your application settings and database config
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
      # CRITICAL: Replace <YOUR_SERVER_IP> with the IP from Step 3A
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

### C. Save and Exit
- Press `Ctrl + O` then `Enter` to save.
- Press `Ctrl + X` to exit the editor.

> [!IMPORTANT]
> The `API_HOST` must be a URL that your **own computer's browser** can reach. If you are accessing the server over a VPN or local network, use the internal IP. If you are using a domain name, use that instead.

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
