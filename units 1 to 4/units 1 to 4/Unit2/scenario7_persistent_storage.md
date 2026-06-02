# Scenario 7: Persistent Storage for Application Data

## 1. Problem Scenario

A containerized application stores user uploads. The problem is that containers are **temporary by nature**. If the container is deleted, everything stored **inside the container filesystem** is usually lost. A company wants the uploaded files to remain available even if:

- the container stops
- the container is removed
- a new container is created later

So the correct solution is to use **persistent storage** with either:

- a **Docker volume**, or
- a **bind mount**

For this classroom demonstration, the main recommended solution is a **Docker volume**, because it is easier, safer, and better for beginners.

---

## 2. Learning Goals

Students must be able to:

1. understand why container data is lost by default
2. create a Docker volume
3. attach the volume to a container
4. store data in the mounted path
5. delete the container
6. start a new container with the same volume
7. verify that the old data is still available

---

## 3. Core Concept in Very Simple Words

Think of a container like a **temporary rented room**.

- The application lives in the room
- Files written inside the room stay there only while that room exists
- If the room is demolished, the files inside are gone

A Docker volume is like a **safe storage locker outside the room**.

- The room can be removed
- the locker remains
- when a new room is created, the same locker can be attached again
- the files are still there

That is why Docker volumes are used for persistent data.

---

## 4. Why Data Is Lost Without a Volume

When you run a normal container, the writable layer belongs to that container only.

Example:

```bash
docker run --name tempapp alpine sh
```

If you create files inside that container and then delete the container, those files disappear with it.

Reason:

- container writable layer is not meant for long-term data
- it is tied to that specific container instance
- removing the container removes that writable layer too

This is why uploads, logs, user documents, and database files should not be stored only inside the container filesystem.

---

## 5. Best Practice Choice for This Scenario

For beginner-friendly and production-friendly persistence, use a **named Docker volume**.

Why this is the best default choice:

- Docker manages it automatically
- no need to remember host directory paths
- works well across container recreation
- cleaner than storing everything directly in random host folders
- safer for beginners than bind mounts

---

## 6. Project Demonstration Plan

We will create a very small Python application that:

- writes uploaded data into `/data`
- lists files stored in `/data`

Then we will:

1. build the image
2. create a Docker volume
3. mount the volume into `/data`
4. run the container
5. create a file
6. remove the container
7. create a new container using the same volume
8. verify that the file still exists

---

## 7. Folder Structure

Create a project folder like this:

```text
persistent-storage-demo/
│
├── app.py
└── Dockerfile
```

---

## 8. Application Code

Create a file named `app.py`.

```python
import os
from datetime import datetime

DATA_DIR = "/data"
FILE_PATH = os.path.join(DATA_DIR, "uploads.txt")

os.makedirs(DATA_DIR, exist_ok=True)

def write_sample_data():
    with open(FILE_PATH, "a", encoding="utf-8") as f:
        f.write(f"Sample upload stored at {datetime.now()}\n")
    print("Data written successfully.")

def read_data():
    if os.path.exists(FILE_PATH):
        print("\nStored file content:\n")
        with open(FILE_PATH, "r", encoding="utf-8") as f:
            print(f.read())
    else:
        print("No stored data found.")

print("Application started.")
write_sample_data()
read_data()
```

### Why this code is useful for demonstration

This code is intentionally very simple.

- `DATA_DIR = "/data"` means the application stores data in `/data`
- later we will attach a Docker volume to `/data`
- if persistence works correctly, the file in `/data` should remain available across container deletion

The line:

```python
os.makedirs(DATA_DIR, exist_ok=True)
```

creates the `/data` folder if it does not exist.

This is important because beginners often assume that folders appear automatically. In reality, the application should make sure the required path exists.

The code appends to `uploads.txt`, instead of overwriting it.

That is helpful because each new run adds one more line, making persistence easy to observe.

---

## 9. Dockerfile

Create a file named `Dockerfile`.

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

---

## 10. Detailed Reasoning for Every Dockerfile Line

### `FROM python:3.11-slim`

This tells Docker which base image to use.

Why this step is needed:

- a container image must start from some base
- our app is written in Python
- so we need a Python runtime already available inside the image

Why `slim` is used:

- it is smaller than the full Python image
- it reduces image size
- smaller images download and build faster

### `WORKDIR /app`

This sets `/app` as the working directory inside the container.

Why this is needed:

- future instructions run relative to `/app`
- it keeps files organized
- avoids confusion about where the code is located

### `COPY app.py .`

This copies the application file from the host machine into the image.

Why this is needed:

- without copying the code, the container image would not contain our application
- the dot (`.`) means copy into the current working directory, which is `/app`

### `CMD ["python", "app.py"]`

This defines the default command that runs when the container starts.

Why this is needed:

- containers need a main process
- here the main process is the Python application
- when the container starts, it automatically runs `python app.py`

---

## 11. Build the Image

Move into the project folder and run:

```bash
docker build -t persistentapp:v1 .
```

### Detailed reasoning

- `docker build` tells Docker to create an image
- `-t persistentapp:v1` gives the image a name and tag
- `.` means use the current folder as the build context

Why build is required:

- Docker cannot run the application until the image is created
- the image packages the runtime and code together

---

## 12. Create the Volume

Run:

```bash
docker volume create appdata
```

### Detailed reasoning

This creates a named Docker volume called `appdata`.

Why this is needed:

- the volume is the persistent storage area
- it exists independently of containers
- even if the container is deleted, the volume remains

You can think of this as preparing the external storage locker before connecting it to the application.

---

## 13. Check the Volume Exists

Run:

```bash
docker volume ls
```

You should see `appdata` in the list.

### Why this verification step matters

Beginners should always verify important setup steps.

Do not assume a command worked just because no obvious error appeared. Checking the volume list confirms that the volume has really been created.

---

## 14. Run the Container with the Volume Attached

Run:

```bash
docker run --name mypersistentcontainer -v appdata:/data persistentapp:v1
```

### Detailed reasoning for each part

- `docker run` starts a container
- `--name mypersistentcontainer` gives the container a readable name
- `-v appdata:/data` attaches the named volume `appdata` to the container path `/data`
- `persistentapp:v1` is the image to run

Why `-v appdata:/data` is the key part:

- the application writes files into `/data`
- Docker connects `/data` to the external volume `appdata`
- data written into `/data` is therefore stored in the volume, not only in the container layer

This is the most important step in the entire scenario.

---

## 15. What Should Happen on First Run

The application will:

1. start
2. create `/data/uploads.txt` if needed
3. append a timestamp line
4. print the file content

Expected output will look similar to this:

```text
Application started.
Data written successfully.

Stored file content:

Sample upload stored at 2026-03-21 10:15:30.123456
```

---

## 16. Remove the Container

Now delete the container:

```bash
docker rm mypersistentcontainer
```

If it is still running and needs force removal:

```bash
docker rm -f mypersistentcontainer
```

### Why this step matters

This step is used to prove persistence.

If data survives after the container is removed, then the storage design is working correctly.

If no volume had been used, data stored only in the container's writable layer would usually be lost here.

---

## 17. Create a New Container Using the Same Volume

Run:

```bash
docker run --name mypersistentcontainer2 -v appdata:/data persistentapp:v1
```

### Why this proves persistence

This is a new container with a different name.

Important idea:

- old container is gone
- new container is created fresh
- but the same volume is mounted again

So if the file from the previous run still appears, that proves the data lived in the volume, not in the deleted container.

---

## 18. Expected Output on the Second Run

You should now see two timestamp lines in `uploads.txt`, something like:

```text
Application started.
Data written successfully.

Stored file content:

Sample upload stored at 2026-03-21 10:15:30.123456
Sample upload stored at 2026-03-21 10:18:02.654321
```

### Why this is strong proof

This shows that:

- the first line remained from the old container session
- the second line was added by the new container
- therefore the data persisted across container deletion

---

## 19. Verify the Volume Is Mounted

Run:

```bash
docker inspect mypersistentcontainer2
```

Look for the `Mounts` section.

### Why this is useful

This shows Docker’s detailed container configuration, including mounted volumes.

This is useful in labs and viva because it provides formal proof that `/data` is backed by `appdata`.

---

## 20. Inspect the Volume Itself

Run:

```bash
docker volume inspect appdata
```

### Why this helps

This command shows details such as:

- volume name
- driver
- mountpoint on the host

This helps students understand that Docker volumes are real storage objects managed by Docker.

---

## 21. Alternative Approach: Bind Mount

A second valid solution is a bind mount.

Example:

```bash
docker run --name mybindcontainer -v "$(pwd)/hostdata:/data" persistentapp:v1
```

On Windows PowerShell, a similar command may be:

```powershell
docker run --name mybindcontainer -v ${PWD}\hostdata:/data persistentapp:v1
```

### What is the difference

With a bind mount:

- a specific host folder is connected directly to the container path
- files are visible immediately on the host filesystem

### Why bind mounts are useful

- easy to inspect files directly from the host
- useful during development
- useful when an exact host directory is required

### Why volumes are still preferred here

- simpler and cleaner for beginners
- less dependent on host path formatting
- fewer permission and path issues
- more portable across environments

---

## 22. Common Mistakes and Their Reasons

### Mistake 1: Writing data to a path that is not mounted

If the app writes to `/app/data` but the volume is mounted to `/data`, persistence will fail.

Reason:

- only the mounted path is persistent
- data written elsewhere stays inside container layer

### Mistake 2: Forgetting to create or attach the volume

If `-v appdata:/data` is missing, the app still runs, but data is stored only in the container layer.

### Mistake 3: Deleting the volume accidentally

If someone runs:

```bash
docker volume rm appdata
```

then the persistent data is removed.

### Mistake 4: Confusing image with volume

An image stores the application blueprint.
A volume stores changing runtime data.

These are different things and should not be mixed up.

---

## 23. Full Command List for Demonstration

```bash
mkdir persistent-storage-demo
cd persistent-storage-demo
docker build -t persistentapp:v1 .
docker volume create appdata
docker volume ls
docker run --name mypersistentcontainer -v appdata:/data persistentapp:v1
docker rm -f mypersistentcontainer
docker run --name mypersistentcontainer2 -v appdata:/data persistentapp:v1
docker inspect mypersistentcontainer2
docker volume inspect appdata
```

---

## 24. `docker history` Demonstration

Students can also inspect image layers:

```bash
docker history persistentapp:v1
```

### Why include this

Although the main topic is persistent storage, students are often required to inspect image history in Docker-related tasks.

This helps them see:

- base image layer
- copied application layer
- command layer

It reinforces the idea that the image contains the application, while the volume contains persistent runtime data.

---

## 25. Simple Viva Answers

### Q1. Why is a Docker volume needed?
A Docker volume is needed because data stored only inside the container writable layer may be lost when the container is deleted.

### Q2. Why is `/data` important in this example?
Because the volume is mounted to `/data`, and the application writes to `/data`, so the written files go into persistent storage.

### Q3. What happens if I remove the container but not the volume?
The container is deleted, but the data in the volume remains available.

### Q4. What is the difference between volume and bind mount?
A volume is managed by Docker, while a bind mount directly maps a host folder into the container.

---

## 26. Final Conclusion

The correct solution for persistent application data is to store uploads in a Docker volume or bind mount instead of the container's internal writable layer. In this demonstration, a named Docker volume is attached to `/data`, the application writes data into that mounted path, and the same data remains available even after the original container is removed and a new one is started. This proves true persistence.

---

## 27. Final Submission-Ready Answer

### Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

### Application Code

```python
import os
from datetime import datetime

DATA_DIR = "/data"
FILE_PATH = os.path.join(DATA_DIR, "uploads.txt")

os.makedirs(DATA_DIR, exist_ok=True)

def write_sample_data():
    with open(FILE_PATH, "a", encoding="utf-8") as f:
        f.write(f"Sample upload stored at {datetime.now()}\n")
    print("Data written successfully.")

def read_data():
    if os.path.exists(FILE_PATH):
        print("\nStored file content:\n")
        with open(FILE_PATH, "r", encoding="utf-8") as f:
            print(f.read())
    else:
        print("No stored data found.")

print("Application started.")
write_sample_data()
read_data()
```

### Commands

```bash
docker build -t persistentapp:v1 .
docker volume create appdata
docker run --name mypersistentcontainer -v appdata:/data persistentapp:v1
docker rm -f mypersistentcontainer
docker run --name mypersistentcontainer2 -v appdata:/data persistentapp:v1
docker volume inspect appdata
docker history persistentapp:v1
```
