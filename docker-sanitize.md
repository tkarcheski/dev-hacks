To clean up all containers in Docker, including both running and stopped containers, you can follow these steps:

### 1. **Stop all running containers**
   First, stop all running containers using the following command:

   ```bash
   docker stop $(docker ps -q)
   ```

   - `docker ps -q` returns the IDs of all running containers.
   - `docker stop` stops those containers.

### 2. **Remove all containers**
   After stopping them, you can remove all containers (both running and stopped) with:

   ```bash
   docker rm $(docker ps -a -q)
   ```

   - `docker ps -a -q` returns the IDs of all containers, including stopped ones.
   - `docker rm` removes those containers.

### 3. **Optional: Remove all images**
   If you also want to remove all images, you can do so with:

   ```bash
   docker rmi $(docker images -q)
   ```

   - `docker images -q` returns the IDs of all images.
   - `docker rmi` removes those images.

### 4. **Optional: Remove all volumes**
   To remove all volumes, use:

   ```bash
   docker volume rm $(docker volume ls -q)
   ```

   - `docker volume ls -q` returns the names of all volumes.
   - `docker volume rm` removes those volumes.

### 5. **Optional: Remove all networks**
   If you want to remove all Docker networks (except the default ones), use:

   ```bash
   docker network rm $(docker network ls -q)
   ```

   - `docker network ls -q` returns the names of all networks.
   - `docker network rm` removes those networks.

### Summary of Commands:
- Stop all running containers: `docker stop $(docker ps -q)`
- Remove all containers: `docker rm $(docker ps -a -q)`
- Optional: Remove all images: `docker rmi $(docker images -q)`
- Optional: Remove all volumes: `docker volume rm $(docker volume ls -q)`
- Optional: Remove all networks: `docker network rm $(docker network ls -q)`

This set of commands will thoroughly clean up your Docker environment, removing all containers, images, volumes, and networks.
