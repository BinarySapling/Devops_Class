# Scenario 08: Docker Image Build in CI

## 1. Scenario

You are working on a project named `docker-build-demo`. The task is to implement GitHub Actions for **Dockerfile build validation**. This scenario focuses on **docker build** and helps students understand a real CI/CD use case through actual YAML implementation.

---

## 2. Learning Objectives

After completing this scenario, students will be able to:

- Create a valid workflow under `.github/workflows/`.
- Identify the correct trigger for the given automation need.
- Configure jobs, runners, and steps.
- Use actions and shell commands correctly.
- Debug common workflow errors.

---

## 3. Required Tools

- GitHub account
- Git installed locally
- Code editor such as VS Code
- Basic command-line knowledge

---

## 4. Project Structure

```text
docker-build-demo/
└── .github/
    └── workflows/
        └── docker-build.yml
```

---


## Project Structure

```text
docker-build-demo/
├── Dockerfile
├── package.json
├── index.js
└── .github/
    └── workflows/
        └── docker-build.yml
```

## Sample Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["node", "index.js"]
```

## Workflow File

```yaml
name: Docker Build CI

on:
  push:
  pull_request:

jobs:
  docker-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t my-node-app:ci .

      - name: Show Docker images
        run: docker images
```

## Step-by-Step Implementation

1. Add a `Dockerfile` to your project.
2. Add the workflow file.
3. Push code.
4. Check if the Docker build succeeds.

## Expected Output

```text
Successfully built image
my-node-app   ci
```

## Explanation

This workflow verifies that the Dockerfile is correct. It does not push the image to a registry.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Missing Dockerfile | Build fails | Add Dockerfile |
| Wrong `COPY` path | Files not found | Check project structure |
| Large build context | Slow build | Add `.dockerignore` |


---

## Student Submission Checklist

Students should submit:

1. GitHub repository link.
2. Screenshot of workflow YAML file.
3. Screenshot of successful workflow run.
4. Short explanation of trigger, job, runner, and steps.
5. Error encountered and how it was fixed.

---

## Viva Questions

1. What is the purpose of this workflow?
2. Which trigger is used and why?
3. Which runner is used in this scenario?
4. What are the key steps in the workflow?
5. What common error can occur in this scenario?
