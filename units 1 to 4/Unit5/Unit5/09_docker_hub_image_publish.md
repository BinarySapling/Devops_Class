# Scenario 09: Docker Hub Image Publish

## 1. Scenario

You are working on a project named `dockerhub-publish-demo`. The task is to implement GitHub Actions for **Docker Hub authentication and push**. This scenario focuses on **Docker Hub** and helps students understand a real CI/CD use case through actual YAML implementation.

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
dockerhub-publish-demo/
└── .github/
    └── workflows/
        └── dockerhub-publish.yml
```

---


## Required Secrets

Create these in GitHub:

```text
Settings → Secrets and variables → Actions → New repository secret
```

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

## Workflow File

```yaml
name: Docker Hub Publish

on:
  push:
    branches:
      - main

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build Docker image
        run: docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/node-ci-demo:latest .

      - name: Push Docker image to Docker Hub
        run: docker push ${{ secrets.DOCKERHUB_USERNAME }}/node-ci-demo:latest
```

## Step-by-Step Implementation

1. Create Docker Hub account.
2. Create Docker Hub access token.
3. Store username and token as GitHub Secrets.
4. Add Dockerfile to project.
5. Add workflow.
6. Push to `main`.
7. Check image on Docker Hub.

## Expected Output

```text
Login Succeeded
Successfully built image
Pushed image to Docker Hub
```

## Explanation

The workflow logs in securely using secrets. Credentials are never written directly in the YAML file.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Wrong token | Login fails | Regenerate Docker Hub token |
| Typing password in YAML | Security risk | Use GitHub Secrets |
| Wrong image name | Push denied | Use `username/image:tag` format |


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
