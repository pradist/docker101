# Lab 4: Build Your Own Image

> **Estimated time:** 20 minutes
> **Difficulty:** ⭐⭐ Beginner–Intermediate

---

## Objectives

By the end of this lab, you will be able to:

- Write a `Dockerfile` to define a custom image
- Understand the core Dockerfile instructions: `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`
- Build an image with `docker build`
- Run and test a container from your custom image

---

## Prerequisites

- Completed [Lab 3: Working with Images](lab-03-images.md)
- Google Cloud Shell open: [https://ssh.cloud.google.com](https://ssh.cloud.google.com)

> 💡 You can use the built-in Cloud Shell Editor (click the pencil icon) as your code editor.

---

## Concepts First

A **Dockerfile** is a plain text file that contains instructions for building a Docker image.
Think of it as a **step-by-step recipe** Docker follows to assemble your application into an image.

```bash
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
(recipe)        (cook)          (dish)      (serve)        (on table)
```

---

## Steps

### Step 1 — Create a project folder

```bash
mkdir my-app
cd my-app
```

---

### Step 2 — Create a simple Python web app

Create a file named `app.py`:

```python
# app.py
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Hello from my Docker container!")

    def log_message(self, format, *args):
        pass  # suppress access logs for clarity

if __name__ == "__main__":
    print("Server running on port 8080...")
    HTTPServer(("", 8080), Handler).serve_forever()
```

---

### Step 3 — Write the Dockerfile

Create a file named `Dockerfile` (no extension) in the same folder:

```dockerfile
# Start from an official Python image (slim = smaller size)
FROM python:3.12-slim

# Set the working directory inside the container
WORKDIR /app

# Copy our app file into the container
COPY app.py .

# Expose port 8080 so Docker knows this container listens there
EXPOSE 8080

# The command to run when the container starts
CMD ["python", "app.py"]
```

---

### Step 4 — Understand each instruction

| Instruction | What it does |
| ----------- | ------------ |
| `FROM python:3.12-slim` | Use the official Python 3.12 slim image as the base |
| `WORKDIR /app` | Set `/app` as the working directory (created automatically if missing) |
| `COPY app.py .` | Copy `app.py` from your machine into `/app` inside the image |
| `EXPOSE 8080` | Document that the app listens on port 8080 (informational) |
| `CMD ["python", "app.py"]` | Default command to run when the container starts |

---

### Step 5 — Verify your project structure

```bash
my-app/
├── app.py
└── Dockerfile
```

---

### Step 6 — Build the image

Make sure you are inside the `my-app` folder, then run:

```bash
docker build -t my-app:v1 .
```

| Part | Meaning |
| ---- | ------- |
| `-t my-app:v1` | Tag the image with the name `my-app` and version `v1` |
| `.` | Build context = current folder (Docker reads the `Dockerfile` here) |

**Expected output:**

```bash
[+] Building 12.3s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load metadata for docker.io/library/python:3.12-slim
 => [1/3] FROM docker.io/library/python:3.12-slim
 => [2/3] WORKDIR /app
 => [3/3] COPY app.py .
 => exporting to image
 => => naming to docker.io/library/my-app:v1
```

---

### Step 7 — Confirm the image exists

```bash
docker images my-app
```

```bash
REPOSITORY   TAG   IMAGE ID       CREATED          SIZE
my-app       v1    3a7f1e2b9c4d   10 seconds ago   131MB
```

---

### Step 8 — Run your custom container

```bash
docker run -d -p 8080:8080 --name my-running-app my-app:v1
```

| Flag | Meaning |
| ---- | ------- |
| `-d` | Run in background |
| `-p 8080:8080` | Map port 8080 on your machine → port 8080 in the container |
| `--name my-running-app` | Give the container a name |

---

### Step 9 — Test it

Open a new terminal tab and run:

```bash
curl http://localhost:8080
```

```bash
Hello from my Docker container!
```

Or open your browser and go to `http://localhost:8080`.

---

### Step 10 — View the container logs

```bash
docker logs my-running-app
```

```bash
Server running on port 8080...
```

---

### Step 11 — Clean up

```bash
docker stop my-running-app
docker rm my-running-app
```

---

## What Just Happened?

```text
docker build -t my-app:v1 .
                │      │  └── build context (current folder)
                │      └── tag (name:version)
                └── flag for tagging

Each Dockerfile instruction = one image LAYER
Layers are cached — unchanged layers are reused on rebuild (faster!)
```

---

## Key Takeaways

| Command | What it does |
| ------- | ------------ |
| `docker build -t <name>:<tag> .` | Build an image from a Dockerfile in the current folder |
| `docker run -p <host>:<container>` | Map a port from your machine to the container |
| `docker logs <name>` | View stdout/stderr output from a container |
| `COPY` | Copies files from your machine into the image |
| `CMD` | Defines the default start command (can be overridden at `docker run`) |

---

## Quick Reference

```bash
# Build
docker build -t my-app:v1 .

# Run with port mapping
docker run -d -p 8080:8080 --name my-running-app my-app:v1

# Test
curl http://localhost:8080

# Logs
docker logs my-running-app

# Clean up
docker stop my-running-app && docker rm my-running-app
```

---

## Navigation

- Previous: [Lab 3: Working with Images](lab-03-images.md)
- [Home](README.md)
- Next: [Lab 5: Docker Compose & Environment Variables](lab-05-docker-compose.md)
