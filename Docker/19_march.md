# Docker Basics & Commands – CA-1 Prep Notes

## Core Concepts (First Principles)
* **Docker:** Tool to run applications in isolated environments (containers)
* **Image:** Blueprint/template
* **Container:** Running instance of an image
* **Volume:** Persistent storage (data remains even after container deletion)
* **Network:** Communication layer between containers

---

## Container Commands

### Run Container
```bash
docker run -d --name <name> <image>
```

### List Containers
```bash
docker ps       # (running)
docker ps -a    # (all)
```

### Stop Container
```bash
docker stop <container>
```

### Remove Container
```bash
docker rm <container>
docker rm -f <container> # (force remove)
```

---

## Debugging Commands

### View Logs
```bash
docker logs <container>
docker logs -f <container> # (live logs)
```

### Inspect Container
```bash
docker inspect <container>
```

### Resource Usage
```bash
docker stats
```

### Running Processes
```bash
docker top <container>
```

### Enter Container
```bash
docker exec -it <container> bash
```

---

## Volumes

### Create Volume
```bash
docker volume create <name>
```

### List Volumes
```bash
docker volume ls
```

### Inspect Volume
```bash
docker volume inspect <name>
```

### Use Volume
```bash
docker run -v <volume>:<path> <image>
```

### Mount Path (host side)
`/var/lib/docker/volumes/<volume>/_data`

---

## Networks

### List Networks
```bash
docker network ls
```

### Create Network
```bash
docker network create <name>
```

### Inspect Network
```bash
docker network inspect <name>
```

### Remove Network
```bash
docker network rm <name>
```

---

## Bulk Commands

### Stop all containers
```bash
docker stop $(docker ps -q)
```

### Remove all containers
```bash
docker rm $(docker ps -aq)
```

### Force remove all
```bash
docker rm -f $(docker ps -aq)
```

### Remove all images
```bash
docker rmi $(docker images -q)
```

---

## Important Differences
* **logs** → shows container output
* **inspect** → shows configuration and metadata
* **stats** → shows performance usage
* **top** → shows running processes

## Network Types
* **bridge** → default network
* **host** → no isolation
* **none** → no networking

---

## Apache Deployment Task
```bash
docker pull httpd
docker images
docker run -d --name apache-web -p 8081:80 httpd
docker ps
```
**Access in browser:**  
`http://localhost:8081`

---

## Key Mental Models
* **Container** → temporary compute
* **Volume** → permanent storage
* **Network** → communication layer

## Common Mistakes
* Using wrong IDs (container vs image)
* Forgetting `-d` (detached mode)
* Wrong port mapping (`:` instead of `-` or vice-versa)
* Trying to remove running containers without `-f`

## Debugging Flow
1. Enter container
2. Check logs
3. Inspect configuration
4. Check processes
