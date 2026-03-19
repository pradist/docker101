# Lab 4: Build Your Own Image

> **Estimated time:** 20 minutes
> **Difficulty:** ⭐⭐ Beginner–Intermediate

---

## Objectives

By the end of this lab, you will be able to:

- Write a `Dockerfile` to define a custom image
- Understand the core Dockerfile instructions: `FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`
- Use a **multi-stage build** to create a small production image
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
mkdir lab4-app
cd lab4-app
```

---

### Step 2 — Create a simple Go web app

Create a file named `main.go`:

```go
// main.go
package main

import (
  "fmt"
  "log"
  "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello from my Docker container!")
}

func main() {
  http.HandleFunc("/", handler)
  log.Println("Server running on port 8080...")
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

### Step 3 — Write the Dockerfile

Create a file named `Dockerfile` (no extension) in the same folder:

```dockerfile
# ── Stage 1: Build ──────────────────────────────────────
# Use the official Go image to compile the binary
FROM golang:1.22-alpine AS builder

WORKDIR /app

# Copy source code into the build container
COPY main.go .

# Compile the Go binary
RUN go build -o server main.go

# ── Stage 2: Run ────────────────────────────────────────
# Use a minimal Alpine image — no Go toolchain needed
FROM alpine:3.19

WORKDIR /app

# Copy only the compiled binary from Stage 1
COPY --from=builder /app/server .

EXPOSE 8080

CMD ["./server"]
```

---

### Step 4 — Understand each instruction

| Instruction | What it does |
| ----------- | ------------ |
| `FROM golang:1.22-alpine AS builder` | Start a **build stage** using the official Go image; name it `builder` |
| `WORKDIR /app` | Set `/app` as the working directory (created automatically if missing) |
| `COPY main.go .` | Copy `main.go` from your machine into the build container |
| `RUN go build -o server main.go` | **Compile** the Go source into a binary named `server` |
| `FROM alpine:3.19` | Start a **new, minimal stage** — the final image (no Go toolchain) |
| `COPY --from=builder /app/server .` | Copy only the compiled binary from the `builder` stage |
| `EXPOSE 8080` | Document that the app listens on port 8080 (informational) |
| `CMD ["./server"]` | Default command to run when the container starts |

> 💡 **Multi-stage build:** The final image contains only the binary (~15 MB), not the Go compiler (~250 MB). This is a best practice for production images.

---

### Step 5 — Verify your project structure

```text
lab4-app/
├── main.go
└── Dockerfile
```

---

### Step 6 — Build the image

Make sure you are inside the `lab4-app` folder, then run:

```bash
docker build -t lab4-app:v1 .
```

| Part | Meaning |
| ---- | ------- |
| `-t lab4-app:v1` | Tag the image with the name `lab4-app` and version `v1` |
| `.` | Build context = current folder (Docker reads the `Dockerfile` here) |

**Expected output:**

```text
[+] Building 28.5s (9/9) FINISHED
 => [internal] load build definition from Dockerfile
 => [builder 1/3] FROM docker.io/library/golang:1.22-alpine
 => [builder 2/3] WORKDIR /app
 => [builder 3/3] COPY main.go .
 => [builder 4/3] RUN go build -o server main.go
 => [stage-2 1/2] FROM docker.io/library/alpine:3.19
 => [stage-2 2/2] COPY --from=builder /app/server .
 => exporting to image
 => => naming to docker.io/library/lab4-app:v1
```

---

### Step 7 — Confirm the image exists

```bash
docker images lab4-app
```

```text
REPOSITORY   TAG   IMAGE ID       CREATED          SIZE
lab4-app       v1    3a7f1e2b9c4d   10 seconds ago   15.6MB
```

> Notice how small the image is — only **~15 MB**! The Go compiler (~250 MB) was used only in the build stage and is not included in the final image.

---

### Step 8 — Run your custom container

```bash
docker run -d -p 8080:8080 --name lab4-running-app lab4-app:v1
```

| Flag | Meaning |
| ---- | ------- |
| `-d` | Run in background |
| `-p 8080:8080` | Map port 8080 on your machine → port 8080 in the container |
| `--name lab4-running-app` | Give the container a name |

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
docker logs lab4-running-app
```

```text
2026/03/19 08:00:00 Server running on port 8080...
```

---

### Step 11 — Clean up

```bash
docker stop lab4-running-app
docker rm lab4-running-app
```

---

## What Just Happened?

```text
docker build -t lab4-app:v1 .
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
| `RUN` | Execute a command during the image build (e.g., compile, install) |
| `COPY --from=<stage>` | Copy files from a previous build stage (multi-stage) |
| `CMD` | Defines the default start command (can be overridden at `docker run`) |

---

## Quick Reference

```bash
# Build
docker build -t lab4-app:v1 .

# Run with port mapping
docker run -d -p 8080:8080 --name lab4-running-app lab4-app:v1

# Test
curl http://localhost:8080

# Logs
docker logs lab4-running-app

# Clean up
docker stop lab4-running-app && docker rm lab4-running-app
```

---

## Navigation

- Previous: [Lab 3: Working with Images](lab-03-images.md)
- [Home](README.md)
- Next: [Lab 5: Docker Compose & Environment Variables](lab-05-docker-compose.md)
