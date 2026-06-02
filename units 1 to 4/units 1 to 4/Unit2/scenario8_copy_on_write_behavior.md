# Scenario 8: Copy-on-Write Behavior Investigation

## 1. Scenario Statement

A developer modifies files inside a running container and expects new containers created from the same image to automatically contain those changes.  
This expectation is incorrect in most normal Docker workflows.

Docker uses a **copy-on-write filesystem**. A running container gets its own writable layer on top of the image's read-only layers. Changes made inside one container normally stay only inside that container's writable layer. They do **not** automatically go back into the original image, and they do **not** automatically appear in brand new containers started later from the same image.

This experiment is designed to prove that behavior clearly.

---

## 2. Student Objectives

Students must demonstrate all of the following:

- Start a container from an image
- Modify files inside the running container
- Observe the modified result in that same container
- Start a new container from the same original image
- Compare the result and show that the new container does **not** contain the previous container's changes
- Explain why this happens using Docker's copy-on-write concept

---

## 3. Core Concept in Very Simple Words

Think of a Docker image like a **master photocopy**.

When Docker starts a container from that image:

- it does **not** change the master image
- it creates a **new writable layer** for that container
- any edits made inside the container go into that container's writable layer only

So:

- **Container A** can be changed
- **Container B** can start from the same image later
- **Container B** still starts from the original unchanged image state

That is the idea of copy-on-write in this experiment.

---

## 4. What Does "Copy-on-Write" Mean?

Copy-on-write means Docker does not duplicate the full image for every container at start time.  
Instead, Docker reuses the existing image layers as read-only layers and adds one thin writable layer for each container.

When a file is changed inside the container:

- Docker keeps the original image layer unchanged
- Docker records the changed version in the container's writable layer

This saves storage and improves efficiency.

---

## 5. Demo Design

We will create a tiny text-based application image.

Inside the image, a file called `message.txt` will contain:

```text
Original message from image
```

Then we will do the following experiment:

1. Build the image
2. Start **Container 1**
3. Read `message.txt`
4. Modify `message.txt` inside Container 1
5. Confirm the changed text inside Container 1
6. Stop or leave Container 1
7. Start **Container 2** from the **same image**
8. Read `message.txt` inside Container 2
9. Observe that Container 2 still shows the original message

This proves that changes in Container 1 were not written back into the image.

---

## 6. Project Folder Structure

Create a folder such as this:

```text
cow-demo/
├── Dockerfile
└── message.txt
```

Reason:

- `message.txt` is the file baked into the image
- `Dockerfile` defines how the image is created
- keeping the demo small helps beginners focus on the Docker concept instead of application complexity

---

## 7. Create the Initial File

Create `message.txt`:

```text
Original message from image
```

Reason:

- this is the baseline content
- later we will compare this original file against the modified file inside a running container

---

## 8. Create the Dockerfile

Create a file named `Dockerfile` with the following content:

```dockerfile
FROM alpine:3.20

WORKDIR /app

COPY message.txt /app/message.txt

CMD ["sh"]
```

---

## 9. Detailed Explanation of Each Dockerfile Line

### `FROM alpine:3.20`

This chooses a small Linux base image.

Reason:
- Alpine is lightweight
- fast to download
- perfect for teaching simple Docker concepts
- reduces unnecessary complexity

### `WORKDIR /app`

This sets the working directory inside the container.

Reason:
- keeps files organized
- future commands run from `/app`
- beginners can easily remember where the file is stored

### `COPY message.txt /app/message.txt`

This copies the file from the host machine into the image.

Reason:
- the file becomes part of the built image
- every new container started from this image will initially receive this same original file through the image layers

### `CMD ["sh"]`

This starts a shell when the container runs.

Reason:
- gives interactive access to inspect and edit files manually
- ideal for experiments and learning
- beginners can directly type commands inside the container

---

## 10. Build the Image

Run:

```bash
docker build -t cowdemo:v1 .
```

Reason:
- `docker build` creates an image from the Dockerfile
- `-t cowdemo:v1` gives the image a name and version tag
- `.` means use the current folder as the build context

Expected result:
- Docker builds the image
- the image contains `/app/message.txt`

---

## 11. Verify the Image Exists

Run:

```bash
docker images
```

Reason:
- confirms the image was built successfully
- helps students see the image name and tag

---

## 12. Start Container 1

Run:

```bash
docker run -it --name container1 cowdemo:v1
```

Reason:
- `docker run` starts a new container from the image
- `-it` gives an interactive terminal
- `--name container1` makes the container easy to identify
- `cowdemo:v1` is the source image

After this command, you will be inside the container shell.

---

## 13. Read the File in Container 1

Inside the container, run:

```sh
cat /app/message.txt
```

Expected output:

```text
Original message from image
```

Reason:
- this confirms the file exists
- this confirms the container started from the original image content

---

## 14. Modify the File in Container 1

Inside Container 1, run:

```sh
echo "Modified inside container1" > /app/message.txt
```

Reason:
- this overwrites the file content
- the change is stored in **container1's writable layer**
- the original image layer is still unchanged

---

## 15. Verify the Change in Container 1

Inside Container 1, run:

```sh
cat /app/message.txt
```

Expected output:

```text
Modified inside container1
```

Reason:
- this proves the file was changed successfully
- but the change exists only inside Container 1's writable layer

---

## 16. Leave Container 1

Type:

```sh
exit
```

Reason:
- exits the interactive shell
- container may stop after exit, but its writable layer still exists until the container is removed

---

## 17. Start Container 2 from the Same Image

Now start a completely new container:

```bash
docker run -it --name container2 cowdemo:v1
```

Reason:
- this creates a fresh new container
- Docker uses the same original image again
- Container 2 gets its **own separate writable layer**
- it does **not** inherit the writable layer from Container 1

---

## 18. Read the File in Container 2

Inside Container 2, run:

```sh
cat /app/message.txt
```

Expected output:

```text
Original message from image
```

This is the most important observation in the experiment.

Reason:
- Container 2 started from the same image
- the image still contains the original file
- changes made in Container 1 were never written back into the image

This proves Docker's copy-on-write behavior.

---

## 19. Compare the Two Containers

### In Container 1
`message.txt` became:

```text
Modified inside container1
```

### In Container 2
`message.txt` remains:

```text
Original message from image
```

Conclusion:
- the file change in Container 1 was local to Container 1
- the image stayed unchanged
- new containers start from the original image state

---

## 20. Why Beginners Get Confused

Many beginners assume:

> "If I changed the file in one container, Docker should remember that change for the next container."

This is incorrect because:

- a container is **not** the same thing as an image
- the image is the reusable blueprint
- the container is a running instance with its own writable layer

A change inside one container does not automatically become part of the image.

---

## 21. Optional Extra Check Using `docker diff`

Docker provides a command to inspect file changes made inside a container.

After modifying Container 1, from the host terminal run:

```bash
docker diff container1
```

Possible output might look like:

```text
C /app
C /app/message.txt
```

Meaning:
- `C` indicates a changed file or directory
- Docker is reporting that Container 1 has filesystem changes relative to the original image

Reason:
- this is additional proof that the change belongs to the container layer

---

## 22. Optional Extra Check Using `docker commit`

If students want to see how changes can be preserved intentionally, they can use:

```bash
docker commit container1 cowdemo:modified
```

Then run:

```bash
docker run -it --rm cowdemo:modified
```

Inside that new container:

```sh
cat /app/message.txt
```

Expected output:

```text
Modified inside container1
```

Reason:
- `docker commit` creates a **new image** from the modified container state
- this is different from normal automatic behavior
- it shows that changes become reusable only when explicitly turned into a new image

---

## 23. Important Conceptual Difference

### Image
- blueprint
- read-only layers
- reusable source for containers

### Container
- running instance of an image
- gets a private writable layer
- changes stay local unless intentionally saved

This distinction is essential.

---

## 24. Common Mistakes

### Mistake 1: Thinking a container change updates the image
It does not.

### Mistake 2: Confusing "same image" with "same container"
A new container from the same image is still a different container with a different writable layer.

### Mistake 3: Removing the original container before checking results
If you want to compare, inspect the behavior carefully before cleanup.

### Mistake 4: Expecting persistence without volumes or image rebuild
Container-local changes are not the right mechanism for durable shared application data.

---

## 25. Where This Matters in Real Projects

This concept matters when:

- developers edit files manually inside a container for testing
- teams expect changes to survive recreation automatically
- users confuse ephemeral container state with persistent storage
- application data needs durable storage

Correct real-world approaches are:

- rebuild the image properly
- use bind mounts during development
- use volumes for persistent application data

---

## 26. Full Command Sequence

### On the host
```bash
mkdir cow-demo
cd cow-demo
```

Create `message.txt`:
```text
Original message from image
```

Create `Dockerfile`:
```dockerfile
FROM alpine:3.20
WORKDIR /app
COPY message.txt /app/message.txt
CMD ["sh"]
```

Build image:
```bash
docker build -t cowdemo:v1 .
```

Run Container 1:
```bash
docker run -it --name container1 cowdemo:v1
```

### Inside Container 1
```sh
cat /app/message.txt
echo "Modified inside container1" > /app/message.txt
cat /app/message.txt
exit
```

### Back on the host
Optional:
```bash
docker diff container1
```

Run Container 2:
```bash
docker run -it --name container2 cowdemo:v1
```

### Inside Container 2
```sh
cat /app/message.txt
exit
```

Optional preservation:
```bash
docker commit container1 cowdemo:modified
docker run -it --rm cowdemo:modified
```

---

## 27. Expected Final Observation

- Container 1 shows the modified text
- Container 2 shows the original text
- therefore Docker uses a separate writable layer per container
- therefore container file modifications do not automatically appear in new containers

---

## 28. Viva-Style Answer

**Question:** Why did Container 2 not show the file modified in Container 1?

**Answer:**  
Because Docker gives each container its own writable layer on top of the image's read-only layers. The file was modified only inside Container 1's writable layer. The image was not changed, so Container 2 started from the original image content.

---

## 29. One-Line Conclusion

Docker's copy-on-write filesystem means that file changes inside one running container stay local to that container unless they are deliberately saved into a new image or stored outside the container using a volume or bind mount.
