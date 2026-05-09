# Practical Laboratory Manual: DevOps Engineering (GitHub Actions)
## Target Audience: University Practical Exams, Lab Record Submissions, & DevOps Engineers

This laboratory manual provides step-by-step practical experiments for mastering GitHub Actions. Each experiment is structured with a clear **Objective**, **Prerequisites**, **Source Code (YAML)**, **Execution Steps**, **Verification Methods**, and **Troubleshooting Guidelines** to serve as a complete lab record or study guide.

---

## 🧪 Table of Laboratory Experiments

1. **Lab 1**: Establishing a Foundational "Hello World" Pipeline (Directory Structure & Shell Executions)
2. **Lab 2**: Event Filtering and Path-Based Triggers (Fine-Grained Git Webhooks)
3. **Lab 3**: Manual Deployments with Parameterized Inputs (`workflow_dispatch`)
4. **Lab 4**: Multi-Job Directed Acyclic Graph (DAG) & Build Artifact Transfers
5. **Lab 5**: Multi-OS Concurrency Grid (Matrix Strategy) & Package Caching (`actions/cache`)
6. **Lab 6**: Enterprise Containerized CI/CD: Building, Caching, Pushing to GHCR & Remote Staging Deployment via SSH

---

## 📘 Experiment 1: Foundational "Hello World" Pipeline

### 1. Objective
To understand the standard directory layout for GitHub Actions, write a basic YAML workflow file, trigger a job using a push event, and execute sequential shell commands on a hosted runner VM.

### 2. Prerequisites
* A GitHub account.
* A local git repository connected to a remote GitHub repository.

### 3. Repository Setup
Create the required directory structure:
```bash
mkdir -p .github/workflows
```

### 4. Workflow Configuration (`.github/workflows/01-hello-world.yml`)
```yaml
name: Lab 1 - Hello World Pipeline

on:
  push:
    branches: [main]

jobs:
  say-hello:
    name: Hello Actions Job
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Print Message
        run: echo "Hello, GitHub Actions! Pipeline triggered successfully."

      - name: Inspect Runner Environment
        run: |
          echo "Current User: $(whoami)"
          echo "Operating System: $(uname -a)"
          echo "Workspace Directory: $GITHUB_WORKSPACE"
```

### 5. Execution & Verification Steps
1. Commit the file: `git add .github/workflows/01-hello-world.yml && git commit -m "feat: add lab 1"`.
2. Push changes: `git push origin main`.
3. Open your GitHub Repository in a web browser.
4. Click on the **Actions** tab.
5. Click on the active run "Lab 1 - Hello World Pipeline".
6. Open the "Hello Actions Job" and verify that all steps completed successfully, checking the printed environment variables in the console logs.

---

## 📘 Experiment 2: Event Filtering and Path-Based Triggers

### 1. Objective
To configure advanced Git event filters to control workflow runs. The pipeline must trigger *only* when files inside the `src/` directory are modified, and ignore changes within the `docs/` folder to prevent unnecessary consumption of build minutes.

### 2. Workflow Configuration (`.github/workflows/02-event-filtering.yml`)
```yaml
name: Lab 2 - Path Event Filtering

on:
  push:
    branches:
      - main
    paths:
      - 'src/**'       # Trigger only on files modified under src/
      - '!docs/**'     # Exclude any changes under docs/

jobs:
  run-src-validation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Verify Trigger
        run: echo "Workflow triggered successfully. Source code change detected under src/ directory."
```

### 3. Verification Lab Steps
1. Create two directories: `src/` and `docs/` in your repository.
2. Commit and push a new markdown file under `docs/test.md`. Verify in the GitHub Actions tab that **no pipeline execution is triggered**.
3. Create and commit a dummy file under `src/app.js`. Push the changes.
4. Verify that the **"Lab 2 - Path Event Filtering" workflow triggers immediately** and executes successfully.

---

## 📘 Experiment 3: Manual Deployments with Parameterized Inputs

### 1. Objective
To build a manual deployment pipeline that does not run automatically on code push. The workflow must accept parameterized inputs (text version tags and choice-based deployment target environments) from the user via the GitHub Web UI.

### 2. Workflow Configuration (`.github/workflows/03-manual-dispatch.yml`)
```yaml
name: Lab 3 - Manual Release Automation

on:
  workflow_dispatch:
    inputs:
      release_version:
        description: 'Semantic Version (e.g., v1.0.0)'
        required: true
        type: string
        default: 'v1.0.0'
      target_env:
        description: 'Target Deployment Environment'
        required: true
        type: choice
        options:
          - Development
          - Staging
          - Production
      run_migrations:
        description: 'Execute database schema migration?'
        required: true
        type: boolean
        default: false

jobs:
  manual-release:
    runs-on: ubuntu-latest
    steps:
      - name: Read Input Parameters
        run: |
          echo "==== MANUAL RELEASE PARAMETERS ===="
          echo "Release Version: ${{ inputs.release_version }}"
          echo "Target Environment: ${{ inputs.target_env }}"
          echo "Run Migrations?: ${{ inputs.run_migrations }}"

      - name: Database Migration Step
        if: ${{ inputs.run_migrations == true }}
        run: echo "Executing database schema migrations..."
```

### 3. Execution & Verification Steps
1. Push the file to the **default branch (main)** of the repository.
2. Navigate to the **Actions** tab in the GitHub web interface.
3. Select **"Lab 3 - Manual Release Automation"** from the left sidebar.
4. Locate the blue drop-down button labeled **"Run workflow"** on the right.
5. Enter a version tag (e.g., `v2.4.0`), select `Staging`, and toggle the database migration checkbox to `true`.
6. Click **Run workflow**, open the running job, and verify that the output logs correctly display your input choices.

---

## 📘 Experiment 4: Multi-Job DAG & Build Artifact Transfers

### 1. Objective
To implement a Directed Acyclic Graph (DAG) using the `needs` keyword. Job A compiles assets and uploads them as a build artifact. Job B and Job C download the compiled assets, run validation suites in parallel, and report status before execution proceeds.

### 2. Workflow Configuration (`.github/workflows/04-multi-job-dag.yml`)
```yaml
name: Lab 4 - Multi-Job DAG and Artifacts

on:
  push:
    branches: [main]

jobs:
  # Job 1: Build Phase
  build-phase:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Compile Distribution Assets
        run: |
          mkdir -p build
          echo "Build-Artifact-ID: v1.0.0" > build/output.txt
          echo "Compilation Date: $(date)" >> build/output.txt

      - name: Upload Build Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: compile-assets
          path: build/
          retention-days: 2

  # Job 2: Validation Phase (Runs in parallel with Job 3, dependent on Job 1)
  test-unit:
    needs: build-phase
    runs-on: ubuntu-latest
    steps:
      - name: Download Build Assets
        uses: actions/download-artifact@v4
        with:
          name: compile-assets
          path: downloaded-build/

      - name: Run Unit Tests
        run: |
          echo "Verifying build assets contents:"
          cat downloaded-build/output.txt
          echo "Unit testing passed."

  # Job 3: Security Scan Phase (Runs in parallel with Job 2, dependent on Job 1)
  security-audit:
    needs: build-phase
    runs-on: ubuntu-latest
    steps:
      - name: Download Build Assets
        uses: actions/download-artifact@v4
        with:
          name: compile-assets
          path: downloaded-build/

      - name: Perform Vulnerability Scan
        run: |
          echo "Auditing package file dependencies..."
          cat downloaded-build/output.txt
          echo "Vulnerability scan completed. 0 threats found."
```

### 3. Verification Steps
1. Push the workflow to GitHub.
2. In the **Actions** tab, open the running workflow.
3. Observe the visual graph generated by GitHub. It will show a box named `build-phase` with lines branching out to `test-unit` and `security-audit`.
4. Open the summary page and scroll to the bottom to verify the `compile-assets.zip` artifact is available for download.

---

## 📘 Experiment 5: Multi-OS Concurrency Grid (Matrix Strategy) & Package Caching

### 1. Objective
To build a highly parallelized testing pipeline using a Matrix Strategy. The workflow must execute tests concurrently across both Linux (`ubuntu-latest`) and Windows (`windows-latest`) environments, using multiple Node.js runtime versions. It must also implement dependency caching using `actions/cache` to speed up subsequent run times.

### 2. Workflow Configuration (`.github/workflows/05-matrix-caching.yml`)
```yaml
name: Lab 5 - Matrix Testing and Dependency Caching

on:
  push:
    branches: [main]

jobs:
  run-tests:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false  # Ensure all matrix jobs run even if one fails
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]

    name: Test Grid - ${{ matrix.os }} (Node ${{ matrix.node-version }})
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js Runtime
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      # Configure Dependency Cache
      - name: Cache Node Packages
        id: cache-step
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      # Create dummy package.json / package-lock.json if not present
      - name: Initialize Dummy Project
        run: |
          if [ ! -f package.json ]; then
            echo '{"name": "lab-app", "version": "1.0.0", "scripts": {"test": "echo Run mock tests"}}' > package.json
            npm install
          fi
        shell: bash

      - name: Install dependencies
        run: npm ci

      - name: Execute Test Commands
        run: npm test
```

### 3. Execution & Verification Steps
1. Push the workflow to GitHub.
2. Check the Actions tab. You will see **4 separate runs** executing concurrently:
   * `Test Grid - ubuntu-latest (Node 18)`
   * `Test Grid - ubuntu-latest (Node 20)`
   * `Test Grid - windows-latest (Node 18)`
   * `Test Grid - windows-latest (Node 20)`
3. Trigger a second commit and push. Check the logs of the cache step; it will output **`Cache restored successfully`**, saving significant package installation time.

---

## 📘 Experiment 6: Enterprise Containerized CI/CD to GHCR & Remote SSH Deployment

### 1. Objective
To build an end-to-end containerized pipeline. The workflow must build a Docker image using a Multi-stage Dockerfile, configure layer caching with the GitHub Actions cache service (`gha`), publish the compiled container image to GitHub Container Registry (GHCR) securely using the default `GITHUB_TOKEN`, and SSH into a remote target VM to pull the new container and restart the service.

### 2. Prerequisites
1. A valid `Dockerfile` in the root of the repository.
2. Staging VM SSH secrets set in GitHub Repository Settings:
   * `STAGING_SERVER_IP`
   * `STAGING_SERVER_USER`
   * `STAGING_SSH_PRIVATE_KEY`

### 3. Workflow Configuration (`.github/workflows/06-container-deploy.yml`)
```yaml
name: Lab 6 - Container CI/CD and Remote Deployment

on:
  push:
    tags:
      - 'v*'  # Run only when version tags are pushed (e.g. v1.2.0)

permissions:
  contents: read
  packages: write

jobs:
  # Job 1: Build & Publish OCI Image to GHCR
  build-and-push-ghcr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Login using automatic GITHUB_TOKEN
      - name: Authenticate with GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # Extract metadata (tags, labels)
      - name: Generate Docker Metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository_owner }}/lab6-app
          tags: |
            type=semver,pattern={{version}}
            type=raw,value=latest

      # Build and Push image with GHA layer caching
      - name: Build and Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Job 2: Remote Deploy
  remote-ssh-deploy:
    needs: build-and-push-ghcr
    runs-on: ubuntu-latest
    steps:
      - name: Execute Deployment Script on Staging VM
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_SERVER_IP }}
          username: ${{ secrets.STAGING_SERVER_USER }}
          key: ${{ secrets.STAGING_SSH_PRIVATE_KEY }}
          port: 22
          script: |
            echo "==== DEPLOYING DOCKER CONTAINER ON REMOTE VM ===="
            # Login to GHCR from target server
            echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            
            # Stop existing instance
            docker stop staging-app-run || true
            docker rm staging-app-run || true
            
            # Pull latest package
            docker pull ghcr.io/${{ github.repository_owner }}/lab6-app:latest
            
            # Run container
            docker run -d \
              -p 8080:80 \
              --name staging-app-run \
              --restart unless-stopped \
              ghcr.io/${{ github.repository_owner }}/lab6-app:latest
            
            echo "Deployment successfully verified."
```

### 4. Verification Steps
1. Create a lightweight Dockerfile in the root of the repository:
   ```dockerfile
   FROM nginx:alpine
   RUN echo "<h1>DevOps Lab 6 Container</h1>" > /usr/share/nginx/html/index.html
   EXPOSE 80
   ```
2. Commit and tag: `git tag v1.0.0` and push tags: `git push origin v1.0.0`.
3. Check the Actions tab to monitor the compilation, registry push, and SSH command execution.
4. Navigate to your GitHub profile and select **Packages** to verify your published `lab6-app` container.
5. (If deploying to an active server) Open a browser and browse to `http://<STAGING_SERVER_IP>:8080` to verify your running container app.

---

## 🛠️ General Lab Troubleshooting Guidelines

### 1. YAML Parser Error
* **Symptom**: `Invalid workflow file: ... mapping values are not allowed here` or `root-level key is invalid`.
* **Solution**: Ensure your workflow files use **spaces, never tabs** for indentation. Use a YAML validation tool or the built-in VS Code editor helper to verify that keys at the same level align properly (typically using 2-space increments).

### 2. Actions Run Queued Indefinitely
* **Symptom**: The job remains in a grey "Queued" status with a spinning icon.
* **Solution**: Verify that your `runs-on` label matches available infrastructure. GitHub-hosted runners must use valid keywords like `ubuntu-latest`, `windows-latest`, or `macos-latest`. For self-hosted runners, verify that the runner agent software is running (`./run.sh`) and has matching labels.

### 3. GITHUB_TOKEN Push Denied to GHCR
* **Symptom**: `denied: Permission to write packaging data has been denied`.
* **Solution**: Ensure your workflow includes the explicit package permissions block at the top level or job level:
  ```yaml
  permissions:
    packages: write
  ```
  Ensure that GitHub Packaging Settings allow workflow-initiated updates.
