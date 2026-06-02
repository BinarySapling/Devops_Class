# Scenario 08: Docker Image Build in CI — Complete Beginner-Friendly Implementation Guide

**Project name:** `docker-build-demo`  
**GitHub username:** `alokmisra`  
**Git email:** `alokalokmm@gmail.com`  
**Scenario focus:** Validate a Dockerfile using GitHub Actions CI  
**Main goal:** Whenever code is pushed to GitHub, GitHub Actions should automatically run `docker build` and confirm whether the Docker image can be built successfully.

---

## 1. What You Are Implementing

In this scenario, you will create a small Node.js project and add a Dockerfile. Then you will create a GitHub Actions workflow file inside:

```text
.github/workflows/docker-build.yml
```

When you push the project to GitHub, GitHub Actions will:

1. Start a fresh Ubuntu machine called a **runner**.
2. Download your repository code using `actions/checkout@v4`.
3. Run the Docker build command.
4. Show the Docker images available on the runner.
5. Mark the workflow as successful if the Dockerfile is correct.

This workflow only checks whether the Docker image builds successfully. It does **not** push the image to Docker Hub or any other registry.

---

## 2. Final Project Structure

After completing all steps, your folder should look like this:

```text
docker-build-demo/
├── Dockerfile
├── .dockerignore
├── index.js
├── package.json
└── .github/
    └── workflows/
        └── docker-build.yml
```

---

## 3. Required Tools

Before starting, make sure these tools are installed:

1. **Git**
2. **GitHub account**
3. **VS Code or any text editor**
4. **Docker Desktop**  
   Docker Desktop is optional for GitHub Actions because GitHub runner already has Docker. However, it is useful if you want to test locally before pushing.

---

# PART A — Create the Project Locally

## Step 1: Open Command Prompt

Open **Command Prompt** on Windows.

You can search:

```text
cmd
```

Then press **Enter**.

---

## Step 2: Go to a Working Folder

Use any folder where you want to create the project. Example:

```cmd
cd %USERPROFILE%\Desktop
```

### Explanation

`cd` means **change directory**.  
This command moves you to the Desktop folder of your Windows user account.

---

## Step 3: Create the Project Folder

Run:

```cmd
mkdir docker-build-demo
```

### Explanation

`mkdir` means **make directory**.  
This creates a new folder named `docker-build-demo`.

---

## Step 4: Enter the Project Folder

Run:

```cmd
cd docker-build-demo
```

### Explanation

This moves you inside the project folder.

---

## Step 5: Initialize Git

Run:

```cmd
git init
```

### Explanation

This command makes the folder a Git repository.  
Git will now start tracking files in this project.

---

## Step 6: Configure Git Username

Run:

```cmd
git config user.name "alokmisra"
```

### Explanation

This sets your Git username for this project.

---

## Step 7: Configure Git Email

Run:

```cmd
git config user.email "alokalokmm@gmail.com"
```

### Explanation

This sets your Git email for this project.  
This email will be used in your Git commits.

---

# PART B — Create Node.js Application Files

## Step 8: Create `package.json`

Run:

```cmd
notepad package.json
```

When Notepad opens, paste the following code:

```json
{
  "name": "docker-build-demo",
  "version": "1.0.0",
  "description": "A simple Node.js application for Docker image build validation using GitHub Actions CI.",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "keywords": [
    "docker",
    "github-actions",
    "ci",
    "nodejs"
  ],
  "author": "alokmisra",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.3"
  }
}
```

Save the file using:

```text
Ctrl + S
```

Then close Notepad.

### Explanation

`package.json` is the configuration file for a Node.js project.

Important fields:

| Field | Meaning |
|---|---|
| `name` | Name of the project |
| `version` | Current version of the project |
| `main` | Main file that starts the project |
| `scripts` | Commands that can be run using npm |
| `dependencies` | External packages required by the project |

Here, we are using **Express.js** to create a simple web server.

---

## Step 9: Create `index.js`

Run:

```cmd
notepad index.js
```

Paste the following code:

```javascript
const express = require("express");

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.send("Docker Build CI is working successfully!");
});

app.listen(PORT, () => {
  console.log(`Application is running on port ${PORT}`);
});
```

Save and close Notepad.

### Explanation

This is a simple Node.js web application.

Line-by-line explanation:

| Code | Explanation |
|---|---|
| `const express = require("express");` | Imports the Express.js package |
| `const app = express();` | Creates an Express application |
| `const PORT = process.env.PORT || 3000;` | Uses server port from environment or default port 3000 |
| `app.get("/", ...)` | Creates a route for the homepage |
| `res.send(...)` | Sends a response to the browser |
| `app.listen(...)` | Starts the server |

---

# PART C — Create Docker Files

## Step 10: Create the Dockerfile

Run:

```cmd
notepad Dockerfile
```

Paste the following Dockerfile:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "index.js"]
```

Save and close Notepad.

---

## Step 11: Understand the Dockerfile

### Line-by-line explanation

| Dockerfile Line | Explanation |
|---|---|
| `FROM node:18-alpine` | Uses a lightweight Node.js version 18 image as the base image |
| `WORKDIR /app` | Creates and switches to `/app` folder inside the container |
| `COPY package*.json ./` | Copies `package.json` and optionally `package-lock.json` into the container |
| `RUN npm install` | Installs Node.js dependencies |
| `COPY . .` | Copies all project files into the container |
| `EXPOSE 3000` | Documents that the container application uses port 3000 |
| `CMD ["node", "index.js"]` | Starts the application when the container runs |

---

## Step 12: Create `.dockerignore`

Run:

```cmd
notepad .dockerignore
```

Paste the following content:

```text
node_modules
npm-debug.log
.git
.github
Dockerfile
.dockerignore
README.md
```

Save and close Notepad.

### Explanation

`.dockerignore` tells Docker which files should not be copied into the Docker build context.

This is useful because:

1. It makes Docker builds faster.
2. It avoids unnecessary files.
3. It prevents large folders like `node_modules` from being copied.

---

# PART D — Test Docker Build Locally

This part is optional but strongly recommended.

## Step 13: Check Docker Version

Run:

```cmd
docker --version
```

### Expected output example

```text
Docker version 25.x.x, build xxxxx
```

### Explanation

This checks whether Docker is installed.

If this command fails, open Docker Desktop and try again.

---

## Step 14: Build Docker Image Locally

Run:

```cmd
docker build -t my-node-app:local .
```

### Explanation

| Part | Meaning |
|---|---|
| `docker build` | Builds a Docker image |
| `-t my-node-app:local` | Gives name `my-node-app` and tag `local` to the image |
| `.` | Uses the current folder as the build context |

### Expected successful output

You should see output ending similar to:

```text
Successfully built
Successfully tagged my-node-app:local
```

In newer Docker versions, you may see BuildKit output such as:

```text
naming to docker.io/library/my-node-app:local done
```

Both are acceptable.

---

## Step 15: Run the Docker Container Locally

Run:

```cmd
docker run -d --name docker-build-demo-test -p 3000:3000 my-node-app:local
```

### Explanation

| Part | Meaning |
|---|---|
| `docker run` | Runs a container from an image |
| `-d` | Runs the container in background mode |
| `--name docker-build-demo-test` | Gives the container a name |
| `-p 3000:3000` | Maps local port 3000 to container port 3000 |
| `my-node-app:local` | Image name and tag |

---

## Step 16: Test the Running Application

Open a browser and visit:

```text
http://localhost:3000
```

You should see:

```text
Docker Build CI is working successfully!
```

Or run this command:

```cmd
curl http://localhost:3000
```

Expected output:

```text
Docker Build CI is working successfully!
```

---

## Step 17: Stop and Remove the Test Container

Run:

```cmd
docker stop docker-build-demo-test
```

Then run:

```cmd
docker rm docker-build-demo-test
```

### Explanation

`docker stop` stops the running container.  
`docker rm` removes the stopped container.

---

# PART E — Create GitHub Actions Workflow

## Step 18: Create `.github` Folder

Run:

```cmd
mkdir .github
```

### Explanation

GitHub Actions workflow files must be placed inside the `.github` folder.

---

## Step 19: Create `workflows` Folder

Run:

```cmd
mkdir .github\workflows
```

### Explanation

All workflow YAML files must be placed inside:

```text
.github/workflows/
```

---

## Step 20: Create Workflow File

Run:

```cmd
notepad .github\workflows\docker-build.yml
```

Paste the following YAML code:

```yaml
name: Docker Build CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  docker-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Display Docker version
        run: docker --version

      - name: Build Docker image
        run: docker build -t my-node-app:ci .

      - name: Show Docker images
        run: docker images
```

Save and close Notepad.

---

## Step 21: Understand the Workflow File

### 1. Workflow name

```yaml
name: Docker Build CI
```

This is the name shown in the GitHub Actions tab.

---

### 2. Trigger section

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

This means the workflow will run when:

1. Code is pushed to the `main` branch.
2. A pull request is created against the `main` branch.

### Beginner explanation

A **trigger** tells GitHub Actions when to start the automation.

---

### 3. Jobs section

```yaml
jobs:
  docker-build:
```

This creates a job named `docker-build`.

### Beginner explanation

A **job** is a group of steps that run together.

---

### 4. Runner section

```yaml
runs-on: ubuntu-latest
```

This tells GitHub to run the job on a fresh Ubuntu Linux machine.

### Beginner explanation

A **runner** is the machine where your workflow commands are executed.

---

### 5. Checkout step

```yaml
- name: Checkout repository code
  uses: actions/checkout@v4
```

This downloads your repository code into the GitHub runner.

### Beginner explanation

Without this step, GitHub Actions will not have your project files.

`@v4` means version 4 of the official checkout action.

---

### 6. Docker version step

```yaml
- name: Display Docker version
  run: docker --version
```

This confirms that Docker is available on the GitHub runner.

---

### 7. Docker build step

```yaml
- name: Build Docker image
  run: docker build -t my-node-app:ci .
```

This builds the Docker image.

| Part | Meaning |
|---|---|
| `docker build` | Build Docker image |
| `-t my-node-app:ci` | Give image name `my-node-app` and tag `ci` |
| `.` | Use current repository folder as build context |

---

### 8. Docker images step

```yaml
- name: Show Docker images
  run: docker images
```

This displays Docker images present on the runner after the build.

---

# PART F — Push Project to GitHub

## Step 22: Check Project Files

Run:

```cmd
dir
```

Expected files:

```text
Dockerfile
index.js
package.json
```

To see hidden folder structure, run:

```cmd
dir .github\workflows
```

Expected file:

```text
docker-build.yml
```

---

## Step 23: Check Git Status

Run:

```cmd
git status
```

### Explanation

This shows which files are new or modified.

---

## Step 24: Add Files to Git

Run:

```cmd
git add .
```

### Explanation

This stages all project files for commit.

---

## Step 25: Commit the Files

Run:

```cmd
git commit -m "Implement Docker image build CI"
```

### Explanation

This creates a save point in Git with a message.

---

## Step 26: Rename Branch to Main

Run:

```cmd
git branch -M main
```

### Explanation

This makes sure your branch name is `main`.

---

# PART G — Create GitHub Repository

## Step 27: Create Repository on GitHub

Go to GitHub and create a new repository with this exact name:

```text
docker-build-demo
```

Use these settings:

| Option | Value |
|---|---|
| Repository name | `docker-build-demo` |
| Visibility | Public or Private |
| Add README | No |
| Add .gitignore | No |
| Add license | No |

Important: Do not initialize the repository with README because you already created files locally.

---

## Step 28: Add GitHub Remote

Run:

```cmd
git remote add origin https://github.com/alokmisra/docker-build-demo.git
```

### Explanation

This connects your local project to your GitHub repository.

---

## Step 29: Verify Remote URL

Run:

```cmd
git remote -v
```

Expected output:

```text
origin  https://github.com/alokmisra/docker-build-demo.git (fetch)
origin  https://github.com/alokmisra/docker-build-demo.git (push)
```

---

## Step 30: Push Code to GitHub

Run:

```cmd
git push -u origin main
```

### Explanation

This uploads your local project to GitHub.

---

# PART H — Check GitHub Actions Result

## Step 31: Open Repository on GitHub

Open:

```text
https://github.com/alokmisra/docker-build-demo
```

---

## Step 32: Open Actions Tab

Click:

```text
Actions
```

You should see a workflow named:

```text
Docker Build CI
```

---

## Step 33: Open the Latest Workflow Run

Click the latest workflow run.

Then open:

```text
docker-build
```

You should see steps similar to:

```text
Checkout repository code
Display Docker version
Build Docker image
Show Docker images
```

---

## Step 34: Expected Successful Output

In the Docker build step, you should see messages similar to:

```text
Successfully built
Successfully tagged my-node-app:ci
```

Or, in newer Docker BuildKit output:

```text
naming to docker.io/library/my-node-app:ci done
```

In the `Show Docker images` step, you should see something similar to:

```text
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my-node-app     ci        xxxxxxxxxxxx   few seconds ago  xxxMB
```

---

# PART I — Complete Command Sequence

Use the following commands in sequence.

## Windows CMD command sequence

```cmd
cd %USERPROFILE%\Desktop
mkdir docker-build-demo
cd docker-build-demo

git init
git config user.name "alokmisra"
git config user.email "alokalokmm@gmail.com"

notepad package.json
notepad index.js
notepad Dockerfile
notepad .dockerignore

mkdir .github
mkdir .github\workflows
notepad .github\workflows\docker-build.yml

git status
git add .
git commit -m "Implement Docker image build CI"
git branch -M main

git remote add origin https://github.com/alokmisra/docker-build-demo.git
git remote -v
git push -u origin main
```

---

# PART J — Optional Local Docker Test Commands

Run these only if Docker Desktop is installed and running.

```cmd
docker --version
docker build -t my-node-app:local .
docker run -d --name docker-build-demo-test -p 3000:3000 my-node-app:local
curl http://localhost:3000
docker stop docker-build-demo-test
docker rm docker-build-demo-test
```

---

# PART K — Common Errors and Fixes

## Error 1: `fatal: remote origin already exists`

### Why this happens

You already added a remote URL earlier.

### Fix

Run:

```cmd
git remote set-url origin https://github.com/alokmisra/docker-build-demo.git
```

Then push again:

```cmd
git push -u origin main
```

---

## Error 2: `'https:' is not recognized as an internal or external command`

### Why this happens

You pasted only the GitHub URL directly into CMD:

```cmd
https://github.com/alokmisra/docker-build-demo.git
```

CMD thinks it is a command, but it is only a URL.

### Correct command

Use the URL with `git remote add origin`:

```cmd
git remote add origin https://github.com/alokmisra/docker-build-demo.git
```

Or if remote already exists:

```cmd
git remote set-url origin https://github.com/alokmisra/docker-build-demo.git
```

---

## Error 3: `Dockerfile: no such file or directory`

### Why this happens

The Dockerfile is missing or not placed in the root folder.

### Fix

Make sure the Dockerfile is directly inside:

```text
docker-build-demo/
```

Correct location:

```text
docker-build-demo/Dockerfile
```

Wrong location:

```text
docker-build-demo/.github/workflows/Dockerfile
```

---

## Error 4: `COPY failed`

### Why this happens

Dockerfile is trying to copy a file that does not exist.

### Fix

Check that these files exist:

```text
package.json
index.js
Dockerfile
```

Run:

```cmd
dir
```

---

## Error 5: `npm ERR! enoent Could not read package.json`

### Why this happens

`package.json` is missing or incorrectly named.

### Fix

Make sure the file name is exactly:

```text
package.json
```

Not:

```text
package.json.txt
```

In Windows, enable file extensions to check this.

---

## Error 6: GitHub Actions workflow not starting

### Possible reasons

1. Workflow file is not inside `.github/workflows/`.
2. YAML file extension is wrong.
3. Code was not pushed to GitHub.
4. Branch name is not `main`.

### Fix checklist

Run:

```cmd
dir .github\workflows
```

You should see:

```text
docker-build.yml
```

Then run:

```cmd
git branch
```

You should see:

```text
* main
```

Then push again:

```cmd
git add .
git commit -m "Fix workflow file location"
git push
```

---

## Error 7: YAML indentation error

### Why this happens

YAML is space-sensitive. Wrong indentation can break the workflow.

### Fix

Use exactly this indentation:

```yaml
name: Docker Build CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  docker-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Display Docker version
        run: docker --version

      - name: Build Docker image
        run: docker build -t my-node-app:ci .

      - name: Show Docker images
        run: docker images
```

---

# PART L — Explanation of Important GitHub Actions Terms

## 1. Workflow

A workflow is an automation file written in YAML.

Example:

```text
docker-build.yml
```

It tells GitHub what task to perform automatically.

---

## 2. Trigger

A trigger tells GitHub Actions when to run.

Example:

```yaml
on:
  push:
  pull_request:
```

This means the workflow runs on push and pull request.

---

## 3. Job

A job is a collection of steps.

Example:

```yaml
jobs:
  docker-build:
```

Here, `docker-build` is the job name.

---

## 4. Runner

A runner is the machine that executes the workflow.

Example:

```yaml
runs-on: ubuntu-latest
```

This means GitHub will use a Linux machine.

---

## 5. Step

A step is a single task inside a job.

Example:

```yaml
- name: Build Docker image
  run: docker build -t my-node-app:ci .
```

---

## 6. Action

An action is a reusable task created by GitHub or the community.

Example:

```yaml
uses: actions/checkout@v4
```

This action downloads repository code into the runner.

---

# PART M — Student Submission Checklist

Submit the following:

1. GitHub repository link:

```text
https://github.com/alokmisra/docker-build-demo
```

2. Screenshot of project structure.
3. Screenshot of `docker-build.yml`.
4. Screenshot of successful GitHub Actions workflow.
5. Screenshot of `Build Docker image` step.
6. Short explanation of:
   - Trigger
   - Job
   - Runner
   - Steps
   - Docker build command
7. Any error encountered and how it was fixed.

---

# PART N — Viva Questions with Answers

## Q1. What is the purpose of this workflow?

The purpose is to validate whether the Dockerfile can successfully build a Docker image whenever code is pushed or a pull request is created.

---

## Q2. Which trigger is used in this scenario?

The workflow uses `push` and `pull_request` triggers.

---

## Q3. Which runner is used?

The workflow uses:

```yaml
runs-on: ubuntu-latest
```

This means the workflow runs on a GitHub-hosted Ubuntu machine.

---

## Q4. What is the purpose of `actions/checkout@v4`?

It downloads the repository code into the GitHub Actions runner so that the workflow can access files like `Dockerfile`, `package.json`, and `index.js`.

---

## Q5. What does this command do?

```cmd
docker build -t my-node-app:ci .
```

It builds a Docker image from the current folder and gives it the name `my-node-app` with tag `ci`.

---

## Q6. Does this workflow push the image to Docker Hub?

No. This workflow only builds the Docker image. It does not push the image to Docker Hub.

---

## Q7. What happens if the Dockerfile has an error?

The workflow fails, and GitHub Actions shows the error in the build logs.

---

## Q8. Why do we use `.dockerignore`?

We use `.dockerignore` to avoid copying unnecessary files into the Docker build context.

---

# PART O — Final Summary

In this implementation, you created a Node.js application, wrote a Dockerfile, added a GitHub Actions workflow, and pushed the project to GitHub. The workflow automatically validates the Docker image build using GitHub Actions.

The most important command in this scenario is:

```cmd
docker build -t my-node-app:ci .
```

The most important workflow file is:

```text
.github/workflows/docker-build.yml
```

Once GitHub Actions shows a green tick, it means your Dockerfile build validation is successful.
