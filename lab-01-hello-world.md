# Lab 1: Your First Container

> **Estimated time:** 10 minutes
> **Difficulty:** ⭐ Beginner

---

## Objectives

By the end of this lab, you will be able to:

- Run your first Docker container using `docker run`
- Understand what happens when Docker cannot find an image locally
- Recognize Docker Hub as the default public image registry

---

## Prerequisites

- A Google account
- Open Google Cloud Shell: [https://ssh.cloud.google.com](https://ssh.cloud.google.com)

> 💡 Docker is pre-installed in Cloud Shell — no local installation needed.

---

## Steps

### Step 1 — Run the hello-world container

Open your terminal and type the following command, then press **Enter**:

```bash
docker run hello-world
```

---

## Expected Output

```text
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:...
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

---

## What Just Happened?

| # | Step | What Docker did |
| --- | ---- | --------------- |
| 1 | **Local check** | Looked for the `hello-world` image on your local machine → **Not found** |
| 2 | **Pull** | Automatically downloaded the image from **Docker Hub** (public cloud registry) |
| 3 | **Create & Run** | Created a container from the image and started it |
| 4 | **Exit** | The container printed the message, then **exited automatically** |

> **Key concept:** A container only lives as long as its main process runs.
> `hello-world` prints one message and exits — so the container exits too.

---

## Step 2 — Run it again

Run the exact same command a second time:

```bash
docker run hello-world
```

**What's different this time?**

There is **no "Pulling from…" message** — Docker finds the image already on your machine and reuses it instantly.

---

## Key Takeaways

| Concept | Explanation |
| ------- | ----------- |
| `docker run <image>` | Pull (if needed) + Create + Start a container — all in one command |
| Docker Hub | The default public registry where Docker looks for images |
| Auto-exit | Containers stop when their main process finishes |
| Image caching | Downloaded images are stored locally; no re-download needed next time |

---

## Quick Reference

```bash
# Run a container
docker run hello-world
```

---

## Navigation

- [Home](README.md)
- Next: [Lab 2: Managing Containers](lab-02-container-management.md)
