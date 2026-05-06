# Docker Compose Deployment Guide

A guide to running the full CloudBlitz Student Registration System (Frontend + Backend) using a single `docker-compose.yml` file on AWS EC2.

---

## Prerequisites

- AWS EC2 instance running Ubuntu
- Docker and Docker Compose installed
- Port **80** and **8080** open in EC2 Security Group inbound rules
- Your project cloned from GitHub

---

## Step 1 — Install Docker

```bash
apt update && apt install docker.io -y
systemctl start docker
systemctl enable docker
```

### Verify Docker installation

```bash
docker --version
```

---

## Step 2 — Install Docker Compose

```bash
apt install docker-compose -y
```

### Verify Docker Compose installation

```bash
docker-compose --version
```

---

## Step 3 — Clone Your GitHub Repo

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

---

## Step 4 — Configure Backend Before Building

Update DB credentials in `application.properties` before running Compose:

```bash
vim backend/src/main/resources/application.properties
```

```properties
server.port=8080
spring.datasource.url=jdbc:mariadb://<DB_HOST>:3306/student_db
spring.datasource.username=<DB_USER>
spring.datasource.password=<DB_PASS>
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Update frontend API URL in `.env`:

```bash
vim frontend/.env
```

```env
VITE_API_URL="http://<EC2_PUBLIC_IP>:8080/api"
```

---

## Step 5 — Project Structure

Make sure your project has the following structure before running Compose:

```
your-repo/
├── docker-compose.yml        ← Compose file (root level)
├── backend/
│   ├── dockerfile            ← Backend Dockerfile
│   ├── pom.xml
│   └── src/
│       └── main/resources/
│           └── application.properties
└── frontend/
    ├── dockerfile            ← Frontend Dockerfile
    ├── .env
    └── src/
```

---

## Step 6 — The docker-compose.yml File

```yaml
version: "3.8"

services:

  backend:
    build:
      context: ./backend
      dockerfile: dockerfile
    ports:
      - 8080:8080

  frontend:
    build:
      context: ./frontend
      dockerfile: dockerfile
    ports:
      - 80:80
    depends_on:
      - backend
```

### What each section means

| Section | Description |
|---|---|
| `version: "3.8"` | Docker Compose file format version |
| `services` | Defines all containers to run |
| `backend.build.context` | Path to the backend folder containing the Dockerfile |
| `backend.ports: 8080:8080` | Maps EC2 port 8080 → container port 8080 |
| `frontend.build.context` | Path to the frontend folder containing the Dockerfile |
| `frontend.ports: 80:80` | Maps EC2 port 80 → container port 80 |
| `depends_on: backend` | Frontend container starts only after backend container is up |

> **Note:** The `db` service is currently commented out. If you are using AWS RDS or an external MariaDB instance, this is correct — the DB is managed separately and only the connection string in `application.properties` is needed.

---

## Step 7 — Build and Run All Services

```bash
docker-compose up --build -d
```

- `--build` — rebuilds Docker images from Dockerfiles
- `-d` — runs containers in detached (background) mode

---

## Step 8 — Verify Running Containers

```bash
docker ps
```

Expected output:

```
CONTAINER ID   IMAGE              PORTS                    NAMES
xxxxxxxxxxxx   your-repo-frontend 0.0.0.0:80->80/tcp       your-repo-frontend-1
xxxxxxxxxxxx   your-repo-backend  0.0.0.0:8080->8080/tcp   your-repo-backend-1
```

---

## Step 9 — Access the Application

| Service | URL |
|---|---|
| Frontend | `http://<EC2_PUBLIC_IP>:80` |
| Backend API | `http://<EC2_PUBLIC_IP>:8080/api` |

---

## Useful Docker Compose Commands

### View live logs (all services)

```bash
docker-compose logs -f
```

### View logs for a specific service

```bash
docker-compose logs -f backend
docker-compose logs -f frontend
```

### Stop all containers

```bash
docker-compose down
```

### Stop and remove images too

```bash
docker-compose down --rmi all
```

### Restart a specific service

```bash
docker-compose restart backend
```

### Rebuild a single service without restarting others

```bash
docker-compose up --build -d backend
```

---

## Quick Reference — All Commands in Order

```bash
# 1. Install Docker and Compose
apt update && apt install docker.io docker-compose -y
systemctl start docker && systemctl enable docker

# 2. Clone repo
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>

# 3. Configure credentials
vim backend/src/main/resources/application.properties
vim frontend/.env

# 4. Build and run all services
docker-compose up --build -d

# 5. Verify containers
docker ps

# 6. View logs
docker-compose logs -f
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `docker-compose: command not found` | Run `apt install docker-compose -y` |
| Port 80 already in use | Stop Apache2: `systemctl stop apache2` |
| Port 8080 already in use | Run `kill $(lsof -t -i:8080)` |
| Frontend can't reach backend | Check `VITE_API_URL` in `frontend/.env` has correct EC2 IP |
| Backend can't connect to DB | Check `application.properties` DB credentials and Security Group port 3306 |
| Container exits immediately | Run `docker-compose logs backend` to see the error |
| Changes not reflected after edit | Run `docker-compose up --build -d` to rebuild |

---

*Guide covers: Docker + Docker Compose install · multi-service setup · build & run · logs · useful commands · troubleshooting*