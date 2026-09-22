# Intro to Docker 

## What is docker?
```
Docker is an open-source platform that uses OS-level virtualization to deliver software in packages called containers.

Instead of bundling an entire operating system (like a Virtual Machine does), a container bundles an application together with all of its dependencies, libraries, configuration files, and runtime environment into a single isolated unit.
```

---

## Key Concepts

1. **Docker Image**: A lightweight, standalone, read-only blueprint or template that contains the application code, dependencies, and environment setup required to run a software program.

2. **Docker Container**: A runnable, isolated instance of a Docker Image. Containers execute application processes directly on the host system's OS kernel while remaining isolated from other processes.

3. **Dockerfile**: A text file containing a sequential list of commands used to assemble and build a Docker Image.

4. **Docker Hub**: A public registry service for sharing, discovering, and downloading pre-built container images (e.g., PostgreSQL, Node.js, Nginx).

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