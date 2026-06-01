# Docker

---

## Table of Contents
1. [Basic Concepts (Q1–Q20)](#1-basic-concepts)
2. [Dockerfile (Q21–Q35)](#2-dockerfile)
3. [Docker Commands (Q36–Q50)](#3-docker-commands)
4. [Docker Networking (Q51–Q62)](#4-docker-networking)
5. [Docker Volumes & Storage (Q63–Q72)](#5-docker-volumes--storage)
6. [Docker Compose (Q73–Q85)](#6-docker-compose)
7. [Security & Best Practices (Q86–Q98)](#7-security--best-practices)
8. [Advanced & Real-World Scenarios (Q99–Q110)](#8-advanced--real-world-scenarios)

---

## 1. Basic Concepts

**Q1. What is Docker?**
> Docker is a platform that packages your application and all its dependencies (code, runtime, libraries, config) into a **container**. This container runs the same way everywhere — on your laptop, staging server, or production — no more "it works on my machine" problems.

---

**Q2. What is a Container?**
> A container is a lightweight, isolated process running on a host machine. It shares the host OS kernel but has its own filesystem, networking, and process space.
> Think of it like: **a container is to a VM what an apartment is to a house** — smaller, faster, shares common infrastructure.

---

**Q3. What is the difference between a Container and a Virtual Machine?**
> | Feature | Container | Virtual Machine |
> |---|---|---|
> | OS | Shares host OS kernel | Has its own full OS |
> | Size | Megabytes | Gigabytes |
> | Startup time | Seconds | Minutes |
> | Performance | Near-native | Slower (hypervisor overhead) |
> | Isolation | Process-level | Hardware-level |
> | Use case | Microservices, CI/CD | Full OS isolation |

---

**Q4. What is a Docker Image?**
> A Docker Image is a read-only template used to create containers. It contains:
> - Application code
> - Runtime (Java, Python, Node.js)
> - Libraries and dependencies
> - Configuration files
> Think of an image as a **recipe** — a container is the dish cooked from that recipe.

---

**Q5. What is a Docker Container?**
> A container is a **running instance of a Docker image**. You can run many containers from the same image. Each container is isolated and has its own writable layer on top of the image.

---

**Q6. What is Docker Hub?**
> Docker Hub is a public registry where Docker images are stored and shared. It has:
> - Official images: `nginx`, `mysql`, `node`, `python`, `ubuntu`
> - Community images: Anyone can publish
> - Private repositories: Store your own private images
> Other registries: AWS ECR, Google GCR, GitHub Container Registry, Harbor (self-hosted)

---

**Q7. What is a Docker Registry?**
> A Docker Registry is a storage and distribution system for Docker images.
> - **Public:** Docker Hub (`hub.docker.com`)
> - **Private:** AWS ECR, Harbor, JFrog Artifactory
> - **Self-hosted:** Your own registry using `registry:2` image

---

**Q8. What is the Docker Architecture?**
> Docker uses a **Client-Server** architecture:
> - **Docker Client:** The `docker` CLI you type commands into
> - **Docker Daemon (dockerd):** Background service that builds, runs, manages containers
> - **Docker Registry:** Stores images
>
> Flow: `docker run nginx` → Client sends request → Daemon pulls image from registry → Daemon creates and starts container

---

**Q9. What is a Docker Layer?**
> Docker images are built in layers. Each instruction in a Dockerfile creates a new layer.
> - Layers are **cached** — if nothing changed, Docker reuses cached layers
> - Layers are **shared** — if two images use the same base layer, it's stored only once
> - This makes builds faster and saves disk space

---

**Q10. What is the difference between `docker run` and `docker start`?**
> - `docker run` — Creates a NEW container from an image and starts it
> - `docker start` — Starts an already EXISTING (stopped) container
> - `docker run = docker create + docker start`

---

**Q11. What is Docker Desktop?**
> Docker Desktop is a GUI application for Mac and Windows that includes Docker Engine, Docker CLI, Docker Compose, and Kubernetes. Makes it easy to run Docker locally without Linux.

---

**Q12. What is containerd?**
> containerd is the container runtime that Docker uses under the hood. It handles:
> - Pulling images
> - Managing container lifecycle
> - Storage and networking
> Kubernetes also uses containerd directly (without Docker) in modern setups.

---

**Q13. What is the difference between Docker CE and Docker EE?**
> - **Docker CE (Community Edition):** Free, open-source. For developers and small teams.
> - **Docker EE (Enterprise Edition):** Paid, with enterprise features — security scanning, RBAC, support. Now called **Docker Business**.

---

**Q14. What happens when you run `docker run hello-world`?**
> 1. Docker client contacts Docker daemon
> 2. Daemon checks if `hello-world` image exists locally
> 3. If not, pulls image from Docker Hub
> 4. Daemon creates a container from the image
> 5. Container runs, prints message, exits
> 6. Daemon streams output to client (your terminal)

---

**Q15. What is the difference between Image and Container?**
> - **Image:** Static, read-only template (like a class in OOP)
> - **Container:** Running instance of an image (like an object in OOP)
> One image → Many containers (just like one class → many objects)

---

**Q16. What is Docker Swarm?**
> Docker Swarm is Docker's native container orchestration tool. It lets you manage a cluster of Docker hosts as one. However, most teams now use **Kubernetes** instead of Swarm — Kubernetes is more powerful and widely adopted.

---

**Q17. What is the difference between Docker Swarm and Kubernetes?**
> | Feature | Docker Swarm | Kubernetes |
> |---|---|---|
> | Complexity | Simple | Complex |
> | Scaling | Basic | Advanced |
> | Community | Smaller | Huge |
> | Features | Limited | Rich |
> | Production use | Declining | Industry standard |

---

**Q18. What is a Base Image?**
> A base image is the starting point for your Dockerfile. Common base images:
> - `ubuntu:22.04` — Full Ubuntu OS (large ~70MB)
> - `alpine:3.18` — Minimal Linux (tiny ~5MB) — preferred for small images
> - `python:3.11-slim` — Python with slim Debian
> - `node:18-alpine` — Node.js on Alpine
> - `scratch` — Empty image (for static binaries)

---

**Q19. What is Alpine Linux and why is it used in Docker?**
> Alpine Linux is an extremely small Linux distribution (~5MB). It's popular for Docker because:
> - Smaller image size = faster pulls, less storage
> - Smaller attack surface = more secure
> - But: uses `musl libc` instead of `glibc` — some apps may have compatibility issues

---

**Q20. What is the difference between `ENTRYPOINT` and `CMD`?**
> | | ENTRYPOINT | CMD |
> |---|---|---|
> | Purpose | Main command (fixed) | Default arguments |
> | Override | Needs `--entrypoint` flag | Overridden by `docker run` arguments |
> | Combine | ENTRYPOINT + CMD = command + default args | |
>
> ```dockerfile
> ENTRYPOINT ["python", "app.py"]   # always runs python app.py
> CMD ["--port", "8080"]            # default arg, can be overridden
> ```
> `docker run myapp --port 9090` → runs `python app.py --port 9090`

---

## 2. Dockerfile

**Q21. What is a Dockerfile?**
> A Dockerfile is a text file with instructions to build a Docker image. Each instruction creates a layer. Docker reads it top to bottom.

---

**Q22. Write a simple Dockerfile for a Node.js app.**
```dockerfile
# Use official Node.js base image
FROM node:18-alpine

# Set working directory inside container
WORKDIR /app

# Copy package files first (for layer caching)
COPY package*.json ./

# Install dependencies
RUN npm install --production

# Copy rest of application code
COPY . .

# Expose the port app listens on
EXPOSE 3000

# Command to run the app
CMD ["node", "server.js"]
```

---

**Q23. Write a Dockerfile for a Python Flask app.**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Copy requirements first for better caching
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

ENV FLASK_APP=app.py
ENV FLASK_ENV=production

CMD ["flask", "run", "--host=0.0.0.0"]
```

---

**Q24. What is a Multi-Stage Build in Docker?**
> Multi-stage builds use multiple `FROM` statements in one Dockerfile. The final image only contains what you need — not build tools.
```dockerfile
# Stage 1: Build
FROM maven:3.9-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run (small final image)
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=builder /app/target/myapp.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```
> Result: Final image is ~200MB instead of ~500MB (no Maven, no source code).

---

**Q25. Explain all common Dockerfile instructions.**
> | Instruction | Purpose | Example |
> |---|---|---|
> | `FROM` | Base image | `FROM ubuntu:22.04` |
> | `WORKDIR` | Set working directory | `WORKDIR /app` |
> | `COPY` | Copy files from host to image | `COPY . .` |
> | `ADD` | Like COPY but supports URLs and tar extraction | `ADD app.tar.gz /app` |
> | `RUN` | Execute command during build | `RUN apt-get update` |
> | `CMD` | Default command when container starts | `CMD ["node", "app.js"]` |
> | `ENTRYPOINT` | Fixed command that always runs | `ENTRYPOINT ["nginx"]` |
> | `EXPOSE` | Document which port app uses | `EXPOSE 8080` |
> | `ENV` | Set environment variables | `ENV NODE_ENV=production` |
> | `ARG` | Build-time variables | `ARG VERSION=1.0` |
> | `VOLUME` | Create mount point | `VOLUME /data` |
> | `USER` | Set user to run as | `USER appuser` |
> | `LABEL` | Add metadata | `LABEL version="1.0"` |
> | `HEALTHCHECK` | Container health check | `HEALTHCHECK CMD curl -f http://localhost/` |

---

**Q26. What is the difference between `COPY` and `ADD`?**
> - **COPY:** Simply copies files from host to image. Preferred — explicit and predictable.
> - **ADD:** Does everything COPY does PLUS:
>   - Can download files from URLs
>   - Automatically extracts tar files
> Best practice: Use `COPY` unless you specifically need `ADD`'s extra features.

---

**Q27. What is the difference between `RUN`, `CMD`, and `ENTRYPOINT`?**
> - **RUN:** Executes during **image build** time. Creates a new layer. Used to install software.
> - **CMD:** Executes when **container starts**. Can be overridden. Default command.
> - **ENTRYPOINT:** Executes when **container starts**. Cannot be easily overridden. Fixed command.

---

**Q28. What is `.dockerignore` file?**
> Like `.gitignore` — tells Docker which files to EXCLUDE when building an image. Speeds up builds and keeps images small.
```
# .dockerignore
node_modules/
.git/
*.log
.env
__pycache__/
*.pyc
tests/
README.md
Dockerfile
.dockerignore
```

---

**Q29. How do you reduce Docker image size?**
> 1. Use Alpine or slim base images
> 2. Use multi-stage builds
> 3. Combine RUN commands to reduce layers:
>    ```dockerfile
>    # Bad (3 layers)
>    RUN apt-get update
>    RUN apt-get install -y curl
>    RUN rm -rf /var/lib/apt/lists/*
>
>    # Good (1 layer)
>    RUN apt-get update && apt-get install -y curl \
>        && rm -rf /var/lib/apt/lists/*
>    ```
> 4. Use `.dockerignore`
> 5. Don't install unnecessary packages
> 6. Use `--no-cache` flag for package managers

---

**Q30. What is Docker layer caching and how to use it effectively?**
> Docker caches each layer. If a layer hasn't changed, it reuses the cache (faster builds).
> **Best practice:** Put things that change LESS OFTEN at the top, things that change MORE OFTEN at the bottom.
```dockerfile
FROM node:18-alpine        # rarely changes → cached
WORKDIR /app
COPY package*.json ./      # changes when dependencies change
RUN npm install            # expensive step — cached unless package.json changes
COPY . .                   # changes every code change — always re-runs
CMD ["node", "app.js"]
```

---

**Q31. What is `ARG` vs `ENV` in Dockerfile?**
> - **ARG:** Available only during **build time**. Not in the running container.
>   ```dockerfile
>   ARG APP_VERSION=1.0
>   RUN echo "Building version $APP_VERSION"
>   ```
>   ```bash
>   docker build --build-arg APP_VERSION=2.0 .
>   ```
> - **ENV:** Available during **build time AND runtime** (inside the container).
>   ```dockerfile
>   ENV NODE_ENV=production
>   ```

---

**Q32. What is HEALTHCHECK in Dockerfile?**
```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```
> Docker periodically runs this command to check if the container is healthy. Status: `healthy`, `unhealthy`, or `starting`. Used by Docker Swarm and Kubernetes to manage containers.

---

**Q33. What is the USER instruction in Dockerfile?**
> By default containers run as `root` — a security risk. `USER` switches to a non-root user:
```dockerfile
# Create a user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Switch to that user
USER appuser

CMD ["node", "app.js"]
```

---

**Q34. What is `WORKDIR` and why use it?**
> `WORKDIR` sets the working directory for all subsequent instructions. It's better than `RUN cd /app` because:
> - Creates the directory if it doesn't exist
> - Persists across RUN, CMD, ENTRYPOINT, COPY, ADD instructions
> - Makes Dockerfile more readable

---

**Q35. How do you pass secrets to a Docker build safely?**
> Use Docker BuildKit secrets (doesn't store secret in image layers):
```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret
```
```bash
docker build --secret id=mysecret,src=./secret.txt .
```
> **Never** do: `COPY secret.txt .` or `ARG SECRET_KEY` — these get baked into image layers!

---

## 3. Docker Commands

**Q36. What are the most important Docker commands?**
```bash
# Images
docker images                          # list local images
docker pull nginx:latest               # pull image from registry
docker build -t myapp:1.0 .            # build image from Dockerfile
docker push myrepo/myapp:1.0           # push image to registry
docker rmi myapp:1.0                   # remove image
docker image prune                     # remove unused images

# Containers
docker run -d -p 8080:80 nginx         # run container (detached, port mapping)
docker ps                              # list running containers
docker ps -a                           # list all containers (including stopped)
docker stop container_id               # gracefully stop container
docker kill container_id               # force stop container
docker rm container_id                 # remove stopped container
docker logs container_id               # view container logs
docker logs -f container_id            # follow logs (like tail -f)
docker exec -it container_id bash      # open shell inside container
docker inspect container_id            # detailed container info (JSON)
docker stats                           # live resource usage
docker top container_id                # processes inside container
```

---

**Q37. Explain `docker run` flags.**
```bash
docker run \
  -d \                          # detached (background)
  -p 8080:80 \                  # port mapping (host:container)
  -v /host/data:/container/data \  # volume mount
  -e DB_HOST=localhost \         # environment variable
  --name mycontainer \          # give container a name
  --network mynetwork \         # connect to network
  --restart always \            # restart policy
  --memory 512m \               # memory limit
  --cpus 1.0 \                  # CPU limit
  nginx:latest
```

---

**Q38. What is the difference between `docker stop` and `docker kill`?**
> - `docker stop` — Sends SIGTERM signal. Container gracefully shuts down (saves state, closes connections). Waits 10 seconds, then sends SIGKILL.
> - `docker kill` — Immediately sends SIGKILL. Container is force-terminated instantly.
> Always prefer `docker stop` for graceful shutdown.

---

**Q39. How do you view logs of a container?**
```bash
docker logs mycontainer              # all logs
docker logs -f mycontainer          # follow (stream) logs
docker logs --tail 100 mycontainer  # last 100 lines
docker logs --since 1h mycontainer  # logs from last 1 hour
docker logs -t mycontainer          # with timestamps
```

---

**Q40. How do you execute a command inside a running container?**
```bash
docker exec -it mycontainer bash         # open interactive bash shell
docker exec -it mycontainer sh           # if bash not available (Alpine)
docker exec mycontainer ls /app          # run single command
docker exec -e VAR=value mycontainer cmd # with env variable
```
> `-i` = interactive, `-t` = allocate TTY (pseudo-terminal)

---

**Q41. How do you copy files between host and container?**
```bash
# Host to container
docker cp ./myfile.txt mycontainer:/app/myfile.txt

# Container to host
docker cp mycontainer:/app/logs/app.log ./app.log
```

---

**Q42. What is `docker inspect`?**
```bash
docker inspect mycontainer
```
> Returns detailed JSON about a container/image: IP address, mounts, environment variables, network settings, labels, etc. Useful for debugging.
```bash
# Get just the IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mycontainer
```

---

**Q43. How do you clean up Docker resources?**
```bash
docker system prune              # remove stopped containers, unused networks, dangling images
docker system prune -a           # also remove unused images (not just dangling)
docker container prune           # remove all stopped containers
docker image prune               # remove dangling images
docker volume prune              # remove unused volumes
docker network prune             # remove unused networks

# Check disk usage
docker system df
```

---

**Q44. What are dangling images?**
> Dangling images are images with no tag and not referenced by any container — shown as `<none>:<none>`. They're created when you rebuild an image with the same tag. They waste disk space.
```bash
docker images -f dangling=true    # list dangling images
docker image prune                # remove them
```

---

**Q45. How do you tag a Docker image?**
```bash
docker tag myapp:latest myrepo/myapp:1.0.0
docker tag myapp:latest myrepo/myapp:latest
# Now push both tags
docker push myrepo/myapp:1.0.0
docker push myrepo/myapp:latest
```

---

**Q46. How do you save and load Docker images?**
```bash
# Save image to tar file (for offline transfer)
docker save myapp:latest -o myapp.tar

# Load image from tar file
docker load -i myapp.tar

# Export container filesystem (running container)
docker export mycontainer -o mycontainer.tar

# Import as image
docker import mycontainer.tar myapp:imported
```

---

**Q47. What is `docker commit`?**
```bash
docker commit mycontainer myapp:modified
```
> Creates a new image from a running/stopped container's current state. Not recommended for production — use Dockerfile instead. Useful for debugging (save a container state for investigation).

---

**Q48. What is `docker diff`?**
```bash
docker diff mycontainer
```
> Shows files changed in a container compared to its image:
> - `A` = Added
> - `C` = Changed
> - `D` = Deleted

---

**Q49. What are Docker restart policies?**
```bash
docker run --restart always nginx       # always restart (even after Docker restart)
docker run --restart unless-stopped nginx  # restart unless manually stopped
docker run --restart on-failure nginx   # restart only if exit code != 0
docker run --restart on-failure:3 nginx # restart on failure, max 3 times
docker run --restart no nginx           # never restart (default)
```

---

**Q50. How do you limit container resources?**
```bash
docker run \
  --memory 512m \          # max 512MB RAM
  --memory-swap 1g \       # total memory+swap limit
  --cpus 1.5 \             # max 1.5 CPU cores
  --cpu-shares 512 \       # relative CPU weight (default 1024)
  nginx
```

---

## 4. Docker Networking

**Q51. What are Docker network types?**
> - **bridge (default):** Containers on same host communicate via a virtual bridge. Each container gets its own IP.
> - **host:** Container shares host's network stack. No isolation. Best performance.
> - **none:** No networking. Completely isolated container.
> - **overlay:** For multi-host networking (Docker Swarm). Containers on different hosts communicate.
> - **macvlan:** Container gets a MAC address and appears as physical device on network.

---

**Q52. What is the default Docker network (bridge)?**
> When you run `docker run nginx`, it joins the default `bridge` network automatically.
> - All containers on bridge network can communicate by IP
> - BUT they cannot communicate by container NAME on the default bridge
> - Create a **custom bridge network** — then containers can reach each other by name (DNS)

---

**Q53. How do containers communicate with each other?**
> **Best way:** Create a custom bridge network
```bash
docker network create mynetwork
docker run -d --name db --network mynetwork postgres
docker run -d --name app --network mynetwork myapp

# Inside app container, connect to db using name 'db'
# DB_HOST=db (not IP address!)
```
> Docker's built-in DNS resolves container names to IPs on custom networks.

---

**Q54. How do you expose a container port to the host?**
```bash
docker run -p 8080:80 nginx       # host port 8080 → container port 80
docker run -p 443:443 nginx       # map same port
docker run -P nginx               # map all EXPOSED ports to random host ports
docker run -p 127.0.0.1:8080:80 nginx  # bind only to localhost (more secure)
```

---

**Q55. What is the difference between `-p` and `--expose`?**
> - `-p 8080:80` — Publishes port to HOST. Accessible from outside.
> - `--expose 80` / `EXPOSE 80` in Dockerfile — Only documents the port. Doesn't publish to host. Only accessible to other containers in same network.

---

**Q56. How do you connect a container to multiple networks?**
```bash
docker network create frontend
docker network create backend
docker run -d --name app --network frontend myapp
docker network connect backend app  # connect to second network
```
> Now `app` is in both networks — can talk to frontend services AND backend services.

---

**Q57. What is Docker DNS?**
> Docker has a built-in DNS server for custom networks. Containers can resolve each other by **container name** or **service name** (in Docker Compose).
> - Default bridge network: NO DNS (use IPs)
> - Custom bridge networks: DNS enabled (use names) ✅

---

**Q58. What are Docker network commands?**
```bash
docker network ls                          # list networks
docker network create mynetwork            # create network
docker network inspect mynetwork           # details of network
docker network connect mynetwork container # add container to network
docker network disconnect mynetwork container  # remove container from network
docker network rm mynetwork                # remove network
docker network prune                       # remove unused networks
```

---

**Q59. What is `host` network mode and when to use it?**
```bash
docker run --network host nginx
```
> Container uses host's network directly. No port mapping needed — nginx on port 80 is directly accessible on host port 80.
> Use when: Performance-critical apps where network overhead matters (high-traffic proxies, monitoring agents).
> Don't use when: You need isolation or run multiple containers on same port.

---

**Q60. What is an overlay network?**
> Overlay network spans multiple Docker hosts (used in Docker Swarm). Containers on different machines communicate as if they're on the same local network. Encrypted traffic between hosts.

---

**Q61. How does container-to-internet communication work?**
> Container (private IP) → Docker bridge → NAT (masquerade) → Host → Internet
> Outbound traffic is automatically NAT'd by the host. Containers can reach the internet without any special config (unless you block it).

---

**Q62. What is `--link` flag in Docker? Is it still used?**
> `--link` was the old way to connect containers. It's **deprecated** and should NOT be used. Use custom Docker networks instead — they're better in every way.

---

## 5. Docker Volumes & Storage

**Q63. What are Docker Volumes?**
> Volumes are the preferred way to persist data in Docker. Container filesystem is temporary — when container dies, data is gone. Volumes store data OUTSIDE the container.
> Three types:
> - **Volumes:** Managed by Docker (`/var/lib/docker/volumes/`)
> - **Bind Mounts:** Map a specific host path to container path
> - **tmpfs Mounts:** In-memory only (Linux only)

---

**Q64. What is the difference between Volume and Bind Mount?**
> | Feature | Volume | Bind Mount |
> |---|---|---|
> | Managed by | Docker | You (host filesystem) |
> | Location | `/var/lib/docker/volumes/` | Any path on host |
> | Portability | High | Low (path must exist on host) |
> | Performance | Better on Docker Desktop | Same on Linux |
> | Use case | Databases, persistent data | Dev (live code reload) |
> | Backup | Docker commands | Standard file backup |

---

**Q65. How do you create and use a Docker Volume?**
```bash
# Create a named volume
docker volume create mydata

# Use it when running container
docker run -d \
  -v mydata:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8

# List volumes
docker volume ls

# Inspect volume (find actual path on host)
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune
```

---

**Q66. How do you use a Bind Mount?**
```bash
# Bind mount: map host directory to container
docker run -d \
  -v /home/user/myapp:/app \   # absolute path required
  -p 3000:3000 \
  node:18-alpine \
  node app.js

# Shorter syntax (--mount)
docker run -d \
  --mount type=bind,source=/home/user/myapp,target=/app \
  node:18-alpine
```
> Great for development — changes to host files reflect immediately in container (live reload).

---

**Q67. How do you share a volume between multiple containers?**
```bash
docker volume create shareddata

docker run -d --name writer \
  -v shareddata:/data \
  myapp-writer

docker run -d --name reader \
  -v shareddata:/data:ro \   # :ro = read-only
  myapp-reader
```

---

**Q68. How do you backup a Docker Volume?**
```bash
# Backup: Run a temporary container, tar the volume, save to host
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/mydata-backup.tar.gz -C /data .

# Restore
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/mydata-backup.tar.gz -C /data
```

---

**Q69. What is a tmpfs mount?**
```bash
docker run -d \
  --tmpfs /tmp:rw,size=100m \
  nginx
```
> tmpfs stores data in host memory — never written to disk. Use for:
> - Sensitive data (deleted when container stops)
> - High-speed temporary storage

---

**Q70. Where does Docker store volumes on the host?**
> `/var/lib/docker/volumes/<volume-name>/_data`
> But you should never access this directly — use `docker volume inspect` or mount the volume.

---

**Q71. What is the `:ro` flag in volume mounts?**
```bash
docker run -v myconfig:/config:ro nginx
```
> `:ro` = read-only. Container can read the volume but cannot write to it. Good for:
> - Config files (prevent accidental modification)
> - Sharing secrets (read-only access)

---

**Q72. How do you use volumes in Dockerfile?**
```dockerfile
VOLUME ["/data"]
```
> This creates an anonymous volume at `/data` — data written here persists even without `-v` flag. But anonymous volumes are hard to manage — prefer named volumes with `-v`.

---

## 6. Docker Compose

**Q73. What is Docker Compose?**
> Docker Compose lets you define and run **multi-container** applications using a YAML file (`docker-compose.yml`). Instead of running many `docker run` commands, you define everything in one file and run:
```bash
docker compose up -d    # start everything
docker compose down     # stop everything
```

---

**Q74. Write a basic `docker-compose.yml` for a web app with database.**
```yaml
version: '3.8'

services:
  app:
    build: .                    # build from local Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
      - DB_NAME=myapp
      - DB_USER=postgres
      - DB_PASS=secret
    depends_on:
      db:
        condition: service_healthy
    networks:
      - appnet
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - appnet

volumes:
  pgdata:

networks:
  appnet:
    driver: bridge
```

---

**Q75. What are the main Docker Compose commands?**
```bash
docker compose up             # start services (foreground)
docker compose up -d          # start services (background/detached)
docker compose down           # stop and remove containers
docker compose down -v        # also remove volumes
docker compose ps             # list running services
docker compose logs           # view logs
docker compose logs -f app    # follow logs of 'app' service
docker compose exec app bash  # open shell in running service
docker compose build          # build/rebuild images
docker compose pull           # pull latest images
docker compose restart app    # restart a service
docker compose stop           # stop services (don't remove)
docker compose start          # start stopped services
docker compose scale app=3    # scale service to 3 replicas
```

---

**Q76. What is `depends_on` in Docker Compose?**
```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy  # wait for db to be healthy
      redis:
        condition: service_started  # just wait for redis to start
```
> `depends_on` controls startup ORDER. But `service_started` doesn't mean the DB is ready — use `service_healthy` with a healthcheck for proper dependency management.

---

**Q77. What is an `.env` file in Docker Compose?**
```bash
# .env file
POSTGRES_PASSWORD=mysecretpassword
APP_PORT=3000
NODE_ENV=production
```
```yaml
# docker-compose.yml
services:
  db:
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  app:
    ports:
      - "${APP_PORT}:3000"
```
> Docker Compose automatically reads `.env` file. Never commit `.env` to Git!

---

**Q78. How do you use multiple Compose files (override)?**
```bash
# Base config
docker compose -f docker-compose.yml \
               -f docker-compose.prod.yml up -d

# docker-compose.prod.yml overrides/extends base config
```
```yaml
# docker-compose.prod.yml
services:
  app:
    image: myrepo/myapp:1.0.0    # use specific image instead of build
    replicas: 3
    environment:
      - NODE_ENV=production
```

---

**Q79. What is the difference between `docker-compose` and `docker compose`?**
> - `docker-compose` — Old standalone binary (Python-based). Version 1.
> - `docker compose` — New plugin built into Docker CLI (Go-based). Version 2.
> Use `docker compose` (without hyphen) — it's the current standard.

---

**Q80. How do you scale a service in Docker Compose?**
```bash
docker compose up -d --scale app=3  # run 3 instances of app service
```
> Note: You can't use a fixed `container_name` when scaling (conflicts). Remove `container_name` from compose file to allow scaling.

---

**Q81. What are Docker Compose profiles?**
```yaml
services:
  app:
    image: myapp
  db:
    image: postgres
  pgadmin:
    image: pgadmin4
    profiles: ["tools"]    # only starts with --profile tools
  mailhog:
    image: mailhog
    profiles: ["tools"]
```
```bash
docker compose up -d                    # starts app + db only
docker compose --profile tools up -d   # starts app + db + pgadmin + mailhog
```
> Profiles let you selectively start optional services.

---

**Q82. What is `network_mode` in Docker Compose?**
```yaml
services:
  app:
    network_mode: host      # use host network
  monitor:
    network_mode: "service:app"  # share network with app service
```

---

**Q83. How do you pass secrets in Docker Compose?**
```yaml
services:
  app:
    secrets:
      - db_password
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

**Q84. What is Docker Compose `healthcheck`?**
```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s      # check every 30s
      timeout: 10s       # fail if no response in 10s
      retries: 3         # mark unhealthy after 3 failures
      start_period: 40s  # grace period before health checks start
```

---

**Q85. What is the difference between Docker Compose and Kubernetes?**
> | Feature | Docker Compose | Kubernetes |
> |---|---|---|
> | Best for | Local development, simple apps | Production, large scale |
> | Complexity | Simple | Complex |
> | Scaling | Basic | Advanced auto-scaling |
> | Self-healing | No | Yes |
> | Multi-host | No | Yes |
> | Learning curve | Easy | Steep |
> Use Docker Compose for development, Kubernetes for production.

---

## 7. Security & Best Practices

**Q86. What are Docker security best practices?**
> 1. Never run containers as root — use `USER` instruction
> 2. Use official, trusted base images
> 3. Scan images for vulnerabilities (`docker scout`, Trivy, Snyk)
> 4. Use read-only filesystem: `docker run --read-only`
> 5. Limit capabilities: `--cap-drop ALL --cap-add NET_BIND_SERVICE`
> 6. Set resource limits (memory, CPU)
> 7. Don't store secrets in images or environment variables
> 8. Use multi-stage builds to reduce attack surface
> 9. Keep images and Docker updated
> 10. Use Docker Content Trust (image signing)

---

**Q87. What is Docker Content Trust (DCT)?**
```bash
export DOCKER_CONTENT_TRUST=1
docker pull nginx  # only pulls if image is signed
docker push myrepo/myapp:1.0  # automatically signs image
```
> DCT ensures images are signed by trusted publishers. Prevents pulling tampered/untrusted images.

---

**Q88. How do you scan a Docker image for vulnerabilities?**
```bash
# Docker Scout (built-in, newer Docker versions)
docker scout cves myapp:latest

# Trivy (popular open-source scanner)
trivy image myapp:latest

# Snyk
snyk container test myapp:latest
```

---

**Q89. What is the `--read-only` flag?**
```bash
docker run --read-only \
  --tmpfs /tmp \           # allow writes only to /tmp (in memory)
  myapp
```
> Makes the container's root filesystem read-only. Prevents attackers from writing files to the container. Use `--tmpfs` for directories that need writes.

---

**Q90. What are Linux capabilities in Docker?**
> Linux capabilities are fine-grained permissions for the Linux kernel. By default, Docker drops many dangerous capabilities. Best practice:
```bash
docker run \
  --cap-drop ALL \                   # drop all capabilities
  --cap-add NET_BIND_SERVICE \       # add only what's needed
  nginx
```

---

**Q91. What is a Docker security namespace?**
> Docker uses Linux namespaces to isolate containers:
> - **PID namespace:** Container has its own process IDs
> - **Network namespace:** Container has its own network stack
> - **Mount namespace:** Container has its own filesystem view
> - **UTS namespace:** Container has its own hostname
> - **IPC namespace:** Container has its own inter-process communication
> - **User namespace:** Map container root to non-root on host

---

**Q92. What is `seccomp` in Docker?**
> Seccomp (Secure Computing Mode) filters system calls a container can make. Docker applies a default seccomp profile that blocks ~44 dangerous syscalls. You can customize it:
```bash
docker run --security-opt seccomp=custom-profile.json myapp
docker run --security-opt seccomp=unconfined myapp  # disable (not recommended)
```

---

**Q93. How do you handle environment variables securely in Docker?**
> ❌ **Bad:** `docker run -e DB_PASS=mypassword` (visible in `docker inspect` and process list)
> ✅ **Better:** Use Docker secrets (Swarm) or Kubernetes secrets
> ✅ **Better:** Read from file: `docker run -e DB_PASS_FILE=/run/secrets/db_pass`
> ✅ **Best:** Use HashiCorp Vault or AWS Secrets Manager, fetch at runtime

---

**Q94. What is AppArmor in Docker context?**
> AppArmor is a Linux security module. Docker can apply AppArmor profiles to containers to restrict what they can do (file access, network, capabilities). Docker ships with a default AppArmor profile.

---

**Q95. What is the difference between privileged and non-privileged containers?**
> - **Non-privileged (default):** Limited access to host. Can't access host devices. Recommended.
> - **Privileged:** `docker run --privileged` — Container has full access to host kernel, devices, capabilities. Very dangerous — gives root-like access to host. Only use when absolutely necessary (e.g., Docker-in-Docker).

---

**Q96. What is Docker-in-Docker (DinD)?**
> Running Docker inside a Docker container. Used in CI/CD to build Docker images inside Jenkins/GitLab CI containers.
```bash
docker run --privileged docker:dind
```
> **Security concern:** Requires `--privileged` mode.
> **Better alternative:** Mount the host Docker socket: `-v /var/run/docker.sock:/var/run/docker.sock` (but this also has security implications — container can control host Docker)

---

**Q97. What are some Docker performance best practices?**
> 1. Use Alpine or distroless images (smaller = faster pulls)
> 2. Enable BuildKit: `DOCKER_BUILDKIT=1 docker build`
> 3. Use `.dockerignore` to reduce build context size
> 4. Leverage layer caching properly
> 5. Use multi-stage builds
> 6. Set appropriate resource limits
> 7. Use `--no-install-recommends` for apt packages
> 8. Clean up in the same RUN layer as install

---

**Q98. What is Docker BuildKit?**
> BuildKit is the modern, improved Docker build engine. Features:
> - Parallel layer building (faster)
> - Better cache management
> - Secret mounting during build (no secret leaks)
> - SSH agent forwarding in build
```bash
DOCKER_BUILDKIT=1 docker build -t myapp .
# or in daemon config: { "features": { "buildkit": true } }
```

---

## 8. Advanced & Real-World Scenarios

**Q99. How do you implement CI/CD with Docker and Jenkins?**
```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "myrepo/myapp:${BUILD_NUMBER}"
    }
    stages {
        stage('Build Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Run Tests') {
            steps {
                sh "docker run --rm ${IMAGE_NAME} npm test"
            }
        }
        stage('Scan Image') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}"
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker login -u $USER -p $PASS"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}"
            }
        }
    }
}
```

---

**Q100. What is a distroless image?**
> Distroless images (by Google) contain ONLY the application and its runtime — no shell, no package manager, no OS utilities. Even smaller and more secure than Alpine.
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o myapp .

# Distroless final image
FROM gcr.io/distroless/static-debian12
COPY --from=builder /app/myapp /
CMD ["/myapp"]
```
> Attacker can't exec into container — no shell available!

---

**Q101. How do you debug a container that won't start?**
```bash
# Check why it failed
docker logs mycontainer
docker inspect mycontainer | grep -A 5 "State"

# Override entrypoint to get a shell
docker run -it --entrypoint /bin/sh myapp

# Run with more verbose output
docker run -it myapp /bin/bash -x /start.sh

# Check events
docker events --filter container=mycontainer
```

---

**Q102. What is the difference between `docker compose up --build` and just `docker compose up`?**
> - `docker compose up` — Uses existing images (cached). Doesn't rebuild.
> - `docker compose up --build` — Forces rebuild of images before starting.
> - `docker compose build` — Builds images without starting containers.

---

**Q103. How do you implement zero-downtime deployment with Docker?**
```bash
# Method 1: Docker Compose rolling update
docker compose pull app
docker compose up -d --no-deps --build app

# Method 2: Blue-Green with nginx
# Start new version on port 3001
docker run -d --name app-green -p 3001:3000 myapp:new
# Test it
curl http://localhost:3001/health
# Update nginx to point to green
docker exec nginx nginx -s reload
# Stop old blue container
docker stop app-blue
```

---

**Q104. What is a sidecar container pattern?**
> A sidecar is a helper container that runs alongside the main container in the same pod/service. Examples:
> - **Log collector:** Main app writes logs, sidecar ships them to ELK
> - **Proxy:** Envoy/Nginx sidecar handles traffic (Istio service mesh)
> - **Config reloader:** Sidecar watches for config changes and reloads main app
> In Docker Compose, both containers share a volume for communication.

---

**Q105. How do you do a Docker image vulnerability scan in CI/CD pipeline?**
```yaml
# GitHub Actions example
- name: Scan image with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myrepo/myapp:latest'
    format: 'sarif'
    exit-code: '1'           # fail pipeline if HIGH/CRITICAL found
    severity: 'HIGH,CRITICAL'
```

---

**Q106. What is `docker manifest` and multi-arch images?**
> Multi-arch images work on different CPU architectures (amd64, arm64, arm/v7). Docker manifest creates one tag that automatically serves the right architecture.
```bash
# Build for multiple architectures
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myrepo/myapp:latest --push .
```
> Important for: M1/M2 Macs, Raspberry Pi, AWS Graviton (ARM) servers.

---

**Q107. What is the init process in Docker and why does it matter?**
```bash
docker run --init nginx
```
> By default PID 1 in a container is your app. Apps aren't designed to handle signals properly as PID 1 (zombie processes, SIGTERM ignored). `--init` adds a tiny init process (`tini`) that:
> - Properly handles SIGTERM
> - Reaps zombie processes
> Alternatively, use `tini` in your Dockerfile:
```dockerfile
ENTRYPOINT ["/sbin/tini", "--", "node", "app.js"]
```

---

**Q108. How do you do live reload (hot reload) in Docker for development?**
```yaml
# docker-compose.dev.yml
services:
  app:
    build: .
    volumes:
      - .:/app                    # mount code into container
      - /app/node_modules         # don't overwrite node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev          # nodemon or similar hot-reload tool
    ports:
      - "3000:3000"
```
> Host code changes → immediately reflected in container — no rebuild needed.

---

**Q109. What is Docker layer squashing?**
```bash
docker build --squash -t myapp .
```
> Merges all layers into a single layer in the final image. Smaller image size, but loses caching benefits. Good for final production images where you want the smallest possible size.

---

**Q110. Real-world scenario: Complete Docker workflow for a microservices app.**
```
1. DEVELOP
   └── docker compose up (local dev with hot reload)

2. BUILD
   └── Dockerfile multi-stage build
   └── docker build -t myrepo/service:${GIT_SHA} .

3. TEST
   └── docker run --rm myrepo/service:${GIT_SHA} npm test

4. SCAN
   └── trivy image myrepo/service:${GIT_SHA}

5. PUSH
   └── docker push myrepo/service:${GIT_SHA}
   └── docker push myrepo/service:latest

6. DEPLOY (to Kubernetes)
   └── kubectl set image deployment/service \
         service=myrepo/service:${GIT_SHA}

7. MONITOR
   └── docker stats (locally)
   └── CloudWatch / Prometheus (production)

8. CLEANUP
   └── docker image prune -a --filter "until=24h"
```

---
---


