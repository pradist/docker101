# Lab 2: Managing Containers

> **Estimated time:** 15 minutes
> **Difficulty:** ⭐ Beginner

---

## Objectives

By the end of this lab, you will be able to:

- List running and stopped containers with `docker ps`
- Run a container in the background (detached mode)
- Stop and restart a container
- Remove containers you no longer need

---

## Prerequisites

- Completed [Lab 1: Your First Container](lab-01-hello-world.md)
- Google Cloud Shell open: [https://ssh.cloud.google.com](https://ssh.cloud.google.com)

---

## Concepts First

| Term | Meaning |
| ---- | ------- |
| **Container ID** | A unique ID Docker assigns to every container |
| **Running** | The container's process is currently active |
| **Stopped / Exited** | The container's process has finished or was stopped |
| **Detached mode (`-d`)** | The container runs in the background, freeing up your terminal |

---

## Steps

### Step 1 — See what's running right now

```bash
docker ps
```

This lists only **running** containers. If nothing is running, you get an empty table:

```bash
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

### Step 2 — See ALL containers (including stopped ones)

```bash
docker ps -a
```

You should see the `hello-world` container from Lab 1 with status **Exited**:

```bash
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     NAMES
a1b2c3d4e5f6   hello-world   "/hello"   5 minutes ago   Exited (0) 5 minutes ago   funny_morse
```

> **Note:** Docker auto-assigns a random name (like `funny_morse`) if you don't specify one.

---

### Step 3 — Run an nginx web server in the background

```bash
docker run -d --name my-web nginx
```

| Flag | Meaning |
| ---- | ------- |
| `-d` | Detached — run in background |
| `--name my-web` | Give the container a human-friendly name |
| `nginx` | The image to use |

**Expected output** — Docker prints just the container ID:

```bash
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:latest
7f3b2c1a9e4d8b5c6a2f1e3d9c8b7a6f5e4d3c2b1a0f9e8d7c6b5a4f3e2d1c0
```

---

### Step 4 — Confirm it is running

```bash
docker ps
```

```bash
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS     NAMES
7f3b2c1a9e4d   nginx   "/docker-entrypoint.…"   10 seconds ago  Up 9 seconds   80/tcp    my-web
```

---

### Step 5 — Stop the container

```bash
docker stop my-web
```

```bash
my-web
```

Run `docker ps` again — the container disappears from the list. Run `docker ps -a` — it reappears with status **Exited**.

---

### Step 6 — Start it again

```bash
docker start my-web
```

```bash
my-web
```

Run `docker ps` to confirm it is **Up** again.

---

### Step 7 — Remove a container

You must stop a container before removing it (or use `-f` to force):

```bash
docker stop my-web
docker rm my-web
```

Confirm it's gone:

```bash
docker ps -a
```

> 💡 **Shortcut:** Use `--rm` to auto-remove a container as soon as it exits — no need to `docker rm` manually.

```bash
docker run --rm hello-world
```

---

## What Just Happened?

```bash
docker run -d --name my-web nginx
        │   │   └── human name for the container
        │   └── detached (run in background)
        └── image name
```

- `docker stop` sends a **SIGTERM** signal → container shuts down gracefully
- `docker start` brings a **stopped** container back (same container, same data)
- `docker rm` **permanently deletes** the container record (not the image)

---

## Key Takeaways

| Command | What it does |
| ------- | ------------ |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (running + stopped) |
| `docker run -d --name <name> <image>` | Run a container in background with a custom name |
| `docker stop <name>` | Gracefully stop a container |
| `docker start <name>` | Start a stopped container |
| `docker rm <name>` | Remove a stopped container |

---

## Quick Reference

```bash
docker ps                        # running containers
docker ps -a                     # all containers
docker run -d --name my-web nginx
docker stop my-web
docker start my-web
docker rm my-web
docker run --rm hello-world      # auto-remove on exit
```

## Navigation

- Previous: [Lab 1: Your First Container](lab-01-hello-world.md)
- [Home](README.md)
- Next: [Lab 3: Working with Images](lab-03-images.md)
