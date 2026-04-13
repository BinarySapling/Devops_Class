# Docker Networks: Connecting Containers

Docker Networking allows containers to communicate with each other, the host machine, and the outside world. By default, Docker creates a few standard networks, but you can create custom networks to provide better isolation and DNS resolution between your containers.

## Types of Docker Network Drivers

1. **Bridge (default):** The default network driver. If you don't specify a network, containers attach to the default bridge network. However, custom user-defined bridge networks are superior because they provide automatic DNS resolution between containers.
2. **Host:** Removes network isolation between the container and the Docker host, and uses the host's networking directly.
3. **None:** Disables all networking for the container.
4. **Overlay:** Connects multiple Docker daemons together and enables swarm services to communicate with each other.
5. **Macvlan:** Allows you to assign a MAC address to a container, making it appear as a physical device on your network.

---

## Tutorial: Pinging Between Two Nginx Containers

By default, containers on the default `bridge` network can only communicate via IP addresses. If we create a **custom bridge network**, Docker provides an automatic DNS resolver so containers can resolve each other by their container names.

Let's test this by putting two Nginx containers on a custom network and pinging one from the other.

### Step 1: Create a Custom Network
Create a user-defined bridge network named `my-nginx-net`.

```bash
docker network create my-nginx-net
```

### Step 2: Run Two Nginx Containers
We'll use the `nginx:alpine` image because it includes the `ping` utility by default. We will attach both containers to the newly created `my-nginx-net`.

```bash
# Start the first container
docker run -d --name nginx-alpha --network my-nginx-net nginx:alpine

# Start the second container
docker run -d --name nginx-beta --network my-nginx-net nginx:alpine
```

### Step 3: Verify the Network Connection
Now, let's execute a command inside `nginx-alpha` to ping `nginx-beta` by its container name.

```bash
docker exec -it nginx-alpha ping -c 4 nginx-beta
```

**Expected Output:**
```text
PING nginx-beta (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.089 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.076 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.081 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.078 ms
```
Because they are on the same custom network, `nginx-alpha` successfully resolved `nginx-beta`'s IP address and pinged it!

---

## Complete List of Docker Network Commands

Here are all the foundational `docker network` commands you'll use to manage networking:

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker network ls` | Lists all networks available on the Docker host. | `docker network ls` |
| `docker network create` | Creates a new custom network. | `docker network create my-net` |
| `docker network inspect` | Shows detailed information (like IP subnets and attached containers) for one or more networks. | `docker network inspect my-net` |
| `docker network connect` | Connects a running container to a network. | `docker network connect my-net my-container` |
| `docker network disconnect` | Disconnects a running container from a network. | `docker network disconnect my-net my-container` |
| `docker network rm` | Removes one or more networks (no containers can be attached). | `docker network rm my-net` |
| `docker network prune` | Removes all unused (unattached) networks. | `docker network prune` |

### Pro-Tips:
* **Cleaning up:** Don't forget to stop and remove your testing containers: `docker rm -f nginx-alpha nginx-beta` followed by `docker network rm my-nginx-net`.
* **Port Mapping (`-p`) vs Inner Networking:** You don't need `-p` to let containers talk to each other on the same Docker network. Port mapping is only for exposing the container's port to your local host machine.