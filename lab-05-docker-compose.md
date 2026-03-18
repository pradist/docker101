# Lab 5: Docker Compose & Environment Variables

> **Estimated time:** 25 minutes
> **Difficulty:** ⭐⭐⭐ Intermediate

---

## Objectives

By the end of this lab, you will be able to:

- Write a `docker-compose.yml` file to define multi-container applications
- Use `docker compose up` and `docker compose down`
- Pass environment variables into containers
- Connect multiple services together using Docker Compose networking

---

## Prerequisites

- Completed [Lab 4: Build Your Own Image](lab-04-dockerfile.md)
- Google Cloud Shell open: [https://ssh.cloud.google.com](https://ssh.cloud.google.com)

> 💡 `docker compose` is pre-installed in Cloud Shell.

---

## Concepts First

So far, every `docker run` command is one container at a time.
Real applications often need **multiple containers working together** — for example:

```text
┌────────────────┐      ┌────────────────┐
│   Web App      │─────▶│   Database     │
│  (my-app)      │      │  (PostgreSQL)  │
└────────────────┘      └────────────────┘
```

**Docker Compose** lets you define all of these in a single `docker-compose.yml` file and manage them with one command.

| Concept | Meaning |
| ------- | ------- |
| **Service** | A container defined in `docker-compose.yml` |
| **Environment variable** | A key=value setting passed into a container at runtime |
| **`docker compose up`** | Start all services defined in the file |
| **`docker compose down`** | Stop and remove all containers (keeps images and volumes by default) |

---

## Steps

### Step 1 — Create a project folder

```bash
mkdir compose-demo
cd compose-demo
```

---

### Step 2 — Create the web app

Create `app.py`:

```python
# app.py
import os
from http.server import HTTPServer, BaseHTTPRequestHandler

APP_ENV  = os.environ.get("APP_ENV", "development")
APP_NAME = os.environ.get("APP_NAME", "MyApp")

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        msg = f"Hello from {APP_NAME}! Environment: {APP_ENV}\n"
        self.wfile.write(msg.encode())

    def log_message(self, format, *args):
        pass

if __name__ == "__main__":
    print(f"Starting {APP_NAME} in {APP_ENV} mode on port 8080...")
    HTTPServer(("", 8080), Handler).serve_forever()
```

---

### Step 3 — Create the Dockerfile

Create `Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8080
CMD ["python", "app.py"]
```

---

### Step 4 — Create the docker-compose.yml

Create `docker-compose.yml`:

```yaml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - APP_ENV=production
      - APP_NAME=Docker101App
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

---

### Step 5 — Understand the docker-compose.yml

```yaml
services:          # list of containers (services) to run
  web:             # service name — can be anything
    build: .       # build this service from the Dockerfile in current folder
    ports:
      - "8080:8080"          # host:container port mapping
    environment:             # environment variables passed into the container
      - APP_ENV=production
      - APP_NAME=Docker101App
    restart: unless-stopped  # auto-restart on crash, unless manually stopped

  redis:           # second service — a Redis cache
    image: redis:7-alpine    # use this pre-built image (no Dockerfile needed)
    ports:
      - "6379:6379"
```

| Key | What it does |
| --- | ------------ |
| `services` | Defines all containers in this stack |
| `build: .` | Build image from Dockerfile in current directory |
| `image: redis:7-alpine` | Pull and use this existing image |
| `ports` | Map host ports to container ports |
| `environment` | Set environment variables inside the container |
| `restart: unless-stopped` | Restart policy for fault tolerance |

---

### Step 6 — Verify your project structure

```text
compose-demo/
├── app.py
├── Dockerfile
└── docker-compose.yml
```

---

### Step 7 — Start all services

```bash
docker compose up
```

> 💡 **Note:** Add `-d` to run in the background:

```bash
docker compose up -d
```

**Expected output:**

```text
[+] Building 8.2s (6/6) FINISHED
[+] Running 2/2
 ✔ Container compose-demo-redis-1  Started
 ✔ Container compose-demo-web-1    Started
```

---

### Step 8 — Test the web service

```bash
curl http://localhost:8080
```

```text
Hello from Docker101App! Environment: production
```

The `APP_ENV` and `APP_NAME` values from `docker-compose.yml` are now live inside the container.

---

### Step 9 — List running services

```bash
docker compose ps
```

```bash
NAME                      IMAGE                  COMMAND           SERVICE   STATUS    PORTS
compose-demo-redis-1      redis:7-alpine         "docker-entryp…"  redis     running   0.0.0.0:6379->6379/tcp
compose-demo-web-1        compose-demo-web       "python app.py"   web       running   0.0.0.0:8080->8080/tcp
```

---

### Step 10 — View logs

```bash
docker compose logs
```

To follow logs in real-time:

```bash
docker compose logs -f
```

Press `Ctrl + C` to stop following.

---

### Step 11 — Use an .env file (best practice)

Instead of hardcoding values in `docker-compose.yml`, store them in a `.env` file:

Create `.env`:

```text
APP_ENV=staging
APP_NAME=Docker101App
```

Update `docker-compose.yml` to reference the `.env` variables:

```yaml
    environment:
      - APP_ENV=${APP_ENV}
      - APP_NAME=${APP_NAME}
```

Docker Compose automatically reads `.env` from the same folder.

> **Security tip:** Add `.env` to `.gitignore` — never commit secrets to version control.

---

### Step 12 — Stop everything

```bash
docker compose down
```

```bash
[+] Running 3/3
 ✔ Container compose-demo-web-1    Removed
 ✔ Container compose-demo-redis-1  Removed
 ✔ Network compose-demo_default    Removed
```

All containers are removed. Images and volumes are kept.
To also remove images: `docker compose down --rmi all`

---

## What Just Happened?

```text
docker compose up
      │
      ├── Reads docker-compose.yml
      ├── Builds images (for services with build:)
      ├── Pulls images (for services with image:)
      ├── Creates a shared network automatically
      └── Starts all containers

Services can reach each other by service name:
  web → redis:6379   (not localhost:6379)
```

---

## Key Takeaways

| Command | What it does |
| ------- | ------------ |
| `docker compose up` | Build (if needed), create, and start all services |
| `docker compose up -d` | Same, but in background |
| `docker compose down` | Stop and remove all containers and network |
| `docker compose ps` | List services and their status |
| `docker compose logs` | View logs from all services |
| `environment:` | Pass env vars directly in compose file |
| `.env` file | Better practice — keep secrets separate from config |

---

## Quick Reference

```bash
# Start all services (foreground)
docker compose up

# Start all services (background)
docker compose up -d

# View status
docker compose ps

# View logs
docker compose logs -f

# Stop and remove containers
docker compose down

# Stop, remove containers AND images
docker compose down --rmi all
```

---

## Congratulations

You have completed the Docker 101 Lab Series!

| Lab | Topic | Status |
| --- | ----- | ------ |
| Lab 1 | Your First Container | ✅ |
| Lab 2 | Managing Containers | ✅ |
| Lab 3 | Working with Images | ✅ |
| Lab 4 | Build Your Own Image | ✅ |
| Lab 5 | Docker Compose & Env Vars | ✅ |

---

## Navigation

- Previous: [Lab 4: Build Your Own Image](lab-04-dockerfile.md)
- [Home](README.md)
