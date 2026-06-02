# Corrected Implementation Guide: GitHub Actions Matrix Testing for Multiple Node.js Versions

**Project Name:** `node-matrix-demo`  
**Topic:** Continuous Integration with GitHub Actions Matrix Strategy  
**Author:** Alok Misra, Professor DevOps, Lovely Professional University  

---

## 1. Purpose of This Practical

This practical teaches how to run the same Node.js test cases on multiple Node.js versions using GitHub Actions.

In real software projects, developers must check whether their code works on different versions of Node.js. Instead of writing three separate jobs manually, GitHub Actions provides a feature called **matrix strategy**.

In this project, the workflow will automatically test the project on:

- Node.js 16
- Node.js 18
- Node.js 20

---

## 2. Error Fixed in This Corrected Version

You got this error:

```text
/home/runner/.npm
Error: Dependencies lock file is not found in /home/runner/work/node-matrix-demo/node-matrix-demo.
Supported file patterns: package-lock.json,npm-shrinkwrap.json,yarn.lock
```

### Why This Error Occurred

The workflow was using:

```yaml
cache: npm
```

and/or:

```bash
npm ci
```

Both are good practices, but they require a lock file.

For an npm project, the required lock file is:

```text
package-lock.json
```

The earlier project did not contain `package-lock.json`, so GitHub Actions failed.

### Correct Solution

This corrected project includes:

```text
package-lock.json
```

Now the workflow can safely use:

```yaml
cache: npm
```

and:

```bash
npm ci
```

---

## 3. Final Directory Structure

Your project must look exactly like this:

```text
node-matrix-demo/
├── .github/
│   └── workflows/
│       └── matrix-ci.yml
├── .gitignore
├── index.js
├── package.json
├── package-lock.json
├── README.md
└── test.js
```

---

## 4. File 1: `package.json`

Location:

```text
node-matrix-demo/package.json
```

Content:

```json
{
  "name": "node-matrix-demo",
  "version": "1.0.0",
  "description": "A simple Node.js project to demonstrate GitHub Actions matrix testing across Node.js 16, 18, and 20.",
  "main": "index.js",
  "scripts": {
    "test": "node test.js",
    "start": "node index.js"
  },
  "keywords": [
    "github-actions",
    "ci",
    "matrix",
    "nodejs"
  ],
  "author": "Alok Misra",
  "license": "MIT",
  "dependencies": {}
}
```

### Explanation

Important part:

```json
"test": "node test.js"
```

This means when GitHub Actions runs:

```bash
npm test
```

it will execute:

```bash
node test.js
```

---

## 5. File 2: `package-lock.json`

Location:

```text
node-matrix-demo/package-lock.json
```

This file is generated automatically by running:

```bash
npm install
```

or:

```bash
npm install --package-lock-only
```

### Why This File Is Important

This file locks the dependency versions. Even if the project has no external dependency, this file is still useful because GitHub Actions caching and `npm ci` expect it.

Do not delete this file.

---

## 6. File 3: `index.js`

Location:

```text
node-matrix-demo/index.js
```

Content:

```javascript
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error("Division by zero is not allowed");
  }
  return a / b;
}

module.exports = {
  add,
  subtract,
  multiply,
  divide
};
```

### Explanation

This file contains four simple functions:

- `add()`
- `subtract()`
- `multiply()`
- `divide()`

These functions are exported using:

```javascript
module.exports
```

This allows `test.js` to import and test them.

---

## 7. File 4: `test.js`

Location:

```text
node-matrix-demo/test.js
```

Content:

```javascript
const assert = require("assert");
const { add, subtract, multiply, divide } = require("./index");

console.log("Running unit tests...");

assert.strictEqual(add(2, 3), 5, "2 + 3 should be 5");
assert.strictEqual(subtract(10, 4), 6, "10 - 4 should be 6");
assert.strictEqual(multiply(3, 4), 12, "3 * 4 should be 12");
assert.strictEqual(divide(20, 5), 4, "20 / 5 should be 4");
assert.throws(() => divide(10, 0), /Division by zero/, "Division by zero should throw an error");

console.log("All tests passed successfully!");
```

### Explanation

This file tests the functions written in `index.js`.

Example:

```javascript
assert.strictEqual(add(2, 3), 5);
```

This checks whether `add(2, 3)` gives the correct answer `5`.

If all tests pass, the output will be:

```text
All tests passed successfully!
```

---

## 8. File 5: `.github/workflows/matrix-ci.yml`

Location:

```text
node-matrix-demo/.github/workflows/matrix-ci.yml
```

Content:

```yaml
name: Node.js Matrix Testing

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

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

      - name: Install dependencies using lock file
        run: npm ci

      - name: Run tests on Node.js ${{ matrix.node-version }}
        run: npm test
```

### Explanation of Workflow

#### `name`

```yaml
name: Node.js Matrix Testing
```

This is the name shown in the GitHub Actions tab.

#### `on`

```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```

This workflow runs when code is pushed to the `main` branch or when a pull request is created for the `main` branch.

#### `runs-on`

```yaml
runs-on: ubuntu-latest
```

This means GitHub will run the job on a Linux virtual machine.

#### `matrix`

```yaml
matrix:
  node-version: [16, 18, 20]
```

This is the main concept. GitHub Actions will create three jobs automatically:

```text
Test on Node.js 16
Test on Node.js 18
Test on Node.js 20
```

#### `actions/checkout@v4`

This downloads your repository code into the GitHub Actions runner.

#### `actions/setup-node@v4`

This installs the selected Node.js version.

#### `cache: npm`

This caches npm packages and makes future builds faster.

#### `npm ci`

This installs dependencies exactly according to `package-lock.json`.

#### `npm test`

This runs the test command from `package.json`.

---

## 9. File 6: `.gitignore`

Location:

```text
node-matrix-demo/.gitignore
```

Content:

```text
node_modules/
npm-debug.log
.DS_Store
.env
```

### Explanation

This file tells Git not to upload unnecessary files.

Most important:

```text
node_modules/
```

The `node_modules` folder is very large and should not be pushed to GitHub.

---

## 10. File 7: `README.md`

Location:

```text
node-matrix-demo/README.md
```

Content:

```markdown
# Node Matrix Demo

This project demonstrates GitHub Actions matrix testing for a simple Node.js application.

The workflow runs the same tests on three Node.js versions:

- Node.js 16
- Node.js 18
- Node.js 20

## Run locally

```bash
npm install
npm test
```

## GitHub Actions

Workflow file:

```text
.github/workflows/matrix-ci.yml
```

The workflow uses:

- `actions/checkout@v4`
- `actions/setup-node@v4`
- `cache: npm`
- `npm ci`
- matrix strategy for multiple Node.js versions
```

---

## 11. Exact Commands for Windows CMD

Open **Command Prompt** and run these commands one by one.

### Step 1: Go to your working folder

Example:

```cmd
cd C:\Users\Mamta\devops\unit5
```

If your folder is different, use your actual folder path.

---

### Step 2: Create the project folder

```cmd
mkdir node-matrix-demo
cd node-matrix-demo
```

---

### Step 3: Initialize Node.js project

```cmd
npm init -y
```

This creates `package.json`.

---

### Step 4: Create `package-lock.json`

```cmd
npm install --package-lock-only
```

This creates the important file:

```text
package-lock.json
```

This step fixes the GitHub Actions error related to the missing dependency lock file.

---

### Step 5: Create project files manually

Create these files in VS Code:

```text
index.js
test.js
README.md
.gitignore
.github/workflows/matrix-ci.yml
```

To create the workflow folder using CMD, run:

```cmd
mkdir .github
mkdir .github\workflows
```

Then create the file:

```text
.github/workflows/matrix-ci.yml
```

Copy the file contents from the sections above.

---

### Step 6: Test locally

```cmd
npm test
```

Expected output:

```text
Running unit tests...
All tests passed successfully!
```

---

### Step 7: Initialize Git

```cmd
git init
```

---

### Step 8: Check files

```cmd
git status
```

You should see files such as:

```text
.github/workflows/matrix-ci.yml
index.js
package.json
package-lock.json
test.js
README.md
.gitignore
```

---

### Step 9: Add files to Git

```cmd
git add .
```

---

### Step 10: Commit files

```cmd
git commit -m "Add Node.js matrix testing workflow"
```

---

### Step 11: Create a new GitHub repository

Go to GitHub and create a new repository named:

```text
node-matrix-demo
```

Do not add README, `.gitignore`, or license from GitHub because files already exist locally.

---

### Step 12: Connect local project to GitHub

Replace `YOUR-USERNAME` with your actual GitHub username.

Example for username `alokmisra`:

```cmd
git remote add origin https://github.com/alokmisra/node-matrix-demo.git
```

If your username is different, use:

```cmd
git remote add origin https://github.com/YOUR-USERNAME/node-matrix-demo.git
```

---

### Step 13: Rename branch to `main`

```cmd
git branch -M main
```

---

### Step 14: Push code to GitHub

```cmd
git push -u origin main
```

---

### Step 15: Check GitHub Actions

1. Open your GitHub repository.
2. Click **Actions**.
3. Click **Node.js Matrix Testing**.
4. You should see three jobs:

```text
Test on Node.js 16
Test on Node.js 18
Test on Node.js 20
```

Each job should show a green tick after successful execution.

---

## 12. Exact Commands If Repository Already Exists

If you already pushed the old version and only want to fix the error, run these commands inside the existing project folder:

```cmd
cd C:\Users\Mamta\devops\unit5\node-matrix-demo
npm install --package-lock-only
git add package-lock.json package.json .github\workflows\matrix-ci.yml
git commit -m "Fix GitHub Actions lock file error"
git push origin main
```

Then open the GitHub Actions tab again.

---

## 13. Expected GitHub Actions Output

You should see output similar to this:

```text
Test on Node.js 16 - success
Test on Node.js 18 - success
Test on Node.js 20 - success
```

Inside each job, the steps should be:

```text
Checkout repository
Setup Node.js 16/18/20
Install dependencies using lock file
Run tests on Node.js 16/18/20
```

---

## 14. Common Mistakes and Fixes

| Mistake | Result | Fix |
|---|---|---|
| `package-lock.json` missing | Dependency lock file error | Run `npm install --package-lock-only` and push `package-lock.json` |
| Wrong YAML indentation | Workflow does not start | Use two spaces for indentation |
| Workflow file not inside `.github/workflows/` | GitHub Actions cannot detect workflow | Place file at `.github/workflows/matrix-ci.yml` |
| Using `npm ci` without lock file | Build fails | Add `package-lock.json` |
| Writing `node-version: matrix.node-version` | Node version not detected | Use `${{ matrix.node-version }}` |
| Not pushing to `main` branch | Workflow may not run | Push to `main` branch |
| Repository remote URL is wrong | Push fails | Check with `git remote -v` |

---

## 15. Viva Questions with Simple Answers

### Q1. What is GitHub Actions?

GitHub Actions is a CI/CD tool provided by GitHub. It can automatically build, test, and deploy code.

### Q2. What is matrix strategy?

Matrix strategy allows the same job to run multiple times with different values. Here, the same test job runs on Node.js 16, 18, and 20.

### Q3. Why do we need `package-lock.json`?

It stores exact dependency versions. It is required when using `npm ci` and useful when using npm caching.

### Q4. What is a runner?

A runner is a machine where GitHub Actions executes the workflow. In this project, the runner is `ubuntu-latest`.

### Q5. What is `actions/checkout@v4`?

It downloads the repository code into the runner.

### Q6. What is `actions/setup-node@v4`?

It installs the required Node.js version on the runner.

### Q7. Why are there three jobs in GitHub Actions?

Because the matrix contains three Node.js versions: 16, 18, and 20.

---

## 16. Final Submission Checklist

Students should submit:

1. GitHub repository link.
2. Screenshot of `.github/workflows/matrix-ci.yml`.
3. Screenshot of successful GitHub Actions run.
4. Screenshot showing three matrix jobs.
5. Short explanation of matrix strategy.
6. Error encountered and how it was solved.

---

## 17. Final Important Note

For this corrected workflow, do not delete:

```text
package-lock.json
```

This file is required for stable CI execution with:

```bash
npm ci
```

and:

```yaml
cache: npm
```
