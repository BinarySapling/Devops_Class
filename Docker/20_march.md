# Docker Apache (httpd) Practice Guide

## Core Mental Model
**Image → Container → Port Mapping → Browser**
*   **Image:** blueprint
*   **Container:** running instance
*   **Port mapping:** connection between system and container

## Pull Apache Image
```bash
docker pull httpd
```

**Verify**
```bash
docker images httpd
```

## Run Container
```bash
docker run -d -p 8081:80 --name task3 httpd
```
**Breakdown:**
*   `-d` → run in background
*   `-p 8081:80` → maps `localhost:8081` to container port `80`
*   `--name task3` → custom container name

## Check Containers
```bash
docker ps       # (running)
docker ps -a    # (all)
```

## Access in Browser
`http://localhost:8081`

## Modify Content (Method 1: docker exec)

**Enter container**
```bash
docker exec -it task3 /bin/bash
```

**Go to folder**
```bash
cd /usr/local/apache2/htdocs/
```

**Update file**
```bash
echo "<h1>Hello Aastha</h1>" > index.html
```

**Exit**
```bash
exit
```

## Modify Content (Method 2: Volume Mount)

**Remove old container**
```bash
docker rm -f task3
```

**Create folder and file**
```bash
mkdir mysite
cd mysite
echo "<h1>From Local Folder</h1>" > index.html
```

**Run with volume**
```bash
docker run -d -p 8081:80 --name task3 -v $(pwd):/usr/local/apache2/htdocs/ httpd
```

## Important Concepts
*   Apache serves default file: `index.html`
*   **Container immutability:**
    *   Container → temporary
    *   Image → permanent

## Common Errors
*   **Name conflict:** container name already in use
    *   *Fix:* `docker rm -f task3`
*   **Image not found:**
    *   *Fix:* `docker pull httpd`
*   **`vi` not found:**
    *   *Fix:* use `echo` command
*   **Wrong place for URL:**
    *   Do not run URL in terminal, open in browser

## DevOps Mindset
*Do not guess, always verify.*

**Commands:**
```bash
docker images
docker ps
docker logs <container>
```

## Full Practice Flow
```bash
docker rm -f task3
docker pull httpd
docker run -d -p 8081:80 --name task3 httpd
docker ps

docker exec -it task3 /bin/bash
cd /usr/local/apache2/htdocs/
echo "<h1>Practice</h1>" > index.html
exit
```