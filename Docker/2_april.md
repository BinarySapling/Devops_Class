Bhai mast 🔥 — aaj jo jo tu Docker me kiya hai usko mai **proper notes format + problem statements + answers** me convert kar raha hu. Tu directly isko apne notebook / exam me use kar sakta hai 💯

---

# 🐳 **Docker Practical Notes (Today’s Tasks)**

---

# 🔹 **1. Run a Container and Access It**

### ✅ Problem Statement

Run a container and access its terminal interactively.

### ✅ Command

```bash
docker run -dit --name myContainer ubuntu
docker exec -it myContainer bash
```

### ✅ Explanation

* `-d` → detached mode
* `-it` → interactive terminal
* `exec` → running container me enter karne ke liye

---

# 🔹 **2. Fix docker exec Error**

### ❌ Problem

```bash
docker exec -it bash container_id
```

### ❌ Error

```
No such container: bash
```

### ✅ Solution

```bash
docker exec -it <container_id> bash
```

### ✅ Key Learning

👉 Syntax:

```bash
docker exec -it CONTAINER COMMAND
```

---

# 🔹 **3. Create Directory and Files Inside Container**

### ✅ Problem Statement

Create a directory `/project`, add files, and write content.

### ✅ Commands

```bash
mkdir /project
cd /project

echo "This is my report file" > report.txt
echo "This is our container interaction with our host machine" > notes.txt
```

---

# 🔹 **4. Verify File Content**

### ✅ Problem Statement

Check the content of created files.

### ✅ Command

```bash
cat report.txt
cat notes.txt
```

---

# 🔹 **5. Copy File from Container to Local Machine**

### ✅ Problem Statement

Copy `notes.txt` from container to local system.

### ✅ Command

```bash
docker cp <container_id>:/project/notes.txt .
```

### ⚠️ Issue Faced

```bash
C:\Users\danis\Desktop ❌ not found
```

### ✅ Solution

```bash
docker cp <container_id>:/project/notes.txt .
```

👉 Then manually move file

---

# 🔹 **6. Fix Path Issue (Windows)**

### ❌ Problem

Desktop path not found

### ✅ Reason

Desktop shifted to OneDrive

### ✅ Fix

```bash
docker cp <container_id>:/project/notes.txt "C:\Users\danis\OneDrive\Desktop\"
```

---

# 🔹 **7. Append Data to File**

### ✅ Problem Statement

Append text into a file.

### ✅ Command

```bash
echo "New line" >> notes.txt
```

### ✅ Concept

* `>` → overwrite
* `>>` → append

---

# 🔹 **8. Check Running Containers**

### ✅ Command

```bash
docker ps
```

---

# 🔹 **9. Check Logs**

### ✅ Problem Statement

View logs of a running container.

### ✅ Command

```bash
docker logs -f myContainer
```

---

# 🔹 **10. Check Ports and Processes**

### ✅ Windows

```bash
netstat -ano
netstat -ano | findstr :8080
tasklist | findstr <PID>
```

### ✅ Linux

```bash
ss -tuln
sudo lsof -i :8080
```

---

# 🔹 **11. Nginx Container Run (Correct Way)**

### ❌ Wrong

```bash
docker run -it nginx bash
```

### ✅ Correct

```bash
docker run -d -p 8080:80 --name myNginx nginx
```

---

# 🔹 **12. Environment Variables**

### ❌ Problem

Add env variables to running container

### ❌ Not Possible

### ✅ Solution

```bash
docker run -d -e ENV_MODE=production nginx
```

---

# 🔹 **13. Docker Compose + .env (Best Practice)**

### `.env`

```env
ENV_MODE=production
PORT=8080
```

### `docker-compose.yml`

```yaml
services:
  app:
    image: nginx
    ports:
      - "${PORT}:80"
    environment:
      - ENV_MODE=${ENV_MODE}
```

### Run

```bash
docker-compose up -d
```

---

# 🔥 **MOST IMPORTANT CONCEPTS (REVISION)**

### 🧠 1. Docker Exec Syntax

```bash
docker exec -it <container> bash
```

---

### 🧠 2. Copy File

```bash
docker cp container:path .
```

---

### 🧠 3. Logs

```bash
docker logs -f container
```

---

### 🧠 4. Env Variables

👉 Only at container creation time

---

### 🧠 5. Debug Rule

```bash
docker ps
docker exec
docker logs
```

---

# 💯 **Interview Ready Summary**

👉 Docker containers are **isolated environments**
👉 Use `exec` to access
👉 Use `cp` to transfer files
👉 Use `logs` for debugging
👉 Containers are **immutable (env change = recreate)**

---

# 🔥 Bhai Final Advice

Tu aaj ye cover kar chuka hai:

* container lifecycle
* file operations
* debugging
* env handling

👉 Ye **DevOps basics clear ho gaye tere 💯**

---
