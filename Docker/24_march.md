# Docker Notes (Run Commands + Flags + Environment Variables)

## Core Concepts (First Principles)
*   **Image:** blueprint
*   **Container:** running process
*   **Docker:** environment to run containers

`docker run = image + configuration + command`

## Basic Commands
```bash
docker --version
docker info
```

## Image Commands
```bash
docker pull httpd
docker images
docker history httpd
```

## Container Basics
```bash
docker run ubuntu
```
*Problem: container exits immediately if no process is running.*

## Important Flags

**Detached mode (`-d`)**
```bash
docker run -d httpd
```
Runs container in background

**Interactive mode (`-it`)**
```bash
docker run -it ubuntu bash
```
Opens terminal inside container

**Name container (`--name`)**
```bash
docker run --name my-container httpd
```

**Auto remove (`--rm`)**
```bash
docker run --rm ubuntu
```

**Port mapping (`-p`)**
```bash
docker run -d -p 8080:80 httpd
```
Host port `8080` → container port `80`
*Open in browser:* `http://localhost:8080`

**Environment variables (`-e`)**
```bash
docker run -e MY_VAR=value ubuntu env
```

**Volume mount (`-v`)**
```bash
docker run -v /host/path:/container/path ubuntu
```
Used for data persistence

## Running Commands Inside Container

**Wrong way:**
```bash
docker run httpd echo "hello world"
```
*(This only prints output)*

**Correct way:**
```bash
docker run ubuntu sh -c 'echo "hello" > file.txt'
```

## Environment Variables

**Without `-e`**
```bash
docker run ubuntu env
```

**With `-e`**
```bash
docker run -e MY_VAR=value ubuntu env
```

**Multiple variables**
```bash
docker run -e APP_ENV=production -e APP_VERSION=1.0 nginx
```

**Interactive example**
```bash
docker run -it -e MY_NAME=aastha ubuntu bash
```
Inside container:
```bash
echo $MY_NAME
```

## Passing Env Variables from Host

**Method 1**
```bash
export MY_VAR=hello
docker run -e MY_VAR=$MY_VAR ubuntu env
```

**Method 2**
```bash
export MY_VAR=hello
docker run -e MY_VAR ubuntu env
```

## Combined Example
```bash
docker run -dit --name webapp -p 8080:80 -e APP_ENV=production httpd
```

## Common Mistakes
*   **Wrong command:** `docker run ununtu`
    *   *Correct →* `docker run ubuntu`
*   **Wrong env syntax:** `-e MY VAR=value`
    *   *Correct →* `-e MY_VAR=value`
*   **Wrong usage:** `$docker --version`
    *   *Correct →* `docker --version`

## Mental Model
Before running a container, think:
1.  Do I need an image or container?
2.  Run in background or foreground?
3.  Need terminal access?
4.  Need port mapping?
5.  Need environment variables?

## Practice

```bash
docker pull httpd
docker run -d -p 8080:80 httpd
docker ps
```
*Open browser:* `http://localhost:8080`

**Key Takeaway**
`docker run = create + configure + start container`

## Checkpoint
```bash
docker run -d -p 8080:80 -e ENV=prod httpd
```
**Flow:**
Image → container created → port mapped → env set → process started