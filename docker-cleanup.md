To clean up old Docker containers, you can use the following commands:

### 1. **Remove all stopped containers:**

```bash
docker container prune
```

This command will remove all containers that are not running. You'll be prompted to confirm before the containers are deleted.

### 2. **Remove a specific container:**

```bash
docker rm <container_id>
```

Replace `<container_id>` with the actual ID of the container you want to remove. You can also use the container name instead of the ID.

### 3. **Remove all containers (both running and stopped):**

```bash
docker rm $(docker ps -a -q)
```

This command will remove all containers, both running and stopped. To force remove running containers as well, add the `-f` flag:

```bash
docker rm -f $(docker ps -a -q)
```

### 4. **Remove all unused Docker objects (containers, networks, images, volumes):**

```bash
docker system prune
```

This will remove all stopped containers, unused networks, and dangling images. You'll be asked to confirm the action.

If you want to remove **all unused images, not just dangling ones**, and also unused volumes, you can use:

```bash
docker system prune -a --volumes
```

### 5. **Remove dangling volumes:**

```bash
docker volume prune
```

This will remove all volumes not referenced by any containers.

By using these commands, you can keep your Docker environment clean and free from unnecessary old containers and other objects.
