# Lab 3: Working with Images

> **Estimated time:** 15 minutes
> **Difficulty:** ⭐⭐ Beginner–Intermediate

---

## Objectives

By the end of this lab, you will be able to:

- List images stored on your local machine with `docker images`
- Pull an image from Docker Hub without running it
- Inspect image details (size, tag, ID)
- Remove images you no longer need with `docker rmi`
- Understand what image tags are and how to use them

---

## Prerequisites

- Completed [Lab 2: Managing Containers](lab-02-container-management.md)
- Google Cloud Shell open: [https://ssh.cloud.google.com](https://ssh.cloud.google.com)

---

## Concepts First

Think of a Docker **image** as a **recipe** (or a template).
A **container** is a **dish** cooked from that recipe.

| Term | Analogy | Docker meaning |
| ---- | ------- | -------------- |
| Image | Recipe card | Read-only template that defines what a container looks like |
| Container | Cooked dish | A running (or stopped) instance created from an image |
| Tag | Edition of the recipe (e.g., "v2.0") | Version label on an image (default: `latest`) |
| Docker Hub | Public cookbook library | Free online registry for sharing images |

---

## Steps

### Step 1 — List your local images

```bash
docker images
```

You should see images downloaded in previous labs:

```bash
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    a6bd71f48f68   2 weeks ago    187MB
hello-world   latest    d2c94e258dcb   11 months ago  13.3kB
```

| Column | Meaning |
| ------ | ------- |
| `REPOSITORY` | Image name |
| `TAG` | Version label (`latest` means the newest published version) |
| `IMAGE ID` | Unique short hash of the image |
| `SIZE` | How much disk space it takes |

---

### Step 2 — Pull an image without running it

Use `docker pull` to download an image and store it locally — without creating a container yet.

```bash
docker pull alpine
```

```bash
latest: Pulling from library/alpine
bca4290a9639: Pull complete
Digest: sha256:...
Status: Downloaded newer image for alpine:latest
docker.io/library/alpine:latest
```

Run `docker images` again — `alpine` is now in the list.

---

### Step 3 — Pull a specific tag (version)

Tags let you pin to a specific version instead of always getting `latest`.

```bash
docker pull node:20-alpine
```

```bash
20-alpine: Pulling from library/node
...
Status: Downloaded newer image for node:20-alpine
```

Run `docker images` — you'll see `node` with tag `20-alpine`.

---

### Step 4 — Inspect an image

```bash
docker inspect alpine
```

This outputs detailed JSON: OS, layers, environment variables, and more.
To extract just the OS info:

```bash
docker inspect alpine --format '{{.Os}}'
```

```bash
linux
```

---

### Step 5 — Remove an image

First, make sure no container is using it. Then:

```bash
docker rmi alpine
```

```bash
Untagged: alpine:latest
Untagged: alpine@sha256:...
Deleted: sha256:bca4290a9639...
```

> 💡 **If you see an error** like `image is being used by stopped container`, remove the container first:

```bash
docker ps -a          # find the container using alpine
docker rm <name>      # remove it
docker rmi alpine     # now remove the image
```

---

### Step 6 — Search for images on Docker Hub (Bonus)

```bash
docker search python
```

```bash
NAME                    DESCRIPTION                                     STARS
python                  Python is an interpreted, interactive...         9800
...
```

> **Tip:** Always prefer **Official Images** (marked with `[OK]` in the OFFICIAL column) — they are maintained by Docker and the software authors.

---

### Step 7 — (Optional) Push an image to Docker Hub

To push an image you must:

1. Have a free Docker Hub account at [hub.docker.com](https://hub.docker.com)
2. Log in from the terminal:

   ```bash
   docker login
   ```

3. Tag your image with your username:

   ```bash
   docker tag hello-world YOUR_DOCKERHUB_USERNAME/hello-world:v1
   ```

4. Push it:

   ```bash
   docker push YOUR_DOCKERHUB_USERNAME/hello-world:v1
   ```

> This step is **optional** in this lab. We will practice build + push together in Lab 4.

---

## What Just Happened?

```bash
docker pull node:20-alpine
              │    └── tag (version)
              └── image name (repository)
```

- `docker pull` = download image layers from the registry to your local cache
- Each image is made of **layers** — Docker reuses layers shared between images to save disk space
- `docker rmi` removes from your local cache only — the image stays on Docker Hub

---

## Key Takeaways

| Command | What it does |
| ------- | ------------ |
| `docker images` | List all locally stored images |
| `docker pull <image>:<tag>` | Download an image without running it |
| `docker inspect <image>` | Show detailed image metadata |
| `docker rmi <image>` | Remove a local image |
| `docker search <term>` | Search Docker Hub from the terminal |

---

## Quick Reference

```bash
docker images                       # list local images
docker pull alpine                  # pull latest alpine
docker pull node:20-alpine          # pull specific tag
docker inspect alpine               # detailed image info
docker rmi alpine                   # remove image
docker search python                # search Docker Hub
```

---

## Navigation

- Previous: [Lab 2: Managing Containers](lab-02-container-management.md)
- [Home](README.md)
- Next: [Lab 4: Build Your Own Image](lab-04-dockerfile.md)
