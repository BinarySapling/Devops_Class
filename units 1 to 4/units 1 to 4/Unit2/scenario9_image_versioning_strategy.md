
# Scenario 9: Image Versioning Strategy

## 1. Scenario in simple words

A company releases the same application again and again over time.  
For example:

- first release = version 1
- improved release = version 2
- bug fix release = version 2.1
- testing release = beta version
- stable release = production version

Now the company has a practical problem:

- developers want to deploy a **specific version**
- testers want to check an **older version**
- production team wants only the **approved stable version**
- rollback should be possible if the latest version fails

If Docker images are not tagged properly, then all versions become confusing.  
People may accidentally run the wrong application version.

So the company needs a **Docker image tagging and versioning strategy**.

---

## 2. Objective of this solution

The goal is to design a beginner-friendly and industry-relevant Docker versioning strategy so that students can:

- build Docker images with version tags
- understand why tags matter
- create multiple versions of the same application
- run a specific version when required
- compare different versions
- understand the difference between an image **name** and an image **tag**
- understand why `latest` alone is risky

---

## 3. What students must demonstrate

Students must show:

- how to build Docker images with version tags
- how to create another tag for the same image
- how to list Docker images
- how to run a specific version
- how to inspect image history
- how to understand and explain version control of container images

Example commands:

```bash
docker build -t companyapp:v1 .
docker tag companyapp:v1 companyapp:v2
```

---

## 4. Core concept: what is a Docker image tag?

A Docker image is usually written like this:

```bash
repository_name:tag
```

Example:

```bash
companyapp:v1
```

Here:

- `companyapp` = image name or repository name
- `v1` = tag

The **tag** is like a label attached to the image so that humans and systems know which version it is.

You can think of it like this:

- `companyapp:v1` -> first release
- `companyapp:v2` -> second release
- `companyapp:latest` -> most recent default version
- `companyapp:prod` -> approved production version
- `companyapp:test` -> testing version

So a tag helps identify **which image version should be run**.

---

## 5. Why image versioning is important

Without proper tagging, many real problems happen:

### Problem 1: wrong deployment
A team may accidentally deploy a test build into production.

### Problem 2: no rollback
If the newest version fails, the team cannot easily go back to the old working version.

### Problem 3: confusion between teams
One developer may say “run the app” but another may run the wrong build.

### Problem 4: poor traceability
Nobody knows which image corresponds to which release.

### Problem 5: unstable `latest`
If a team always uses `latest`, the behavior may suddenly change.

So Docker versioning is not just naming convenience.  
It is an important **release-management discipline**.

---

## 6. Real-world versioning strategy

A simple and useful tagging strategy is:

- `v1` -> first release
- `v2` -> second release
- `v1.1` -> minor update
- `dev` -> developer testing version
- `qa` -> quality testing version
- `prod` -> production-approved version
- `latest` -> newest default version

A company may combine these strategies depending on workflow.

### Good practice
Use at least one **explicit version tag**, such as:

- `v1`
- `v1.0`
- `v2.3.1`

### Better practice
Also add environment-oriented tags when useful:

- `companyapp:v2.3.1`
- `companyapp:prod`
- `companyapp:stable`

---

## 7. Beginner-friendly experiment design

We will create a very small Python application and then create multiple versions of it.

This keeps the focus on Docker versioning instead of application complexity.

### Project structure

```text
companyapp/
|-- app.py
|-- Dockerfile
```

---

## 8. Step 1: create a project folder

Open terminal or command prompt and run:

```bash
mkdir companyapp
cd companyapp
```

### Reasoning
This creates a separate working folder for the project.

Why this matters:

- keeps files organized
- avoids confusion with unrelated files
- makes Docker build context clean
- helps beginners clearly see what files are part of the image

A Docker build usually uses the current directory as build context, so keeping a neat folder is always a good habit.

---

## 9. Step 2: create the first application version

Create a file named `app.py` with the following content:

```python
print("Company App - Version 1")
```

### Reasoning
This is our application code.

Why use a very small Python file?

- easier for beginners
- no dependency complexity
- quick to build and run
- very clear output for version comparison

The purpose here is not to build a large application.  
The purpose is to **observe version tags clearly**.

---

## 10. Step 3: create the Dockerfile

Create a file named `Dockerfile` with the following content:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

---

## 11. Detailed explanation of the Dockerfile

### `FROM python:3.11-slim`
This tells Docker which base image to start from.

Why this is needed:

- a container needs a filesystem and runtime
- Python application needs Python installed
- `python:3.11-slim` is smaller than the full Python image
- smaller image means faster pull, build, and storage efficiency

### `WORKDIR /app`
This sets the working directory inside the container.

Why this is useful:

- all next instructions operate from `/app`
- keeps files organized
- avoids writing long paths repeatedly

### `COPY app.py .`
This copies the local `app.py` file into the container working directory.

Why this is needed:

- the container cannot run local files automatically
- application code must be placed inside the image

### `CMD ["python", "app.py"]`
This defines the default command that runs when the container starts.

Why this is needed:

- makes the container automatically execute the application
- user does not need to manually start Python inside the container

---

## 12. Step 4: build version 1 image

Run:

```bash
docker build -t companyapp:v1 .
```

### Meaning of the command

- `docker build` -> builds a Docker image
- `-t` -> assigns a name and tag
- `companyapp:v1` -> image name is `companyapp`, tag is `v1`
- `.` -> current directory is the build context

### Reasoning
This creates the first versioned image.

This is important because we are not building an unnamed image.  
We are creating a **specific, identifiable release**.

---

## 13. Step 5: verify the image exists

Run:

```bash
docker images
```

You should see an entry similar to this:

```text
REPOSITORY   TAG   IMAGE ID       CREATED          SIZE
companyapp   v1    abcdef123456   few seconds ago  ...
```

### Reasoning
This confirms that the image was created successfully.

Why this matters:

- verifies image name and tag
- helps students understand how Docker stores local images
- avoids confusion before moving to next steps

---

## 14. Step 6: run version 1 container

Run:

```bash
docker run --rm companyapp:v1
```

Expected output:

```text
Company App - Version 1
```

### Reasoning
This proves the image actually works.

Why use `--rm`?

- container is removed automatically after it stops
- avoids unused stopped containers
- keeps lab environment clean

This step is important because versioning is meaningful only if each tagged image can actually run correctly.

---

## 15. Step 7: inspect image history

Run:

```bash
docker history companyapp:v1
```

### Reasoning
This shows the layers used to create the image.

Students should learn that:

- an image is made of layers
- each Dockerfile instruction contributes to image creation
- tags identify versions, but history helps inspect composition

This is useful for both learning and debugging.

---

## 16. Step 8: create version 2 of the application

Now update `app.py` to:

```python
print("Company App - Version 2")
```

### Reasoning
We are simulating a new release of the application.

In real life this may represent:

- new feature added
- bug fixed
- UI changed
- performance improved

For our lab, the changed output makes the version difference obvious.

---

## 17. Step 9: build version 2 image directly

Run:

```bash
docker build -t companyapp:v2 .
```

### Reasoning
This creates a second image version.

Now the company has two separate deployable releases:

- `companyapp:v1`
- `companyapp:v2`

This is the heart of image versioning.

Now anyone can choose exactly which release to run.

---

## 18. Step 10: list images again

Run:

```bash
docker images
```

You may now see:

```text
REPOSITORY   TAG   IMAGE ID       CREATED          SIZE
companyapp   v1    abcdef123456   earlier          ...
companyapp   v2    123456abcdef   just now         ...
```

### Reasoning
This allows students to compare versions.

It teaches an important idea:

- same repository name
- different tags
- different image identities

This is how multiple releases can coexist on the same machine.

---

## 19. Step 11: run version 1 and version 2 separately

Run version 1:

```bash
docker run --rm companyapp:v1
```

Expected:

```text
Company App - Version 1
```

Run version 2:

```bash
docker run --rm companyapp:v2
```

Expected:

```text
Company App - Version 2
```

### Reasoning
This is one of the most important demonstrations.

Students clearly see that:

- both images exist separately
- each tag points to its own version
- deployment can target a precise version

This is exactly why versioning is useful in companies.

---

## 20. Step 12: understand `docker tag`

Now let us discuss this command:

```bash
docker tag companyapp:v1 companyapp:v2
```

This command does **not** rebuild the image.

It simply creates another tag pointing to an existing image.

### Very important understanding
`docker tag` is like giving another label to an already existing image.

It does not change application code.  
It does not generate a new image layer.  
It does not compile anything.

It only creates another reference.

---

## 21. Step 13: demonstrate tagging the same image with another label

Suppose version 1 is currently approved for production.

Run:

```bash
docker tag companyapp:v1 companyapp:prod
```

Now run:

```bash
docker images
```

You may see:

```text
REPOSITORY   TAG    IMAGE ID       CREATED          SIZE
companyapp   v1     abcdef123456   earlier          ...
companyapp   prod   abcdef123456   earlier          ...
companyapp   v2     123456abcdef   just now         ...
```

### Reasoning
Notice that `v1` and `prod` may have the same image ID.

That means:

- they are two tags
- but both point to the same image data

This is very useful.

Example industry use:

- `v1` = technical release label
- `prod` = business-approved production label

Both can refer to the same image.

---

## 22. Step 14: run the production-tagged image

Run:

```bash
docker run --rm companyapp:prod
```

Expected output:

```text
Company App - Version 1
```

### Reasoning
This proves that `prod` is just another tag for the same image.

This helps beginners understand that tags are references, not separate rewritten code.

---

## 23. Step 15: add the `latest` tag carefully

If version 2 is considered the newest release, tag it as latest:

```bash
docker tag companyapp:v2 companyapp:latest
```

Run:

```bash
docker images
```

### Reasoning
This creates a default-looking tag.

However, students must understand:

- `latest` does not mean “most advanced” automatically
- Docker uses whatever image was explicitly tagged as `latest`
- if tagging is careless, `latest` can become misleading

So in real projects, teams should not depend only on `latest`.

---

## 24. Why relying only on `latest` is risky

If someone runs:

```bash
docker run --rm companyapp:latest
```

they may not know exactly which release they are getting.

This can cause problems such as:

- hidden version mismatch
- production inconsistency
- rollback difficulty
- confusion in multi-team environments

### Good practice
Always keep explicit version tags such as:

- `v1`
- `v2`
- `v2.1.3`

Use `latest` only as an extra convenience tag, not as the only release identifier.

---

## 25. Recommended company versioning policy

A practical strategy for students to remember:

### Rule 1: every release gets a unique version tag
Examples:

- `v1`
- `v1.1`
- `v2`
- `v2.0.1`

### Rule 2: production image should also get a meaningful alias
Examples:

- `prod`
- `stable`

### Rule 3: do not overwrite old release tags carelessly
Old versions should remain available for rollback.

### Rule 4: use `latest` only as a convenience tag
Not as the only trusted reference.

### Rule 5: choose consistent naming
Do not mix random formats like:

- `version1`
- `release_final`
- `newbuild`
- `appv2new`

Instead use a clean structure.

---

## 26. Complete command sequence for the lab

### Build version 1
```bash
docker build -t companyapp:v1 .
```

### Run version 1
```bash
docker run --rm companyapp:v1
```

### Check history
```bash
docker history companyapp:v1
```

### Modify app.py to version 2 and rebuild
```bash
docker build -t companyapp:v2 .
```

### Run version 2
```bash
docker run --rm companyapp:v2
```

### Tag version 1 as production
```bash
docker tag companyapp:v1 companyapp:prod
```

### Tag version 2 as latest
```bash
docker tag companyapp:v2 companyapp:latest
```

### List images
```bash
docker images
```

### Run production tag
```bash
docker run --rm companyapp:prod
```

### Run latest tag
```bash
docker run --rm companyapp:latest
```

---

## 27. Tiny-detail explanation of what happens internally

When students build `companyapp:v1`, Docker creates an image and stores it locally.

When students later build `companyapp:v2`, Docker creates another image based on the modified application code.

When students use `docker tag`, Docker does not copy the full image content again.  
It simply creates another name that points to the already existing image.

That means tagging is lightweight and fast.

This is why one image can have multiple tags.

---

## 28. Important difference: build vs tag

### `docker build -t companyapp:v1 .`
This creates an image from source files and Dockerfile.

### `docker tag companyapp:v1 companyapp:prod`
This does not build anything new.  
It only assigns another tag to the existing image.

Students must understand this difference very clearly.

---

## 29. Industry-friendly example strategy

A company may use this pattern:

- `companyapp:1.0.0` -> exact release version
- `companyapp:1.0` -> broader minor family
- `companyapp:stable` -> approved stable release
- `companyapp:prod` -> deployed production release
- `companyapp:latest` -> newest default tag
- `companyapp:dev` -> development build

This allows the company to support:

- exact deployment
- testing
- rollback
- environment-specific workflows

---

## 30. Example rollback explanation

Suppose the production team deployed `companyapp:v2` and found a serious bug.

Because the company still has `companyapp:v1`, they can quickly run or redeploy the older stable image.

Example:

```bash
docker run --rm companyapp:v1
```

or in deployment workflow they can point systems back to `v1`.

### Reasoning
This is one of the biggest benefits of Docker image versioning.

Without version tags, rollback becomes messy and unreliable.

---

## 31. Common mistakes students should avoid

### Mistake 1: using only `latest`
This causes ambiguity.

### Mistake 2: overwriting tags without a plan
This destroys clarity.

### Mistake 3: forgetting to test each version
A tag is useful only if the image actually runs correctly.

### Mistake 4: assuming `docker tag` creates a new build
It does not. It only creates another reference.

### Mistake 5: keeping inconsistent naming styles
This makes automation and deployment harder.

---

## 32. Verification checklist

Students should verify all the following:

- Dockerfile created correctly
- version 1 image built successfully
- version 2 image built successfully
- `docker images` shows both tags
- `docker run companyapp:v1` shows version 1 output
- `docker run companyapp:v2` shows version 2 output
- `docker tag` creates a secondary label such as `prod` or `latest`
- `docker history` runs correctly

---

## 33. Sample observation table

| Command | Purpose | Expected Result |
|---|---|---|
| `docker build -t companyapp:v1 .` | Build first release | Image `companyapp:v1` created |
| `docker run --rm companyapp:v1` | Run first release | Output shows Version 1 |
| `docker build -t companyapp:v2 .` | Build second release | Image `companyapp:v2` created |
| `docker run --rm companyapp:v2` | Run second release | Output shows Version 2 |
| `docker tag companyapp:v1 companyapp:prod` | Give same image another label | `prod` tag points to version 1 image |
| `docker history companyapp:v1` | Inspect image history | Shows image layers |

---

## 34. Viva-ready explanation

If examiner asks: **Why do we use image tags?**

A good answer is:

> We use image tags to identify and manage different versions of the same Docker image. Tags help teams deploy exact releases, maintain rollback capability, and avoid confusion between development, testing, and production versions.

If examiner asks: **What is the difference between `docker build` and `docker tag`?**

A good answer is:

> `docker build` creates a new image from source files and Dockerfile, while `docker tag` only creates another label pointing to an existing image. It does not rebuild the application.

If examiner asks: **Why is `latest` risky?**

A good answer is:

> `latest` is only a label and may not always represent the intended production version. If teams rely only on `latest`, they may accidentally run the wrong release. Explicit version tags are safer.

---

## 35. Final Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

---

## 36. Final code examples

### `app.py` for version 1
```python
print("Company App - Version 1")
```

### `app.py` for version 2
```python
print("Company App - Version 2")
```

---

## 37. Final conclusion

A company can manage multiple application releases safely by using Docker image tags.  
The best practice is to build images with explicit version numbers such as `v1`, `v2`, or `v2.1`, and optionally assign additional tags like `prod`, `stable`, or `latest` when needed. This makes deployments precise, rollback easy, and collaboration much safer.

---

## 38. One-line summary

**Docker image versioning makes deployment controlled, rollback possible, and release management much more reliable.**
