# Continuous Integration (CI) with GitHub Actions

## Beginner-Friendly Detailed Notes with Implementation Examples

---

# 1. What is Continuous Integration?

**Continuous Integration (CI)** is a DevOps practice where developers frequently push code to a shared repository, and every code change is automatically checked through build, test, linting, and security verification.

In simple words:

> CI means whenever a developer uploads code to GitHub, GitHub automatically checks whether the code is working correctly or not.

GitHub Actions is GitHub’s built-in automation platform for CI/CD. A workflow is written in YAML and stored inside `.github/workflows/`.

---

# 2. Understanding Workflow Automation

Workflow automation means instructing GitHub to automatically perform tasks such as:

```text
Code pushed → Install dependencies → Run tests → Build project → Create Docker image → Deploy
```

Example:

```yaml
name: Basic CI Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Print message
        run: echo "CI workflow is running successfully"
```

Explanation:

| Line | Meaning |
|---|---|
| `name` | Name of the workflow |
| `on: push` | Workflow runs when code is pushed |
| `jobs` | Group of work to perform |
| `runs-on` | Machine where job runs |
| `steps` | Individual commands/actions |
| `uses` | Use existing marketplace action |
| `run` | Execute shell command |

---

# 3. Events and Triggers

An **event** is something that happens in GitHub.

A **trigger** tells GitHub when to start the workflow.

Common triggers:

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
  schedule:
    - cron: "0 0 * * *"
```

---

## 3.1 Push Trigger

Runs when code is pushed.

```yaml
on:
  push:
    branches:
      - main
```

Use case:

```text
Developer pushes code to main branch → CI starts automatically
```

---

## 3.2 Pull Request Trigger

Runs when someone opens or updates a pull request.

```yaml
on:
  pull_request:
    branches:
      - main
```

Use case:

```text
Student/developer submits code → Tests run before merging
```

---

## 3.3 Schedule Trigger

Runs at a fixed time using cron.

```yaml
on:
  schedule:
    - cron: "0 2 * * *"
```

This runs every day at 2:00 AM UTC.

Use case:

```text
Run nightly build every day
```

---

## 3.4 Manual Workflow Trigger

Manual trigger allows the user to start workflow from GitHub UI.

```yaml
on:
  workflow_dispatch:
```

Use case:

```text
Teacher/developer manually starts deployment or testing
```

---

# 4. Workflow Directory Structure

GitHub Actions workflow files must be stored in:

```text
.github/workflows/
```

Example project structure:

```text
my-project/
│
├── src/
│   └── app.py
│
├── tests/
│   └── test_app.py
│
├── requirements.txt
│
└── .github/
    └── workflows/
        └── ci.yml
```

Important points:

```text
.github/workflows/ci.yml
.github/workflows/test.yml
.github/workflows/deploy.yml
```

A repository can have multiple workflows.

---

# 5. Key Component: Workflow

A **workflow** is the complete automation file.

Example:

```yaml
name: Python CI

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run simple command
        run: echo "Hello GitHub Actions"
```

A workflow can contain:

```text
Name
Trigger
Jobs
Steps
Actions
Shell commands
Environment variables
Secrets
Deployment instructions
```

---

# 6. Key Component: Jobs

A **job** is a collection of steps.

Example:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building project"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing project"
```

Each job usually runs independently.

Example:

```text
Job 1: Build
Job 2: Test
Job 3: Deploy
```

---

# 7. Key Component: Steps

A **step** is one task inside a job.

Example:

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test
```

Steps run in sequence.

```text
Step 1 → Step 2 → Step 3
```

---

# 8. Key Component: Actions

An **action** is a reusable automation component.

Example:

```yaml
uses: actions/checkout@v4
```

This action downloads your repository code into the runner.

Common actions:

```yaml
actions/checkout@v4
actions/setup-node@v4
actions/setup-python@v5
actions/setup-java@v4
docker/login-action@v3
docker/build-push-action@v6
```

---

# 9. Key Component: Runners

A **runner** is the machine where your workflow runs.

Example:

```yaml
runs-on: ubuntu-latest
```

Common GitHub-hosted runners:

```yaml
ubuntu-latest
windows-latest
macos-latest
```

---

# 10. GitHub-Hosted Runners

GitHub-hosted runners are virtual machines provided by GitHub.

Example:

```yaml
runs-on: ubuntu-latest
```

Advantages:

```text
Easy to use
No installation required
Clean machine every time
Good for students and small projects
```

Disadvantages:

```text
Limited control
Time/resource limits
May be slower for very large builds
```

---

# 11. Self-Hosted Runners

A self-hosted runner is your own machine/server connected to GitHub Actions.

Example:

```yaml
runs-on: self-hosted
```

Use case:

```text
You have your own lab server or company server and want workflows to run there.
```

Advantages:

```text
More control
Can use private tools
Can access internal network
Useful for deployment
```

Disadvantages:

```text
Security risk if not managed properly
Needs maintenance
Machine must remain online
```

---

# 12. Runner Security and Management

Important practices:

```text
Do not run untrusted code on self-hosted runners
Use separate runners for public repositories
Keep runner software updated
Use least privilege
Store credentials in GitHub Secrets
Avoid printing secrets in logs
```

Security is important because CI/CD workflows can become a software supply-chain attack surface if permissions and actions are not controlled properly.

---

# 13. Steps and Shell Commands

You can run Linux shell commands directly.

Example:

```yaml
steps:
  - name: Show current directory
    run: pwd

  - name: List files
    run: ls -la

  - name: Print message
    run: echo "Build started"
```

Multiple commands:

```yaml
- name: Run multiple commands
  run: |
    echo "Installing dependencies"
    npm install
    echo "Running tests"
    npm test
```

---

# 14. Using Marketplace Actions

GitHub Marketplace provides reusable actions.

Example:

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

Example Node.js setup:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 20
```

Example Python setup:

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.11"
```

---

# 15. Language-Specific Actions

## 15.1 Node.js CI Example

```yaml
name: Node.js CI

on:
  push:
  pull_request:

jobs:
  node-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

---

## 15.2 Python CI Example

```yaml
name: Python CI

on:
  push:
  pull_request:

jobs:
  python-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

---

## 15.3 Java Maven CI Example

```yaml
name: Java Maven CI

on:
  push:
  pull_request:

jobs:
  maven-build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Build with Maven
        run: mvn clean package
```

---

# 16. Using Caching for Faster Builds

Caching stores downloaded dependencies so the next workflow runs faster.

Example Node.js cache:

```yaml
- name: Setup Node.js with cache
  uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm
```

Example Python cache:

```yaml
- name: Setup Python with pip cache
  uses: actions/setup-python@v5
  with:
    python-version: "3.11"
    cache: pip
```

Example Maven cache:

```yaml
- name: Setup Java with Maven cache
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: 17
    cache: maven
```

Why caching helps:

```text
Without cache: dependencies download every time
With cache: old dependencies are reused
Result: faster builds
```

---

# 17. Jobs and Matrix Strategies

A **matrix strategy** runs the same job with multiple configurations.

Example: test Node.js project on multiple versions.

```yaml
name: Matrix CI

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18, 20, 22]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

Meaning:

```text
GitHub will run test job 3 times:
1. Node.js 18
2. Node.js 20
3. Node.js 22
```

---

# 18. Multi-Job Workflows

A workflow can contain multiple jobs.

Example:

```yaml
name: Multi Job CI

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building application"

  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing application"

  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying application"
```

By default, jobs run in parallel.

To run jobs in order, use `needs`.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build complete"

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing after build"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying after test"
```

Flow:

```text
Build → Test → Deploy
```

---

# 19. Using Artifacts

Artifacts are files generated during workflow execution.

Example:

```yaml
- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

Use case:

```text
Store compiled files, reports, test results, logs
```

---

# 20. Docker and GitHub Actions

GitHub Actions is very useful for Docker-based CI.

Common Docker CI tasks:

```text
Build Docker image
Run container tests
Login to Docker registry
Push image to Docker Hub
Push image to GitHub Container Registry
Deploy image to server/cloud
```

---

# 21. Building Docker Images in CI

Example Dockerfile:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

GitHub Actions workflow:

```yaml
name: Docker Build CI

on:
  push:
    branches:
      - main

jobs:
  docker-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t my-node-app .
```

Explanation:

```text
GitHub runner checks out code
Docker builds image using Dockerfile
Image name becomes my-node-app
```

---

# 22. Pushing Docker Image to Docker Hub

First, add secrets in GitHub:

```text
Repository → Settings → Secrets and variables → Actions → New repository secret
```

Add:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

Workflow:

```yaml
name: Docker Hub Push

on:
  push:
    branches:
      - main

jobs:
  docker-push:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: yourdockerhubusername/my-node-app:latest
```

---

# 23. Pushing Docker Image to GitHub Container Registry

GitHub Container Registry is also called **GHCR**.

Image format:

```text
ghcr.io/username/image-name:tag
```

Workflow:

```yaml
name: Push to GHCR

on:
  push:
    branches:
      - main

permissions:
  contents: read
  packages: write

jobs:
  ghcr-push:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}/my-app:latest
```

---

# 24. Deploying to Servers Using GitHub Actions

Deployment means copying or running your application on a production server.

Example deployment flow:

```text
Push code → Build Docker image → Push image → SSH into server → Pull image → Restart container
```

Example using SSH:

```yaml
name: Deploy to Server

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy using SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            docker pull yourdockerhubusername/my-node-app:latest
            docker stop my-node-app || true
            docker rm my-node-app || true
            docker run -d --name my-node-app -p 80:3000 yourdockerhubusername/my-node-app:latest
```

Secrets required:

```text
SERVER_HOST
SERVER_USER
SERVER_SSH_KEY
```

---

# 25. Deploying to Cloud

GitHub Actions can deploy to:

```text
AWS
Azure
Google Cloud
DigitalOcean
Heroku
Render
Vercel
Netlify
Kubernetes
Docker-based VPS
```

Simple deployment idea:

```text
CI pipeline builds and tests code.
CD pipeline deploys only if tests pass.
```

Example with job dependency:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Tests passed"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to cloud"
```

---

# 26. Complete CI Example for Node.js + Docker

```yaml
name: Complete Node Docker CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test

  docker:
    needs: test
    runs-on: ubuntu-latest

    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: yourdockerhubusername/my-node-app:latest
```

Explanation:

```text
Workflow runs on push, pull request, or manual trigger.
First job runs tests.
Second job builds and pushes Docker image.
Docker job runs only after test job succeeds.
Docker image is pushed only from main branch.
```

---

# 27. Complete CI Example for Java Maven Project

```yaml
name: Java Maven CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  build-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        java-version: [17, 21]

    steps:
      - name: Checkout project
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: ${{ matrix.java-version }}
          cache: maven

      - name: Build project
        run: mvn clean package

      - name: Run tests
        run: mvn test
```

---

# 28. Complete CI Example for Python Project

```yaml
name: Python CI

on:
  push:
  pull_request:

jobs:
  python-ci:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

---

# 29. Environment Variables

Environment variables store reusable values.

Example:

```yaml
env:
  APP_NAME: my-app
  VERSION: 1.0.0

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building $APP_NAME version $VERSION"
```

---

# 30. Secrets

Secrets store sensitive values.

Examples:

```text
Docker password
SSH private key
Cloud API key
Database password
```

Use secrets like this:

```yaml
${{ secrets.DOCKERHUB_TOKEN }}
```

Never write passwords directly in YAML.

Wrong:

```yaml
password: mypassword123
```

Correct:

```yaml
password: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

# 31. Manual Workflow with Inputs

You can ask for input before running workflow manually.

```yaml
name: Manual Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Choose deployment environment"
        required: true
        default: "staging"

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Print environment
        run: echo "Deploying to ${{ github.event.inputs.environment }}"
```

---

# 32. Conditional Execution

Run a step only under certain conditions.

```yaml
- name: Deploy only on main branch
  if: github.ref == 'refs/heads/main'
  run: echo "Deploying..."
```

Example:

```text
If branch is main → deploy
If branch is not main → skip deploy
```

---

# 33. Common Mistakes

| Mistake | Correction |
|---|---|
| Wrong folder | Use `.github/workflows/` |
| Wrong YAML spacing | Use proper indentation |
| Missing checkout | Add `actions/checkout@v4` |
| Secret printed in logs | Never echo secrets |
| Deployment runs on every branch | Use branch condition |
| Self-hosted runner exposed | Restrict access |
| No cache | Add dependency caching |
| No job dependency | Use `needs` |

---

# 34. Best Practices

```text
Use separate workflows for build, test, and deployment
Use secrets for sensitive data
Use caching to improve speed
Use matrix testing for multiple versions
Use branch protection rules
Use pull request checks
Use least-privilege permissions
Pin important actions to stable versions
Avoid running deployment on pull requests
Use self-hosted runners carefully
```

---

# 35. Beginner Learning Flow

For a new learner, follow this order:

```text
1. Create GitHub repository
2. Create .github/workflows/ci.yml
3. Add push trigger
4. Add checkout step
5. Add language setup step
6. Install dependencies
7. Run tests
8. Add caching
9. Add matrix strategy
10. Add Docker build
11. Push image to Docker Hub/GHCR
12. Add deployment step
```

---

# 36. Final Summary

GitHub Actions helps automate software development tasks directly inside GitHub. For CI, it is mainly used to automatically build, test, verify, package, and optionally deploy applications whenever code changes.

The main structure is:

```text
Workflow → Jobs → Steps → Actions/Commands → Runner
```

A complete CI/CD pipeline may look like:

```text
Push Code
   ↓
Checkout Repository
   ↓
Setup Language
   ↓
Install Dependencies
   ↓
Run Tests
   ↓
Build Application
   ↓
Build Docker Image
   ↓
Push Image to Registry
   ↓
Deploy to Server/Cloud
```

---

# 37. Practical Implementation Activity for Students

## Activity Title

**Create Your First GitHub Actions CI Pipeline**

## Objective

To understand how CI works by creating a simple workflow that automatically runs whenever code is pushed to GitHub.

## Required Tools

```text
GitHub account
Git installed on system
Any code editor such as VS Code
A simple Node.js, Python, or Java project
```

## Step-by-Step Implementation

### Step 1: Create a GitHub Repository

Create a new repository on GitHub.

Example repository name:

```text
github-actions-ci-demo
```

### Step 2: Clone the Repository

```bash
git clone https://github.com/your-username/github-actions-ci-demo.git
cd github-actions-ci-demo
```

### Step 3: Create Workflow Folder

```bash
mkdir -p .github/workflows
```

### Step 4: Create CI File

Create a file:

```text
.github/workflows/ci.yml
```

### Step 5: Add Basic Workflow

```yaml
name: First CI Workflow

on:
  push:
    branches:
      - main

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Print welcome message
        run: echo "Welcome to GitHub Actions CI"

      - name: Show files
        run: ls -la
```

### Step 6: Commit and Push

```bash
git add .
git commit -m "Add first GitHub Actions workflow"
git push origin main
```

### Step 7: Check Workflow Execution

Go to:

```text
GitHub Repository → Actions tab
```

You will see your workflow running.

---

# 38. Classroom Explanation Example

Suppose a student writes code and pushes it to GitHub. Earlier, the teacher or team leader had to manually check whether the code was working. With GitHub Actions, this checking becomes automatic.

For example:

```text
Student pushes assignment code
GitHub Actions starts automatically
Workflow installs dependencies
Workflow runs tests
If tests pass, code is accepted
If tests fail, student gets feedback immediately
```

This saves time and improves software quality.

---

# 39. Real-Life Use Case

A company developing an e-commerce website may use GitHub Actions like this:

```text
Developer pushes payment module code
CI pipeline runs unit tests
Security checks are performed
Docker image is created
Image is pushed to Docker Hub
Application is deployed to cloud server
```

This ensures that faulty code does not reach customers.

---

# 40. Viva Questions and Answers

## Q1. What is GitHub Actions?

GitHub Actions is an automation platform provided by GitHub for building, testing, and deploying code directly from a GitHub repository.

## Q2. What is a workflow?

A workflow is an automated process defined in a YAML file inside `.github/workflows/`.

## Q3. What is a runner?

A runner is a machine that executes the jobs defined in a GitHub Actions workflow.

## Q4. What is the difference between job and step?

A job is a collection of steps. A step is an individual task inside a job.

## Q5. What is `actions/checkout` used for?

It is used to download the repository code into the runner so that the workflow can access and process it.

## Q6. What is a matrix strategy?

A matrix strategy allows the same job to run with multiple configurations, such as different Node.js, Python, or Java versions.

## Q7. Why are secrets used?

Secrets are used to store sensitive values such as passwords, tokens, SSH keys, and API keys securely.

## Q8. What is GHCR?

GHCR stands for GitHub Container Registry. It is used to store and manage Docker container images inside GitHub.

## Q9. What is the benefit of caching?

Caching reduces build time by reusing previously downloaded dependencies.

## Q10. Why should self-hosted runners be used carefully?

Because they run on your own machine or server and may expose internal systems if untrusted code is executed.

---

# 41. Assignment Questions

1. Create a GitHub Actions workflow that runs on every push.
2. Create a workflow that runs on pull requests.
3. Create a workflow that runs manually using `workflow_dispatch`.
4. Create a Node.js CI pipeline using `actions/setup-node`.
5. Create a Python CI pipeline using `actions/setup-python`.
6. Create a Java Maven CI pipeline using `actions/setup-java`.
7. Add caching to any one workflow.
8. Create a matrix strategy for multiple language versions.
9. Build a Docker image using GitHub Actions.
10. Push a Docker image to Docker Hub or GHCR.

---

# 42. Mini Project Idea

## Title

**Automated CI/CD Pipeline for a Containerized Web Application Using GitHub Actions**

## Problem Statement

Manual testing and deployment of web applications is time-consuming and error-prone. There is a need for an automated pipeline that can test, build, package, and deploy an application whenever code is updated.

## Objectives

1. To create a GitHub Actions workflow for automatic testing.
2. To build a Docker image using GitHub Actions.
3. To push the image to Docker Hub or GHCR.
4. To deploy the application to a server or cloud platform.
5. To improve development speed and software reliability.

## Expected Outcome

```text
A working GitHub Actions pipeline that automatically builds, tests, dockerizes, and deploys an application.
```

---

# 43. Suggested Lab Record Format

```text
Experiment No.:
Experiment Title:
Objective:
Software Required:
Theory:
Procedure:
Workflow YAML Code:
Execution Screenshots:
Output:
Result:
Viva Questions:
```

---

# 44. Conclusion

GitHub Actions is an important CI/CD tool for modern DevOps. It helps developers automate repetitive tasks such as testing, building, packaging, and deployment. For beginners, the most important concepts to understand are workflow, jobs, steps, actions, triggers, runners, secrets, caching, matrix strategy, and Docker integration.

By learning GitHub Actions, students and developers can understand how modern software companies maintain code quality and deliver applications faster.

