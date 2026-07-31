# Docker Learning Notes

A practical Docker reference covering the topics learned so far, with commands, Dockerfile examples, explanations, and the next important concepts: volumes, bind mounts, environment variables, `.dockerignore`, and multi-stage builds.

---

## 1. Docker Images and Containers

### What is a Docker image?
A Docker image is a read-only template containing the application, runtime, libraries, dependencies, and configuration required to create a container.

### What is a Docker container?
A container is a running (or stopped) instance of an image.

### List images

```bash
docker image ls
```

Alternative:

```bash
docker images
```

### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### Stop a container

```bash
docker stop <container-id-or-name>
```

Example:

```bash
docker stop my-nginx
```

### Start an existing stopped container

```bash
docker start <container-id-or-name>
```

### Remove a container

```bash
docker rm <container-id-or-name>
```

Force remove a running container:

```bash
docker rm -f <container-id-or-name>
```

### Remove an image

```bash
docker rmi <image-id-or-name>
```

---

## 2. Building Docker Images

A `Dockerfile` contains instructions Docker uses to build an image.

### Basic build

```bash
docker build . -t myapp:1.0
```

- `docker build` — builds an image
- `.` — current directory is the build context
- `-t` — assigns a name and tag
- `myapp` — image name
- `1.0` — image tag

### Build using a specific Dockerfile

```bash
docker build -f Dockerfile.dev . -t myapp:dev
```

Using a full/relative path:

```bash
docker build --file "./docker/Dockerfile" . -t myapp:1.0
```

### Build from a Git repository

```bash
docker build https://github.com/user/repository.git -t myapp:1.0
```

The final argument to `docker build` is the **build context**. Usually use either `.` or a Git URL as the context.

### Build without cache

```bash
docker build --no-cache . -t myapp:1.0
```

---

## 3. Running Containers — `docker run`

### Basic run

```bash
docker run ubuntu
```

### Interactive container

```bash
docker run -it ubuntu bash
```

- `-i` — keeps STDIN open
- `-t` — allocates a terminal

### Run in detached/background mode

```bash
docker run -d nginx
```

### Assign a container name

```bash
docker run -d --name my-nginx nginx
```

### Automatically remove container after it exits

```bash
docker run --rm ubuntu echo "Hello Docker"
```

### View container logs

```bash
docker logs <container-id-or-name>
```

Follow logs continuously:

```bash
docker logs -f <container-id-or-name>
```

---

## 4. Port Mapping — `-p`

Containers have their own network namespace. Use `-p` to publish a container port on the Docker host.

```bash
docker run -d -p 8080:80 nginx
```

Syntax:

```text
-p <host-port>:<container-port>
```

For the example above:

```text
Browser / Host                 Container
localhost:8080  ------------>  port 80
```

Another example:

```bash
docker run -d -p 3000:3000 myapp:1.0
```

---

## 5. `EXPOSE` vs `-p`

Dockerfile:

```dockerfile
FROM nginx
EXPOSE 80
```

`EXPOSE 80` documents that the containerized application is expected to listen on port 80. It does **not** by itself publish the port to the host.

Publish it when running:

```bash
docker run -d -p 8080:80 myimage
```

You can also use `-P` to publish exposed ports to automatically selected host ports:

```bash
docker run -d -P nginx
```

Check mappings:

```bash
docker ps
```

---

## 6. `docker exec`

`docker exec` runs a command inside an **already-running** container.

### Open Bash

```bash
docker exec -it <container-id> bash
```

Example:

```bash
docker exec -it my-nginx bash
```

If Bash is unavailable, try `sh`:

```bash
docker exec -it my-nginx sh
```

### Execute a single command

```bash
docker exec my-nginx ls /usr/share/nginx/html
```

---

## 7. Dockerfile `RUN`

`RUN` executes a command **while the image is being built**.

Example:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update
RUN apt-get install -y curl
```

Important distinction:

- `RUN` — executes during `docker build`
- `CMD` / `ENTRYPOINT` — define what runs when a container starts
- `docker exec` — runs a command inside an existing running container

---

## 8. Dockerfile `CMD`

`CMD` provides the default command or default arguments for a container.

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

Run:

```bash
docker build . -t python-app:1.0
docker run python-app:1.0
```

A command supplied after the image name can override `CMD`:

```bash
docker run python-app:1.0 python another-script.py
```

Prefer JSON/exec form:

```dockerfile
CMD ["python", "app.py"]
```

---

## 9. Dockerfile `ENTRYPOINT`

`ENTRYPOINT` defines the main executable for the container.

```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY . .
ENTRYPOINT ["python"]
CMD ["app.py"]
```

The effective command is:

```text
python app.py
```

If you run:

```bash
docker run myapp another.py
```

Docker effectively executes:

```text
python another.py
```

Useful mental model:

```text
ENTRYPOINT = main executable
CMD        = default command/arguments
```

Override an entrypoint when required:

```bash
docker run --entrypoint sh myapp
```

---

## 10. Docker Image Layers

Docker images are built from layers. Dockerfile instructions such as `RUN`, `COPY`, and `ADD` contribute filesystem changes to the resulting image, while Docker also stores configuration metadata for instructions such as `CMD`, `ENTRYPOINT`, `ENV`, and `EXPOSE`.

Example:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update
RUN apt-get install -y curl
COPY . /app
```

Conceptually:

```text
Layer / metadata: FROM ubuntu:24.04
Layer:            RUN apt-get update
Layer:            RUN apt-get install -y curl
Layer:            COPY . /app
```

Docker can reuse cached build results when an instruction and everything it depends on has not changed.

### View image history/layers

```bash
docker history <image-name>
```

Example:

```bash
docker history myapp:1.0
```

### Why ordering matters

If dependencies change less frequently than application source, copy dependency files first so Docker can reuse cached dependency-installation layers.

Example for Node.js:

```dockerfile
FROM node:22-slim
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
CMD ["npm", "start"]
```

---

## 11. Reducing Docker Image Size

### Combine related `RUN` commands

Instead of:

```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y python3
RUN apt-get install -y apache2
```

Use:

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        python3 \
        apache2 && \
    rm -rf /var/lib/apt/lists/*
```

This prevents package-list files from being left behind in an earlier immutable layer.

### Use a smaller appropriate base image

Instead of a general OS image when unnecessary:

```dockerfile
FROM ubuntu:24.04
```

A purpose-built slim image may be better:

```dockerfile
FROM python:3.13-slim
```

Or, when your application and dependencies are compatible:

```dockerfile
FROM alpine:3.22
```

Smaller is not automatically better: consider compatibility, security updates, debugging needs, and native library requirements.

### Other size-reduction techniques

- Use `.dockerignore`.
- Avoid copying build caches and local dependencies.
- Remove package-manager caches in the same `RUN` instruction that creates them.
- Use `--no-install-recommends` with `apt` where appropriate.
- Use multi-stage builds.
- Copy only files required at runtime.

Check image size:

```bash
docker image ls
```

---

## 12. Export and Import Containers

### Export a container filesystem

```bash
docker export <container-id> > container.tar
```

Alternative:

```bash
docker export -o container.tar <container-id>
```

### Import the filesystem as a new image

```bash
docker import container.tar myimage:1.0
```

### `docker save/load` vs `docker export/import`

#### Save an image

```bash
docker save -o myimage.tar myimage:1.0
```

Load it later:

```bash
docker load -i myimage.tar
```

Use `save/load` when you want to transfer Docker images while preserving image metadata, tags, and layers.

#### Export a container

```bash
docker export -o container.tar mycontainer
```

Import it:

```bash
docker import container.tar myimage:1.0
```

`export/import` works with a flattened container filesystem snapshot and does not preserve the original image history/layers and much of its configuration metadata.

---

## 13. Docker Hub — Login, Tag, Push and Pull

### Login

```bash
docker login
```

For automation, prefer a personal access token and `--password-stdin` rather than putting a password directly in a command.

Example pattern:

```bash
echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
```

Do not hardcode credentials in Dockerfiles, Git repositories, or pipeline YAML.

### Tag an image

```bash
docker tag myapp:1.0 username/myapp:1.0
```

### Push

```bash
docker push username/myapp:1.0
```

### Pull

```bash
docker pull username/myapp:1.0
```

Typical workflow:

```bash
docker build . -t myapp:1.0
docker tag myapp:1.0 username/myapp:1.0
docker login
docker push username/myapp:1.0
```

---

# Additional Topics

## 14. Docker Volumes

Containers are disposable. Data stored only in a container's writable layer can disappear when the container is removed.

Docker volumes provide persistent storage managed by Docker.

### Create a volume

```bash
docker volume create mydata
```

### List volumes

```bash
docker volume ls
```

### Inspect a volume

```bash
docker volume inspect mydata
```

### Use a volume with `-v`

```bash
docker run -d \
  --name my-nginx \
  -v mydata:/usr/share/nginx/html \
  nginx
```

Syntax:

```text
-v <volume-name>:<container-path>
```

### Use `--mount`

```bash
docker run -d \
  --name my-nginx \
  --mount source=mydata,target=/usr/share/nginx/html \
  nginx
```

`--mount` is more explicit and is often easier to read in scripts.

### Remove a volume

```bash
docker volume rm mydata
```

Remove unused volumes carefully:

```bash
docker volume prune
```

### Database example

```bash
docker volume create postgres-data

docker run -d \
  --name postgres-db \
  -e POSTGRES_PASSWORD=ChangeMe \
  -v postgres-data:/var/lib/postgresql/data \
  postgres
```

The database files remain in the volume even if the container is removed, unless the volume itself is deleted.

---

## 15. Bind Mounts

A bind mount maps a file or directory from the **host machine** directly into a container.

This is especially useful during development because changes to source files on the host are immediately visible inside the container.

### Using `-v`

Linux/macOS example:

```bash
docker run -d \
  -p 8080:80 \
  -v "$(pwd)/website:/usr/share/nginx/html" \
  nginx
```

PowerShell example:

```powershell
docker run -d -p 8080:80 -v "${PWD}/website:/usr/share/nginx/html" nginx
```

### Using `--mount`

PowerShell:

```powershell
docker run -d `
  -p 8080:80 `
  --mount "type=bind,source=${PWD}/website,target=/usr/share/nginx/html" `
  nginx
```

Linux/macOS:

```bash
docker run -d \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)/website",target=/usr/share/nginx/html \
  nginx
```

### Read-only bind mount

```bash
docker run -d \
  --mount type=bind,source="$(pwd)/config",target=/app/config,readonly \
  myapp:1.0
```

### Volume vs bind mount

| Feature | Volume | Bind Mount |
|---|---|---|
| Managed by Docker | Yes | No |
| Uses host path directly | No | Yes |
| Good for persistent app/database data | Yes | Sometimes |
| Good for live source-code development | Sometimes | Yes |
| Host-directory structure dependency | Low | High |

---

## 16. Environment Variables — `-e`

Environment variables allow configuration to be passed to a container without rebuilding the image.

### Pass one variable

```bash
docker run -e APP_ENV=production myapp:1.0
```

### Pass multiple variables

```bash
docker run \
  -e APP_ENV=production \
  -e APP_PORT=8080 \
  myapp:1.0
```

### Inspect variables inside a running container

```bash
docker exec <container-name> env
```

Example:

```bash
docker exec myapp env
```

### Pass an existing host environment variable

Linux/macOS:

```bash
export APP_ENV=production
docker run -e APP_ENV myapp:1.0
```

PowerShell:

```powershell
$env:APP_ENV = "production"
docker run -e APP_ENV myapp:1.0
```

---

## 17. Environment Files — `--env-file`

Instead of passing many `-e` options, store variables in a file.

Example `.env`:

```text
APP_ENV=production
APP_PORT=8080
LOG_LEVEL=info
```

Run:

```bash
docker run --env-file .env myapp:1.0
```

Important: do not commit files containing passwords, tokens, connection strings, or other secrets to Git. Add sensitive local environment files to `.gitignore`.

---

## 18. Dockerfile `ENV`

`ENV` sets environment variables in the image that are available to the running container by default.

```dockerfile
FROM python:3.13-slim

ENV APP_ENV=production
ENV APP_PORT=8080

WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

Override at runtime:

```bash
docker run -e APP_ENV=development myapp:1.0
```

Runtime `-e` values override the image's default `ENV` value.

Do not use Dockerfile `ENV` for secrets because the value becomes part of image configuration and can be inspected.

---

## 19. Dockerfile `ARG`

`ARG` defines a variable available during the **image build**.

Dockerfile:

```dockerfile
FROM ubuntu:24.04

ARG APP_VERSION=1.0
RUN echo "Building application version ${APP_VERSION}"
```

Build with the default:

```bash
docker build . -t myapp:1.0
```

Override it:

```bash
docker build \
  --build-arg APP_VERSION=2.0 \
  . -t myapp:2.0
```

### `ARG` vs `ENV`

| `ARG` | `ENV` |
|---|---|
| Primarily build-time | Available in the built image/container |
| Passed using `--build-arg` | Passed/overridden using `-e` at runtime |
| Useful for build configuration | Useful for application runtime configuration |

Do **not** use `ARG` for passwords or tokens. Build arguments can be exposed through build metadata/history. Use BuildKit secret mounts for sensitive build-time values when required.

---

## 20. `.dockerignore`

`.dockerignore` prevents unnecessary files from being sent in the Docker build context.

Example project:

```text
myapp/
├── Dockerfile
├── .dockerignore
├── app.py
├── requirements.txt
├── .git/
├── .env
├── logs/
└── __pycache__/
```

Example `.dockerignore`:

```text
.git
.gitignore
.env
*.log
logs/
__pycache__/
*.pyc
node_modules/
coverage/
.vscode/
.idea/
```

Then:

```bash
docker build . -t myapp:1.0
```

Benefits:

- Smaller build context
- Faster transfer to the Docker builder
- Better cache behavior
- Lower risk of accidentally copying local files/secrets into an image

`.dockerignore` is conceptually similar to `.gitignore`, but controls the Docker build context rather than Git tracking.

---

## 21. Multi-Stage Builds

Multi-stage builds allow you to use one image to **build** an application and a different, usually smaller image to **run** it.

This keeps compilers, package caches, source code, and other build-only dependencies out of the final runtime image.

### Simple example

```dockerfile
FROM golang:1.24 AS builder

WORKDIR /app
COPY . .
RUN go build -o myapp .

FROM debian:bookworm-slim
WORKDIR /app
COPY --from=builder /app/myapp ./myapp
CMD ["./myapp"]
```

Build:

```bash
docker build . -t myapp:1.0
```

Run:

```bash
docker run --rm myapp:1.0
```

Only the compiled binary is copied from the `builder` stage into the final image.

### Node.js example

```dockerfile
FROM node:22 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-slim AS runtime

WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=builder /app/dist ./dist

CMD ["node", "dist/index.js"]
```

### Why use multi-stage builds?

```text
Builder image
├── source code
├── compiler/build tools
├── development dependencies
└── build output
          |
          | COPY --from=builder
          v
Runtime image
├── runtime dependencies
└── build output
```

Benefits include:

- Smaller final images
- Fewer unnecessary packages in production
- Reduced attack surface
- Cleaner separation between build and runtime dependencies
- Easier production Dockerfiles

### Build a specific stage

```bash
docker build --target builder . -t myapp-builder:1.0
```

---

## 22. Useful Docker Inspection and Cleanup Commands

### Inspect a container

```bash
docker inspect <container-id-or-name>
```

### Inspect an image

```bash
docker inspect <image-name>
```

### View Docker disk usage

```bash
docker system df
```

### Remove unused stopped containers

```bash
docker container prune
```

### Remove unused images

```bash
docker image prune
```

### Remove unused Docker objects

```bash
docker system prune
```

Be careful with cleanup commands, especially on development/build machines where cached images and volumes may still be useful.

---

# Complete Practice Example

The following small example combines several concepts.

## Project structure

```text
my-python-app/
├── Dockerfile
├── .dockerignore
├── .env
├── requirements.txt
└── app.py
```

## `app.py`

```python
import os

print("Docker application started")
print("Environment:", os.getenv("APP_ENV", "development"))
```

## `requirements.txt`

```text
# Add Python dependencies here when needed
```

## `.dockerignore`

```text
.git
.env
__pycache__/
*.pyc
*.log
.venv/
```

## `Dockerfile`

```dockerfile
FROM python:3.13-slim

ARG APP_VERSION=1.0
ENV APP_ENV=production

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

## Build

```bash
docker build \
  --build-arg APP_VERSION=1.0 \
  . -t my-python-app:1.0
```

PowerShell one-line equivalent:

```powershell
docker build --build-arg APP_VERSION=1.0 . -t my-python-app:1.0
```

## Run

```bash
docker run --rm my-python-app:1.0
```

Override environment:

```bash
docker run --rm -e APP_ENV=development my-python-app:1.0
```

Use an env file:

```bash
docker run --rm --env-file .env my-python-app:1.0
```

---

# Quick Command Cheat Sheet

```bash
# Images
docker image ls
docker pull nginx
docker rmi <image>
docker history <image>

# Build
docker build . -t myapp:1.0
docker build -f Dockerfile.dev . -t myapp:dev
docker build --no-cache . -t myapp:1.0
docker build --build-arg APP_VERSION=2.0 . -t myapp:2.0

# Containers
docker run <image>
docker run -it <image> bash
docker run -d --name myapp <image>
docker run --rm <image>
docker ps
docker ps -a
docker stop <container>
docker start <container>
docker rm <container>
docker logs -f <container>
docker exec -it <container> bash

# Ports
docker run -d -p 8080:80 nginx

# Environment
docker run -e APP_ENV=production myapp
docker run --env-file .env myapp

# Volumes
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker run -v mydata:/data myapp
docker run --mount source=mydata,target=/data myapp
docker volume rm mydata

# Registry
docker login
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0
docker pull username/myapp:1.0

# Image transfer
docker save -o myimage.tar myimage:1.0
docker load -i myimage.tar

# Container filesystem transfer
docker export -o container.tar mycontainer
docker import container.tar myimage:1.0

# Inspection / cleanup
docker inspect <container-or-image>
docker system df
docker container prune
docker image prune
docker system prune
```

---

## Suggested Next Topics

After becoming comfortable with everything in this README, continue with:

1. Docker networking and custom bridge networks
2. Container-to-container communication and Docker DNS
3. Health checks
4. Restart policies
5. Docker Compose
6. Docker security and running as a non-root user
7. Image vulnerability scanning
8. Registry authentication in CI/CD
9. Building and pushing images from Azure DevOps/Jenkins
10. Kubernetes fundamentals

---

**Goal:** Be able to take an application, write an efficient Dockerfile, configure it through environment variables, persist its data, build a small production image, push it to a registry, and run it consistently in different environments.

## COPY and ADD

`COPY` and `ADD` are Dockerfile instructions used to place files or directories from the **build context** into a Docker image.

### COPY

`COPY` is recommended for normal file and directory copying because its behavior is simple and predictable.

**Syntax:**

```dockerfile
COPY <source> <destination>
```

**Copy one file:**

```dockerfile
FROM ubuntu:24.04
COPY index.html /var/www/html/index.html
```

**Copy a directory:**

Project:

```text
my-project/
├── Dockerfile
└── website/
    ├── index.html
    ├── style.css
    └── images/
```

Dockerfile:

```dockerfile
FROM nginx:alpine
COPY website/ /usr/share/nginx/html/
```

Build and run:

```bash
docker build -t my-website:1.0 .
docker run -d -p 8080:80 --name my-website my-website:1.0
```

**Copy multiple files:**

```dockerfile
COPY index.html style.css /var/www/html/
```

**Copy the build-context contents:**

```dockerfile
WORKDIR /app
COPY . .
```

When using `COPY . .`, use `.dockerignore` to keep unnecessary or sensitive files out of the build context.

### ADD

`ADD` can copy local files and directories too, but it has additional behavior.

```dockerfile
ADD <source> <destination>
```

For example:

```dockerfile
FROM ubuntu:24.04
ADD index.html /var/www/html/index.html
```

This works, but when you only want to copy a normal file, prefer:

```dockerfile
COPY index.html /var/www/html/index.html
```

### ADD with a local tar archive

A useful `ADD` feature is automatic extraction of recognized **local tar archives** into the image.

Project:

```text
my-project/
├── Dockerfile
└── website.tar.gz
```

Dockerfile:

```dockerfile
FROM httpd:2.4
ADD website.tar.gz /usr/local/apache2/htdocs/
```

Build and run:

```bash
docker build -t apache-website:1.0 .
docker run -d -p 8080:80 --name apache-site apache-website:1.0
```

Docker extracts the local tar archive into the destination during the image build.

### COPY vs ADD

| Feature | COPY | ADD |
|---|---|---|
| Copy local files | Yes | Yes |
| Copy local directories | Yes | Yes |
| Automatically extract local tar archives | No | Yes |
| Simple and predictable | Yes | Has extra behavior |
| Preferred for ordinary copying | **Yes** | No |
| Useful for local tar extraction | No | **Yes** |

**Rule of thumb:** use `COPY` by default. Use `ADD` when you intentionally need an additional `ADD` capability, such as extracting a local tar archive.

```dockerfile
# Recommended for ordinary copying
COPY website/ /var/www/html/

# Useful when automatic local tar extraction is intentional
ADD website.tar.gz /var/www/html/
```

### COPY/ADD and Docker build cache

`COPY` and `ADD` change the image filesystem and affect build-cache invalidation. Put dependency manifests before frequently changing application source when possible.

Good example:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

If only application code changes, Docker can often reuse the cached dependency-installation layer because `requirements.txt` did not change.

### Build context

When you run:

```bash
docker build -t myapp:1.0 .
```

the final `.` is the **build context**. `COPY` and `ADD` source paths are resolved from that context. Do not rely on them to reach arbitrary files outside it; choose the correct build context instead.

### Practical COPY example

```text
docker-demo/
├── Dockerfile
├── .dockerignore
├── requirements.txt
└── src/
    └── app.py
```

Dockerfile:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

EXPOSE 5000

CMD ["python", "app.py"]
```

`.dockerignore`:

```text
.git
__pycache__
*.pyc
.env
```

Build and run:

```bash
docker build -t docker-demo:1.0 .
docker run -d -p 5000:5000 --name docker-demo docker-demo:1.0
```

> **Remember:** Prefer `COPY` for normal copying. Use `ADD` only when you deliberately need its extra behavior.
