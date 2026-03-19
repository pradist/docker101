# Lab 2: Managing Containers

> **Estimated time:** 15 minutes
> **Difficulty:** ⭐ Beginner

---

## Objectives

By the end of this lab, you will be able to:

- List running and stopped containers with `docker ps`
- Run a container in the background (detached mode)
- Expose a container port to your host machine
- View a running web server in your browser
- Serve your own custom HTML file from a container
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
| **Port mapping (`-p`)** | Connects a port on your host to a port inside the container, e.g. `-p 8080:80` means host:8080 → container:80 |
| **Volume mount (`-v`)** | Shares a file or folder from your host into the container, e.g. `-v /host/path:/container/path` |

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

### Step 3 — Run an nginx web server in the background with port mapping

```bash
docker run -d --name my-web -p 8080:80 nginx
```

| Flag | Meaning |
| ---- | ------- |
| `-d` | Detached — run in background |
| `--name my-web` | Give the container a human-friendly name |
| `-p 8080:80` | Map port 8080 on your host to port 80 inside the container |
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

### Step 4 — Confirm it is running and the port is mapped

```bash
docker ps
```

```bash
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS                  NAMES
7f3b2c1a9e4d   nginx   "/docker-entrypoint.…"   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   my-web
```

Notice the **PORTS** column now shows `0.0.0.0:8080->80/tcp` — traffic arriving at port 8080 on your host is forwarded to port 80 inside the container.

---

### Step 5 — Access the nginx welcome page

**Option A — curl (works anywhere):**

```bash
curl http://localhost:8080
```

You should see the HTML of the nginx welcome page:

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**Option B — Browser (Google Cloud Shell):**

Click the **Web Preview** button (top-right of Cloud Shell, looks like a box with an arrow) → **Preview on port 8080**.

You should see the **"Welcome to nginx!"** page in a new browser tab.

---

### Step 6 — Serve your own HTML page

Right now nginx is showing its default welcome page. Let's replace it with your own.

**1. Create a custom `index.html`** — replace `Your Name` with your actual name:

```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>My Docker Page</title></head>
<body>
  <h1>Hello from Docker!</h1>
  <p>This page is served by <strong>Your Name</strong> 🐳</p>
</body>
</html>
EOF
```

**2. Remove the existing container** (we need to start a new one with a volume mount):

```bash
docker rm -f my-web
```

**3. Run nginx again, this time mounting your HTML file:**

```bash
docker run -d --name my-web -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html \
  nginx
```

| Flag | Meaning |
| ---- | ------- |
| `-v $(pwd)/index.html:/usr/share/nginx/html/index.html` | Mount your local `index.html` into the path nginx serves as its root page |

**4. Verify your page:**

```bash
curl http://localhost:8080
```

You should see your custom HTML with your name in the response. Open the **Web Preview** in Cloud Shell to see it rendered in the browser.

---

### Step 7 — Stop the container

```bash
docker stop my-web
```

```bash
my-web
```

Run `docker ps` again — the container disappears from the list. Run `docker ps -a` — it reappears with status **Exited**.

Try `curl http://localhost:8080` now — you should get a **connection refused** error, confirming the port is no longer accessible.

---

### Step 8 — Start it again

```bash
docker start my-web
```

```bash
my-web
```

Run `docker ps` to confirm it is **Up** again. Port `8080` is accessible once more.

---

### Step 9 — Remove a container

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
docker run -d  --name my-web  -p 8080:80  nginx
           │          │           │         └── image name
           │          │           └── host port 8080 → container port 80
           │          └── human name for the container
           └── detached (run in background)
```

- `docker run` **creates a brand-new container** from an image and starts it — every `docker run` gives you a fresh container
- `docker stop` sends a **SIGTERM** signal → container shuts down gracefully
- `docker start` brings a **stopped** container back (same container, same data)
- `docker rm` **permanently deletes** the container record (not the image)

---

## Key Takeaways

| Command | What it does |
| ------- | ------------ |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (running + stopped) |
| `docker run -d --name <name> -p <host>:<container> <image>` | Run a container in background with a custom name and port mapping |
| `docker run -d ... -v <host-path>:<container-path> <image>` | Mount a host file or folder into the container |
| `docker stop <name>` | Gracefully stop a container |
| `docker start <name>` | Start a stopped container |
| `docker rm <name>` | Remove a stopped container |

---

## Quick Reference

```bash
docker ps                                  # running containers
docker ps -a                               # all containers
docker run -d --name my-web -p 8080:80 nginx
curl http://localhost:8080                 # test the web server

# serve your own HTML
docker rm -f my-web
docker run -d --name my-web -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html nginx

docker stop my-web
docker start my-web
docker rm my-web
docker run --rm hello-world                # auto-remove on exit
```

## Navigation

- Previous: [Lab 1: Your First Container](lab-01-hello-world.md)
- [Home](README.md)
- Next: [Lab 3: Working with Images](lab-03-images.md)
