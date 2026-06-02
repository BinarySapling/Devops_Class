# Scenario 10 - Secure Image Distribution

## 1. Problem Statement

A company wants to distribute container images privately among internal teams.  
They do **not** want images to be publicly visible on the internet.  
They also want only authorized employees, CI/CD systems, or deployment servers to be able to pull or push images.

The company therefore needs a **secure image distribution workflow** using a private container registry such as:

- **Docker Hub (private repositories)**
- **GitHub Container Registry (GHCR)**

This solution explains the complete workflow in a beginner-friendly way, with detailed reasoning for each step.

---

## 2. Learning Objectives

Students must be able to:

- understand what a container registry is
- understand why authentication is needed
- log in to a private registry securely
- tag an image correctly for the target registry
- push an image to the registry
- verify that the push worked
- understand how internal teams later pull the same image
- understand why tokens are safer than passwords

---

## 3. Core Concept Before Implementation

### 3.1 What is a registry?

A **container registry** is like an online storage place for Docker images.

You can think of it as:

- GitHub for source code
- registry for container images

Instead of emailing image files to teammates, the image is pushed once to a registry and then pulled wherever needed.

### 3.2 Why private distribution is needed

If a company image is pushed to a public repository, anyone may download it. That is dangerous because images may contain:

- internal application code
- environment assumptions
- internal dependencies
- company-specific configuration
- metadata that should remain private

So companies prefer:

- **private repositories**
- **authentication**
- **role-based access**
- **tokens instead of passwords**

### 3.3 Why tags matter

Tags help identify versions of the same application image.

For example:

- `companyapp:v1`
- `companyapp:v2`
- `companyapp:prod`
- `companyapp:staging`

This helps teams deploy the correct version when needed.

---

## 4. Two Secure Distribution Options

This scenario can be implemented using either:

### Option A - Docker Hub private repository
Good when the company already uses Docker Hub and wants a simple hosted registry.

### Option B - GitHub Container Registry (GHCR)
Good when the company already uses GitHub for source code, automation, or access control.

Both support authentication tokens.

---

## 5. Recommended Security Principles

Before the steps, keep these rules in mind:

1. **Never use public repositories for private company images unless intentionally required.**
2. **Prefer access tokens over passwords.**
3. **Do not hardcode tokens in Dockerfiles.**
4. **Do not save tokens inside project files.**
5. **Use least-privilege permissions.**
6. **Use version tags clearly.**
7. **Verify repository visibility is private.**

---

## 6. Example Demo Application

To keep the demonstration simple, use a very small Python application.

### 6.1 Project Folder Structure

```text
secure-distribution-demo/
│
├── app.py
└── Dockerfile
```

### 6.2 app.py

```python
print("Secure image distribution demo is running")
```

### 6.3 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

CMD ["python", "app.py"]
```

---

## 7. Why This Dockerfile Is Used

### `FROM python:3.11-slim`
This selects a lightweight Python base image.

Reason:
- smaller image size
- faster upload
- faster pull by internal teams

### `WORKDIR /app`
This sets the working directory inside the container.

Reason:
- keeps files organized
- makes future instructions easier to read

### `COPY app.py .`
This copies the Python script into the image.

Reason:
- this is the actual application file the container will run

### `CMD ["python", "app.py"]`
This defines the default startup command.

Reason:
- when the container starts, the application runs automatically

---

## 8. Step-by-Step Workflow - Common First Steps

These steps are common for both Docker Hub and GHCR.

### Step 1 - Create the project directory

```bash
mkdir secure-distribution-demo
cd secure-distribution-demo
```

**Reasoning:**  
A separate project folder keeps the demo clean and avoids mixing files with unrelated projects.

### Step 2 - Create `app.py`

```python
print("Secure image distribution demo is running")
```

**Reasoning:**  
A tiny file is enough to demonstrate image build, tag, push, and pull.  
The goal here is registry workflow, not complex application logic.

### Step 3 - Create `Dockerfile`

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```

**Reasoning:**  
This gives us a real image that can be built and distributed.

### Step 4 - Build the image locally

```bash
docker build -t companyapp:v1 .
```

**Reasoning:**  
This creates the image on the local machine first.  
You cannot push an image to a registry until it exists locally.

### Step 5 - Verify the image exists

```bash
docker images
```

**Reasoning:**  
This confirms the image was built successfully before proceeding to authentication and push.

---

## 9. Option A - Secure Distribution Using Docker Hub Private Repository

### Step 1 - Create a private repository on Docker Hub

Create a repository such as:

```text
yourdockerhubusername/companyapp
```

Set repository visibility to **Private**.

**Reasoning:**  
A private repository ensures only authorized users can access the image.

### Step 2 - Create an access token in Docker Hub

Generate a Docker Hub access token from account settings.

**Reasoning:**  
Using a token is safer than typing your account password directly into CLI workflows, automation pipelines, or CI systems.

### Step 3 - Log in to Docker Hub

```bash
docker login
```

Docker will ask for:

- username
- password or token

If using token-based authentication, paste the token when prompted.

**Reasoning:**  
Authentication tells Docker who you are and what repositories you are allowed to access.

### Step 4 - Tag the local image for Docker Hub

```bash
docker tag companyapp:v1 yourdockerhubusername/companyapp:v1
```

**Reasoning:**  
Docker needs the full repository path to know where the image should be pushed.  
The tag format tells Docker:

- registry namespace/user = `yourdockerhubusername`
- repository name = `companyapp`
- tag/version = `v1`

### Step 5 - Push the image

```bash
docker push yourdockerhubusername/companyapp:v1
```

**Reasoning:**  
This uploads the local image layers to the private repository.  
Only new layers are transferred if some layers already exist remotely.

### Step 6 - Verify the push

Use either:

- Docker Hub web interface
- or pull from another machine

Example verification command:

```bash
docker pull yourdockerhubusername/companyapp:v1
```

**Reasoning:**  
Verification proves the image is actually available to internal users.

---

## 10. Option B - Secure Distribution Using GitHub Container Registry (GHCR)

### Step 1 - Ensure you have a GitHub account and a repository

Example repository owner:

```text
yourgithubusername
```

Image path example:

```text
ghcr.io/yourgithubusername/companyapp:v1
```

### Step 2 - Create a GitHub Personal Access Token (PAT)

Create a token with appropriate permissions such as:

- `write:packages`
- `read:packages`

Depending on the workflow, you may also need repository-related permissions.

**Reasoning:**  
GHCR uses GitHub authentication.  
The token proves your identity and controls what package actions are allowed.

### Step 3 - Log in to GHCR

```bash
docker login ghcr.io -u yourgithubusername
```

When prompted for password, paste your GitHub token.

**Reasoning:**  
This authenticates Docker to GitHub Container Registry.

### Step 4 - Tag the image for GHCR

```bash
docker tag companyapp:v1 ghcr.io/yourgithubusername/companyapp:v1
```

**Reasoning:**  
The image must be tagged with the GHCR registry path, otherwise Docker will not know the destination registry.

### Step 5 - Push the image

```bash
docker push ghcr.io/yourgithubusername/companyapp:v1
```

**Reasoning:**  
This uploads the image into GitHub Container Registry under your namespace.

### Step 6 - Verify the pushed image

On another authorized machine:

```bash
docker pull ghcr.io/yourgithubusername/companyapp:v1
```

**Reasoning:**  
If pull works, the image distribution pipeline is functioning correctly.

---

## 11. Preferred Secure Login Method Using `--password-stdin`

For better security, avoid pasting the token visibly into terminal history workflows.

### Docker Hub safer login

```bash
echo "YOUR_DOCKERHUB_TOKEN" | docker login -u yourdockerhubusername --password-stdin
```

### GHCR safer login

```bash
echo "YOUR_GITHUB_TOKEN" | docker login ghcr.io -u yourgithubusername --password-stdin
```

**Reasoning:**  
`--password-stdin` is safer because:
- it avoids exposing secrets in command history more than inline parameters
- it is easier to automate in scripts and CI/CD
- it reduces accidental leakage

> In real production systems, the token should come from a secure secret store or CI variable, not be typed literally into a script file.

---

## 12. Complete Command Flow - Docker Hub Example

```bash
mkdir secure-distribution-demo
cd secure-distribution-demo

docker build -t companyapp:v1 .

docker login

docker tag companyapp:v1 yourdockerhubusername/companyapp:v1

docker push yourdockerhubusername/companyapp:v1
```

### Verification

```bash
docker pull yourdockerhubusername/companyapp:v1
docker run --rm yourdockerhubusername/companyapp:v1
```

Expected output:

```text
Secure image distribution demo is running
```

---

## 13. Complete Command Flow - GHCR Example

```bash
mkdir secure-distribution-demo
cd secure-distribution-demo

docker build -t companyapp:v1 .

docker login ghcr.io -u yourgithubusername

docker tag companyapp:v1 ghcr.io/yourgithubusername/companyapp:v1

docker push ghcr.io/yourgithubusername/companyapp:v1
```

### Verification

```bash
docker pull ghcr.io/yourgithubusername/companyapp:v1
docker run --rm ghcr.io/yourgithubusername/companyapp:v1
```

Expected output:

```text
Secure image distribution demo is running
```

---

## 14. How Internal Teams Use the Image

Once the image is pushed privately, authorized internal teams can do:

### Docker Hub

```bash
docker login
docker pull yourdockerhubusername/companyapp:v1
docker run --rm yourdockerhubusername/companyapp:v1
```

### GHCR

```bash
docker login ghcr.io -u yourgithubusername
docker pull ghcr.io/yourgithubusername/companyapp:v1
docker run --rm ghcr.io/yourgithubusername/companyapp:v1
```

**Reasoning:**  
This creates a central delivery mechanism.  
Instead of manually copying builds, teams consume controlled versions from one trusted location.

---

## 15. Why Authentication Tokens Are Better Than Passwords

Tokens are preferred because:

- they can be revoked without changing the main password
- they can have limited scopes/permissions
- they are safer for automation
- they reduce the risk of exposing full account credentials

For example, a CI/CD server may need package push access but not full account access.

---

## 16. Tagging Strategy Recommendation

A company should not use only one tag forever.

Better examples:

```text
companyapp:v1
companyapp:v1.1
companyapp:v2
companyapp:staging
companyapp:prod
companyapp:latest
```

### Recommended rule

- use **version tags** for exact releases
- use **environment tags** carefully for deployment channels
- never rely only on `latest` in production

**Reasoning:**  
Version tags improve rollback, auditing, debugging, and reproducibility.

---

## 17. Security Mistakes to Avoid

### Mistake 1 - Using public repository accidentally
This exposes company images to outsiders.

### Mistake 2 - Putting tokens into Dockerfile
Tokens may become part of image history or leaked artifacts.

### Mistake 3 - Pushing with incorrect tag
If the image is not tagged correctly, push may fail or go to the wrong place.

### Mistake 4 - Giving excessive token permissions
Always prefer minimum permissions needed.

### Mistake 5 - Sharing one token among everyone
Use user-specific or system-specific credentials when possible.

### Mistake 6 - Depending only on `latest`
This makes rollback and version traceability difficult.

---

## 18. Troubleshooting Guide

### Problem: `denied: requested access to the resource is denied`
Possible reasons:
- wrong login
- wrong token
- repository name incorrect
- repository permissions missing

### Problem: `unauthorized`
Possible reasons:
- token expired
- token scopes insufficient
- registry path incorrect

### Problem: image push succeeds locally tagged, but not remotely
Possible reason:
- image was not tagged with full remote repository path

Example of correct remote tag:

```bash
docker tag companyapp:v1 yourdockerhubusername/companyapp:v1
```

or

```bash
docker tag companyapp:v1 ghcr.io/yourgithubusername/companyapp:v1
```

---

## 19. Beginner-Friendly Explanation of the Full Logic

The logic is very simple:

1. build the image locally
2. authenticate to a private registry
3. tag the image with the registry path
4. push the image
5. teammates authenticate and pull the image

This turns image sharing into a secure, repeatable, version-controlled workflow.

---

## 20. Viva-Style Questions and Answers

### Q1. Why is a private registry used?
**Answer:**  
A private registry is used to keep company container images accessible only to authorized internal users and systems.

### Q2. Why do we tag an image again before pushing?
**Answer:**  
Because the registry destination path must be included in the tag so Docker knows where to push the image.

### Q3. Why are tokens preferred over passwords?
**Answer:**  
Tokens are safer, revocable, and can have limited permissions.

### Q4. What happens if you push without correct authentication?
**Answer:**  
The registry rejects the push request with an authorization or access denied error.

### Q5. Why should production not depend only on `latest`?
**Answer:**  
Because exact reproducibility and rollback become difficult when a unique version tag is not used.

---

## 21. Final Recommended Workflow

### For classroom demonstration
Use one of these flows:

#### Docker Hub private repository
```bash
docker build -t companyapp:v1 .
docker login
docker tag companyapp:v1 yourdockerhubusername/companyapp:v1
docker push yourdockerhubusername/companyapp:v1
```

#### GHCR private package
```bash
docker build -t companyapp:v1 .
docker login ghcr.io -u yourgithubusername
docker tag companyapp:v1 ghcr.io/yourgithubusername/companyapp:v1
docker push ghcr.io/yourgithubusername/companyapp:v1
```

---

## 22. Final Conclusion

A secure image distribution workflow requires a private registry, correct tagging, and authenticated push/pull access.  
Docker Hub private repositories and GitHub Container Registry both provide hosted solutions for internal image sharing.  
By using tokens instead of passwords, applying meaningful version tags, and verifying access correctly, companies can distribute container images safely among internal teams.

---

## 23. Short Submission-Ready Conclusion

**Secure image distribution means building the image locally, authenticating to a private registry, tagging it with the correct registry path, and pushing it using token-based access so internal teams can pull only the authorized versions they need.**
