## Orchestration & Low-Level Mechanics

### Multi-Container Orchestration (`docker compose`)

`docker compose` manages multi-container applications using a declarative YAML manifest (`docker-compose.yml`).

```yaml
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    volumes:
      - ./src:/app/src
    environment:
      - NODE_ENV=development
    depends_on:
      - redis
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  redis-data:
```

#### Core Compose Commands

- **`docker compose up -d`**: Builds, creates, and starts all containers defined in `docker-compose.yml` in detached mode.
- **`docker compose down`**: Stops and removes containers, networks, volumes, and images created by `up`.
  - Add `-v` flag to remove named volumes as well (`docker compose down -v`).
- **`docker compose ps`**: Lists containers managed by the current compose file.
- **`docker compose logs -f [SERVICE]`**: Streams logs for all or specific services defined in the configuration.

---

### Low-Level Linux Kernel Primitives

Docker containers are non-virtualized processes isolated on a Linux host via three primary kernel features:

1. **Namespaces (Isolation):** Provides virtualized views of system resources per container:
   - **`pid`**: Process isolation (container process sees itself as PID 1).
   - **`net`**: Network stack isolation (network interfaces, IP routing tables, port bindings).
   - **`mnt`**: Mount point isolation (filesystem tree).
   - **`ipc`**: Inter-Process Communication isolation (Shared Memory, System V IPC).
   - **`uts`**: Hostname and NIS Domain name isolation.
   - **`user`**: User and Group ID mapping (allows non-root host users to map as root inside container).
2. **Control Groups / cgroups (Resource Allocation):** Limits, accounts for, and isolates the resource usage (CPU, Memory, Disk I/O, Network bandwidth) of a group of processes.
3. **Capabilities & Seccomp (Security):**
   - **Capabilities:** Breaks down superuser (`root`) privileges into distinct units, allowing root processes inside containers to be dropped of high-risk capabilities (e.g., `CAP_SYS_ADMIN`).
   - **Seccomp (Secure Computing Mode):** Filters and restricts Linux system calls (syscalls) accessible to a container process.
