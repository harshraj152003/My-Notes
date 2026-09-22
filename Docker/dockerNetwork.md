## Deep Networking

Docker provides multiple network drivers to handle container communication:

| Driver      | Description                                                                                                                            |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------- |
| **Bridge**  | Default network driver. Containers attached to the same custom bridge network can communicate via IP or container name (built-in DNS). |
| **Host**    | Removes network isolation between the container and the host machine (shares host IP stack).                                           |
| **None**    | Disables all networking for the container except the loopback interface (`127.0.0.1`).                                                 |
| **Overlay** | Enables communication between containers running across different Docker Daemon hosts (Swarm/Cluster mode).                            |

```bash
# Network Management
docker network create --driver bridge my-custom-net
docker network ls
docker network inspect my-custom-net
docker network connect my-custom-net my-container
docker network disconnect my-custom-net my-container
docker network rm my-custom-net

# Running containers on custom bridge network
docker run -d --network my-custom-net --name db-service postgres
docker run -d --network my-custom-net --name backend-service my-api-image
```

---
