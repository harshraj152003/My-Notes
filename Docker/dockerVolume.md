### Storage Management

Containers use a Copy-on-Write (CoW) ephemeral storage layer. For persistent or high-performance data, external storage mechanisms are used.

1. **Volumes:** Managed directly by Docker in the host filesystem (`/var/lib/docker/volumes/`). Recommended for data persistence.
2. **Bind Mounts:** Maps an explicit path on the host system to a path inside the container.

```bash
# Create and manage Docker volumes
docker volume create <volume_name>
docker volume ls
docker volume inspect <volume_name>
docker volume rm <volume_name>

# Mount volume inside container
docker run -d -v my-volume:/app/data nginx

# Bind Mount host directory inside container
docker run -d -v /home/user/app_configs:/etc/nginx/conf.d nginx
```

---
