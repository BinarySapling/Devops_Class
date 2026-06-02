# Scenario 10: GitHub Container Registry Image Publish

## 1. Scenario

You are working on a project named `ghcr-publish-demo`. The task is to implement GitHub Actions for **GHCR publish**. This scenario focuses on **GitHub Container Registry** and helps students understand a real CI/CD use case through actual YAML implementation.

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
ghcr-publish-demo/
└── .github/
    └── workflows/
        └── ghcr-publish.yml
```

---


## Workflow File

```yaml
name: Publish Docker Image to GHCR

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  publish-ghcr:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Convert repository owner to lowercase
        run: echo "OWNER_LC=${GITHUB_REPOSITORY_OWNER,,}" >> $GITHUB_ENV

      - name: Build Docker image
        run: docker build -t ghcr.io/${{ env.OWNER_LC }}/node-ci-demo:latest .

      - name: Push Docker image to GHCR
        run: docker push ghcr.io/${{ env.OWNER_LC }}/node-ci-demo:latest
```

## Step-by-Step Implementation

1. Add Dockerfile.
2. Add workflow file.
3. Push to `main`.
4. Go to Actions and check workflow.
5. Check GitHub Packages after successful run.

## Expected Output

```text
Login Succeeded
Successfully built image
Pushed image to ghcr.io/owner/node-ci-demo:latest
```

## Explanation

GHCR means GitHub Container Registry. `GITHUB_TOKEN` is automatically available inside workflows. `packages: write` is required to push package images.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Missing `packages: write` | Push denied | Add permission |
| Uppercase owner name | Invalid image name | Convert to lowercase |
| Wrong registry URL | Login fails | Use `ghcr.io` |


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
