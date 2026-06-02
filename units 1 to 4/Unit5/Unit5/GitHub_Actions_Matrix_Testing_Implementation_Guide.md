# Scenario 04: Matrix Testing for Multiple Node.js Versions using GitHub Actions

**Author:** Alok Misra  
**Designation:** Professor DevOps  
**Institution:** Lovely Professional University  
**Project Name:** `node-matrix-demo`

---

## 1. Aim of This Practical

The aim of this practical is to create a simple Node.js project and configure **GitHub Actions CI** using **matrix strategy** so that the same project is automatically tested on multiple Node.js versions: **16, 18, and 20**.

In simple words, we want GitHub to automatically check whether our Node.js application works correctly on different Node.js versions.

---

## 2. Simple Explanation for Naive Students

Suppose you write one Node.js program. It may work on your laptop, but another person may use a different Node.js version.

For example:

- Your laptop may have Node.js 20.
- Your friend may have Node.js 18.
- A server may still use Node.js 16.

So, instead of manually testing the same code again and again, GitHub Actions can test the code automatically on Node.js 16, 18, and 20.

This is called **matrix testing**.

---

## 3. What is GitHub Actions?

GitHub Actions is a CI/CD automation tool provided by GitHub.

It can automatically perform tasks such as:

- Checking out source code.
- Installing dependencies.
- Running tests.
- Building applications.
- Creating Docker images.
- Deploying applications.

In this practical, we use GitHub Actions for **Continuous Integration**, also called **CI**.

---

## 4. What is Matrix Strategy?

Matrix strategy means running the same job multiple times with different values.

In this practical:

```yaml
node-version: [16, 18, 20]
```

This means GitHub Actions will create three automatic job runs:

```text
Job 1: Test using Node.js 16
Job 2: Test using Node.js 18
Job 3: Test using Node.js 20
```

So, one workflow definition becomes three test executions.

---

## 5. Required Tools

Before starting, make sure the following tools are available:

| Tool | Purpose |
|---|---|
| GitHub Account | To create repository and run GitHub Actions |
| Git | To push project files to GitHub |
| Node.js | To run project locally |
| VS Code | To edit files easily |
| Command Prompt / Terminal | To run commands |

---

## 6. Final Directory Structure

Create the following directory structure:

```text
node-matrix-demo/
├── .github/
│   └── workflows/
│       └── matrix-ci.yml
├── .gitignore
├── index.js
├── package.json
├── README.md
└── test.js
```

---

## 7. Explanation of Each File

| File / Folder | Meaning |
|---|---|
| `node-matrix-demo/` | Main project folder |
| `.github/` | Special GitHub folder for automation files |
| `.github/workflows/` | Folder where GitHub Actions workflow files are stored |
| `matrix-ci.yml` | GitHub Actions workflow file |
| `package.json` | Node.js project configuration file |
| `index.js` | Main application file |
| `test.js` | Test file used by CI |
| `.gitignore` | Tells Git which files/folders to ignore |
| `README.md` | Project explanation file |

---

## 8. Step-by-Step Implementation

### Step 1: Create Project Folder

Open Command Prompt and run:

```bash
mkdir node-matrix-demo
cd node-matrix-demo
```

Explanation:

- `mkdir node-matrix-demo` creates a new folder.
- `cd node-matrix-demo` enters that folder.

---

### Step 2: Initialize Node.js Project

Run:

```bash
npm init -y
```

Explanation:

This creates a `package.json` file automatically.

---

### Step 3: Create Application File

Create a file named:

```text
index.js
```

Add the following code:

```javascript
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

function getNodeVersionMessage() {
  return `Running tests using Node.js version: ${process.version}`;
}

module.exports = {
  add,
  multiply,
  getNodeVersionMessage
};

if (require.main === module) {
  console.log("Node Matrix Demo Application");
  console.log(getNodeVersionMessage());
  console.log("2 + 3 =", add(2, 3));
  console.log("4 * 5 =", multiply(4, 5));
}
```

Explanation:

- `add()` adds two numbers.
- `multiply()` multiplies two numbers.
- `getNodeVersionMessage()` prints the currently used Node.js version.
- `module.exports` allows these functions to be used in the test file.

---

### Step 4: Create Test File

Create a file named:

```text
test.js
```

Add the following code:

```javascript
const assert = require("assert");
const { add, multiply, getNodeVersionMessage } = require("./index");

console.log("Starting test execution...");
console.log(getNodeVersionMessage());

assert.strictEqual(add(2, 3), 5, "2 + 3 should be 5");
assert.strictEqual(add(-1, 1), 0, "-1 + 1 should be 0");
assert.strictEqual(multiply(4, 5), 20, "4 * 5 should be 20");
assert.strictEqual(multiply(0, 9), 0, "0 * 9 should be 0");

console.log("All tests passed successfully.");
```

Explanation:

- `assert` is a built-in Node.js testing module.
- It checks whether actual output is equal to expected output.
- If any test fails, GitHub Actions will show failure.
- If all tests pass, the workflow will show success.

---

### Step 5: Update `package.json`

Replace the content of `package.json` with:

```json
{
  "name": "node-matrix-demo",
  "version": "1.0.0",
  "description": "A simple Node.js project to demonstrate GitHub Actions matrix testing on Node.js 16, 18, and 20.",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "node test.js"
  },
  "keywords": [
    "github-actions",
    "ci",
    "matrix-testing",
    "nodejs"
  ],
  "author": "Alok Misra, Professor DevOps, Lovely Professional University",
  "license": "MIT"
}
```

Explanation:

The most important part is:

```json
"scripts": {
  "test": "node test.js"
}
```

When GitHub Actions runs:

```bash
npm test
```

it will execute:

```bash
node test.js
```

---

### Step 6: Test Locally Before GitHub Actions

Run:

```bash
npm install
npm test
```

Expected output:

```text
Starting test execution...
Running tests using Node.js version: v20.x.x
All tests passed successfully.
```

Your Node.js version may be different locally. That is normal.

---

### Step 7: Create GitHub Actions Workflow Folder

Run the following commands:

```bash
mkdir .github
mkdir .github\workflows
```

For Git Bash, macOS, or Linux, use:

```bash
mkdir -p .github/workflows
```

Explanation:

GitHub Actions only detects workflow files placed inside:

```text
.github/workflows/
```

---

### Step 8: Create Workflow File

Create this file:

```text
.github/workflows/matrix-ci.yml
```

Add the following YAML code:

```yaml
name: Node.js Matrix Testing

on:
  push:
  pull_request:
  workflow_dispatch:

jobs:
  test:
    name: Test on Node.js ${{ matrix.node-version }}
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        node-version: [16, 18, 20]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm

      - name: Show Node.js and npm versions
        run: |
          node --version
          npm --version

      - name: Install dependencies
        run: npm install

      - name: Run tests on Node.js ${{ matrix.node-version }}
        run: npm test
```

---

## 9. Line-by-Line Explanation of Workflow

### Workflow Name

```yaml
name: Node.js Matrix Testing
```

This is the name shown in the GitHub Actions tab.

---

### Workflow Triggers

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
```

Meaning:

| Trigger | Meaning |
|---|---|
| `push` | Workflow runs when code is pushed |
| `pull_request` | Workflow runs when a pull request is created or updated |
| `workflow_dispatch` | Allows manual run from GitHub Actions tab |

---

### Job Name

```yaml
jobs:
  test:
```

This creates a job called `test`.

A job is a group of steps executed on a runner.

---

### Runner

```yaml
runs-on: ubuntu-latest
```

This means GitHub will provide a temporary Ubuntu Linux machine to run the job.

---

### Matrix Strategy

```yaml
strategy:
  fail-fast: false
  matrix:
    node-version: [16, 18, 20]
```

Meaning:

- `node-version` is a matrix variable.
- GitHub runs the same job for Node.js 16, 18, and 20.
- `fail-fast: false` means if one version fails, other versions will still continue.

---

### Checkout Repository

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

This downloads your GitHub repository code into the runner.

Without this step, GitHub Actions will not have access to your project files.

---

### Setup Node.js

```yaml
- name: Setup Node.js ${{ matrix.node-version }}
  uses: actions/setup-node@v4
  with:
    node-version: ${{ matrix.node-version }}
    cache: npm
```

This installs the required Node.js version.

For each matrix run:

```text
First run  -> Node.js 16
Second run -> Node.js 18
Third run  -> Node.js 20
```

---

### Show Versions

```yaml
- name: Show Node.js and npm versions
  run: |
    node --version
    npm --version
```

This helps students see which Node.js version is being used in each job.

---

### Install Dependencies

```yaml
- name: Install dependencies
  run: npm install
```

This installs project dependencies.

In this simple project, there are no external dependencies, but this command is still useful because real projects usually have dependencies.

---

### Run Tests

```yaml
- name: Run tests on Node.js ${{ matrix.node-version }}
  run: npm test
```

This executes the test command from `package.json`.

---

## 10. Create `.gitignore` File

Create `.gitignore` and add:

```gitignore
node_modules/
npm-debug.log
.DS_Store
.env
coverage/
```

Explanation:

- `node_modules/` should not be pushed to GitHub.
- It can be recreated using `npm install`.

---

## 11. Create README File

Create `README.md` and add:

```markdown
# Node Matrix Demo

This project demonstrates GitHub Actions matrix testing for Node.js versions 16, 18, and 20.

## Local commands

```bash
npm install
npm test
```

## Workflow file

```text
.github/workflows/matrix-ci.yml
```
```

---

## 12. Exact Git Commands to Push Project to GitHub

### Step 12.1: Initialize Git

```bash
git init
```

Explanation:

This converts your folder into a Git repository.

---

### Step 12.2: Check Files

```bash
git status
```

Explanation:

This shows which files are new or modified.

---

### Step 12.3: Add Files

```bash
git add .
```

Explanation:

This prepares all files for commit.

---

### Step 12.4: Commit Files

```bash
git commit -m "Add Node.js matrix testing workflow"
```

Explanation:

This saves the current project state in Git.

---

### Step 12.5: Rename Branch to Main

```bash
git branch -M main
```

Explanation:

This ensures the branch name is `main`.

---

### Step 12.6: Create GitHub Repository

Go to GitHub and create a new repository named:

```text
node-matrix-demo
```

Do not initialize with README, `.gitignore`, or license because files already exist locally.

---

### Step 12.7: Add GitHub Remote

Replace `YOUR-USERNAME` with your actual GitHub username:

```bash
git remote add origin https://github.com/YOUR-USERNAME/node-matrix-demo.git
```

Example:

```bash
git remote add origin https://github.com/alokmisra/node-matrix-demo.git
```

---

### Step 12.8: Push Code

```bash
git push -u origin main
```

Explanation:

This uploads your local project to GitHub.

After this command, GitHub Actions will automatically start because the workflow contains the `push` trigger.

---

## 13. How to See GitHub Actions Output

1. Open your GitHub repository.
2. Click on the **Actions** tab.
3. Click on **Node.js Matrix Testing**.
4. Open the latest workflow run.
5. You will see three jobs:

```text
Test on Node.js 16
Test on Node.js 18
Test on Node.js 20
```

If all tests pass, green tick marks will appear.

---

## 14. Expected Output in GitHub Actions

Expected result:

```text
Test on Node.js 16 - success
Test on Node.js 18 - success
Test on Node.js 20 - success
```

Inside logs, students should see output similar to:

```text
Starting test execution...
Running tests using Node.js version: v16.x.x
All tests passed successfully.
```

For another matrix job:

```text
Starting test execution...
Running tests using Node.js version: v18.x.x
All tests passed successfully.
```

For another matrix job:

```text
Starting test execution...
Running tests using Node.js version: v20.x.x
All tests passed successfully.
```

---

## 15. Full Command Sequence for Windows Command Prompt

Use these commands line by line:

```bash
mkdir node-matrix-demo
cd node-matrix-demo
npm init -y
notepad index.js
notepad test.js
notepad package.json
mkdir .github
mkdir .github\workflows
notepad .github\workflows\matrix-ci.yml
notepad .gitignore
notepad README.md
npm install
npm test
git init
git status
git add .
git commit -m "Add Node.js matrix testing workflow"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/node-matrix-demo.git
git push -u origin main
```

Important:

Replace:

```text
YOUR-USERNAME
```

with your actual GitHub username.

---

## 16. Full Command Sequence for Git Bash / Linux / macOS

```bash
mkdir node-matrix-demo
cd node-matrix-demo
npm init -y
touch index.js test.js .gitignore README.md
mkdir -p .github/workflows
touch .github/workflows/matrix-ci.yml
npm install
npm test
git init
git status
git add .
git commit -m "Add Node.js matrix testing workflow"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/node-matrix-demo.git
git push -u origin main
```

---

## 17. Common Errors and Solutions

| Error | Reason | Solution |
|---|---|---|
| Workflow not visible in Actions tab | File not inside `.github/workflows/` | Put `matrix-ci.yml` inside `.github/workflows/` |
| YAML error | Wrong indentation | Use spaces, not tabs |
| `npm test` fails | Test script missing | Add `"test": "node test.js"` in `package.json` |
| Same Node version used repeatedly | Matrix variable not used | Use `${{ matrix.node-version }}` |
| Remote origin already exists | Remote already added | Use `git remote -v` and update remote if needed |
| Repository not found | Wrong GitHub URL | Check username and repository name |

---

## 18. How to Fix Remote Origin Already Exists

Check existing remote:

```bash
git remote -v
```

Remove old remote:

```bash
git remote remove origin
```

Add correct remote:

```bash
git remote add origin https://github.com/YOUR-USERNAME/node-matrix-demo.git
```

Push again:

```bash
git push -u origin main
```

---

## 19. Important Student Observation

Students should observe that GitHub Actions creates three separate jobs automatically from one job definition.

This is the power of matrix strategy.

Instead of writing this manually:

```text
Job for Node.js 16
Job for Node.js 18
Job for Node.js 20
```

we write one matrix job:

```yaml
matrix:
  node-version: [16, 18, 20]
```

---

## 20. Student Submission Checklist

Students should submit:

1. GitHub repository link.
2. Screenshot of project directory structure.
3. Screenshot of `matrix-ci.yml` file.
4. Screenshot of successful GitHub Actions run.
5. Short explanation of:
   - Trigger
   - Job
   - Runner
   - Step
   - Matrix strategy
6. One error faced and how it was fixed.

---

## 21. Viva Questions with Simple Answers

### Q1. What is the purpose of this workflow?

It automatically tests the Node.js project using GitHub Actions.

### Q2. What is matrix strategy?

Matrix strategy runs the same job multiple times with different values.

### Q3. Which Node.js versions are used here?

Node.js 16, 18, and 20.

### Q4. What is the runner used here?

The runner is `ubuntu-latest`.

### Q5. What does `actions/checkout@v4` do?

It downloads the repository code into the GitHub Actions runner.

### Q6. What does `actions/setup-node@v4` do?

It installs the required Node.js version in the runner.

### Q7. What command runs the tests?

```bash
npm test
```

### Q8. Why should we not upload `node_modules`?

Because it is very large and can be recreated using `npm install`.

### Q9. What happens after code is pushed?

The workflow starts automatically because the `push` trigger is configured.

### Q10. Why is matrix testing useful?

It helps verify that the same code works correctly on multiple versions of a language or tool.

---

## 22. Final Learning Outcome

After completing this practical, students will understand how to:

- Create a Node.js project.
- Write a simple test file.
- Create a GitHub Actions workflow.
- Use matrix strategy.
- Test code on Node.js 16, 18, and 20.
- Debug common GitHub Actions workflow errors.

