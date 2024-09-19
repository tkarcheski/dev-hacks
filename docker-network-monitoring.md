To see which ports are in use by Docker containers, you can use the following methods:

### Method 1: Using `docker ps`
You can use the `docker ps` command to list all running containers along with the ports they are using:

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Ports}}"
```

This command will display a table with the container ID, names, and the ports they are using.

### Method 2: Using `docker inspect`
If you want more detailed information about a specific container, you can use the `docker inspect` command:

```bash
docker inspect --format='{{.Name}} - {{.NetworkSettings.Ports}}' <container_id>
```

Replace `<container_id>` with the actual container ID or name. This command will show you the port mappings for that specific container.

### Method 3: Checking Active Ports with `netstat` or `ss`
You can also use `netstat` or `ss` to see which ports are being used by Docker on your host system:

- On Linux, you can use:

  ```bash
  sudo netstat -tuln | grep -i 'docker-proxy'
  ```

  Or, using `ss`:

  ```bash
  sudo ss -tuln | grep -i 'docker-proxy'
  ```

  This will show you all the TCP/UDP ports that Docker is currently binding on your host system.

### Method 4: Checking Docker-specific Ports on Windows with `netstat`
If you are running Docker on Windows, you can use the `netstat` command:

```cmd
netstat -an | findstr LISTENING
```

This will show all the listening ports, and you can look for those associated with Docker.

### Example Output from `docker ps`
Here's what the output might look like:

```bash
CONTAINER ID   NAMES          PORTS
b8f3d8d7e8a4   my_container   0.0.0.0:8080->80/tcp
```

In this example:
- `0.0.0.0:8080->80/tcp` means that port `8080` on the host is forwarded to port `80` in the container.

These methods should help you identify which ports Docker is using on your system.
