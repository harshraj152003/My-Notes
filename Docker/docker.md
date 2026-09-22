# Intro to Docker

## What is docker?

Docker is an open-source platform that uses OS-level virtualization to deliver software in packages called containers.

Instead of bundling an entire operating system (like a Virtual Machine does), a container bundles an application together with all of its dependencies, libraries, configuration files, and runtime environment into a single isolated unit.

---

## Key Concepts

1. **Docker Image**: A lightweight, standalone, read-only blueprint or template that contains the application code, dependencies, and environment setup required to run a software program.

2. **Docker Container**: A runnable, isolated instance of a Docker Image. Containers execute application processes directly on the host system's OS kernel while remaining isolated from other processes.

3. **Dockerfile**: A text file containing a sequential list of commands used to assemble and build a Docker Image.

4. **Docker Hub**: A public registry service for sharing, discovering, and downloading pre-built container images (e.g., PostgreSQL, Node.js, Nginx).

5. **Daemon (`dockerd`):** The background process managing Docker objects such as images, containers, networks, and volumes.

6. **CLI (`docker`):** The command-line interface used to issue commands to the Docker Daemon via REST API.

---

## The Core Problems Before Docker:

- The "**It Works on My Machine**" Syndrome (Environmental Drift)
  Before containers, a developer wrote code on their local laptop (e.g., macOS with Python 3.10, PostgreSQL 14, and specific libraries). When that code moved to testing or production servers (e.g., Ubuntu running Python 3.8 or slightly different dynamic libraries), it often failed.

- Causes: Mismatched language runtime versions, missing system dependencies, conflicting environment variables, or different OS configurations.

- Impact: Hours spent debugging differences in environments rather than actual code bugs.

- Heavy Overhead of Virtual Machines
  Before Docker, isolating applications required running distinct Virtual Machines (VMs) on a hypervisor (e.g., VMware, VirtualBox).

**The Problem**: Every VM requires a full copy of a Guest Operating System (OS), complete with its own kernel, system binaries, and drivers.

# Impact:

- Resource Waste: Running 5 small microservices required 5 separate Guest OS instances, taking up gigabytes of RAM and storage just for OS overhead.

- Slow Boot Times: Booting a VM meant booting an entire OS, taking minutes instead of seconds.

- Dependency Conflicts (Dependency Hell)
- Running multiple applications on a single server without VMs often caused version conflicts. For instance, if App A required Node.js 16 and App B required Node.js 20 on the same host, managing paths, system environment variables, and globally installed packages was complex and prone to breaking.

---

## Level 1: Container Lifecycle & Basic Operations

### `docker ps`

Lists containers on the host system.

```bash
docker ps [OPTIONS]
```

- **Flags:**
  - `-a`, `--all`: Show all containers (default shows only running).
  - `-q`, `--quiet`: Display only numeric container IDs.
  - `-s`, `--size`: Display total file sizes.
  - `-n <int>`: Show the last $N$ created containers.
  - `--filter "key=value"`: Filter output based on conditions (e.g., `status=exited`, `name=web`).
  - `--format "<template>"`: Pretty-print output using Go templates.
    - _Example:_ `docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}\t{{.Ports}}"`

---

### Container Lifecycle Commands

#### `docker run`

Creates and starts a container from a specified image.

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

- **Flags:**
  - `-d`: Run container in detached mode (background).
  - `-it`: Instructs Docker to allocate a pseudo-TTY attached to the container's standard input (`-i` interactive, `-t` TTY).
  - `--name <name>`: Assign a custom container name.
  - `-p <host_port>:<container_port>`: Publish/map host ports to container ports.
  - `-e <KEY=VALUE>`: Set environment variables.
  - `--rm`: Automatically remove the container when it exits.
  - `--restart <policy>`: Restart policy (`no`, `on-failure`, `always`, `unless-stopped`).
- **Example:**
  ```bash
  docker run -d -p 8080:80 --name my-nginx --restart unless-stopped nginx:latest
  ```

#### `docker start`

Starts one or more stopped containers.

```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

- **Flags:**
  - `-i`: Attach container's `STDOUT`/`STDERR` and forward `STDIN`.
  - `-a`: Attach container's `STDOUT`/`STDERR`.

#### `docker stop`

Gracefully stops a running container by sending `SIGTERM`, followed by `SIGKILL` if it fails to stop within the grace period.

```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

- **Flags:**
  - `-t`, `--time <int>`: Seconds to wait before killing the container (default: `10`).

#### `docker kill`

Kills one or more running containers immediately using `SIGKILL` or a specified signal.

```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```

- **Flags:**
  - `-s`, `--signal <signal>`: Send a custom signal (e.g., `SIGHUP`).

#### `docker restart`

Restarts one or more running or stopped containers.

```bash
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

#### `docker rm`

Removes one or more stopped containers from disk.

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

- **Flags:**
  - `-f`, `--force`: Force the removal of a running container (uses `SIGKILL`).
  - `-v`, `--volumes`: Remove anonymous volumes associated with the container.

#### `docker logs`

Fetches log messages generated by the main process (PID 1) of a container.

```bash
docker logs [OPTIONS] CONTAINER
```

- **Flags:**
  - `-f`, `--follow`: Stream live log output.
  - `--tail <int>`: Number of lines to show from the end of the logs (e.g., `--tail 100`).
  - `-t`, `--timestamps`: Show timestamps with log entries.

---

## Level 2: Inspection, Execution, & Storage

### Inspection & Execution

#### `docker exec`

Executes a new command inside an active/running container.

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

- **Flags:**
  - `-it`: Run interactively with a pseudo-TTY.
  - `-w`, `--workdir <dir>`: Set working directory inside the container.
  - `-u`, `--user <user>`: Username or UID to run command as.
- **Example:**
  ```bash
  docker exec -it my-nginx /bin/sh
  ```

#### `docker inspect`

Returns detailed low-level JSON configuration and state data for Docker objects.

```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

- **Flags:**
  - `-f`, `--format "<template>"`: Format output using Go templates.
- **Example:**
  ```bash
  docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-nginx
  ```

#### `docker stats`

Displays a live, streaming data feed of container resource usage.

```bash
docker stats [OPTIONS] [CONTAINER...]
```

- **Flags:**
  - `--no-stream`: Disable streaming stats and pull current snapshot only.

---

## Level 3: Images & Dockerfile Engineering

### Image Architecture

Images consist of read-only layers stacked on top of each other using union file systems (e.g., `OverlayFS2`). Each instruction in a `Dockerfile` creates a separate immutable layer.

### Core `Dockerfile` Directives

```dockerfile
# Specify base image
FROM node:18-alpine AS build

# Set working directory inside container
WORKDIR /usr/src/app

# Copy dependency files
COPY package*.json ./

# Execute build-time commands
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# Multi-stage production image
FROM nginx:alpine
COPY --from=build /usr/src/app/dist /usr/share/nginx/html

# Inform Docker that container listens on specified port at runtime
EXPOSE 80

# Default command executed when container starts
CMD ["nginx", "-g", "daemon off;"]
```

### Image Management Commands

#### `docker build`

Builds an image from a `Dockerfile` and context directory.

```bash
docker build [OPTIONS] PATH | URL
```

- **Flags:**
  - `-t`, `--tag <name:tag>`: Name and optionally a tag in `name:tag` format.
  - `-f`, `--file <path>`: Path to the `Dockerfile`.
  - `--no-cache`: Do not use cache when building the image.
  - `--build-arg <var=value>`: Pass build-time variables.
- **Example:**
  ```bash
  docker build -t my-app:v1.0 -f Dockerfile.prod .
  ```

#### `docker images` / `docker image ls`

Lists locally available images.

```bash
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

#### `docker rmi` / `docker image rm`

Removes one or more images from local host storage.

```bash
docker image rm [OPTIONS] IMAGE [IMAGE...]
```

- **Flags:**
  - `-f`, `--force`: Force removal of the image.

#### `docker history`

Shows the build history and individual layer sizes of an image.

```bash
docker history [OPTIONS] IMAGE
```

---
