# Scenario 1: Correct Basic Node.js CI Pipeline Using GitHub Actions

## Title

**Implementation of a Correct Basic Continuous Integration (CI) Workflow for a Node.js Project Using GitHub Actions**

---

## 1. Purpose of This Scenario

This scenario demonstrates how to create a simple Node.js project and run it automatically using GitHub Actions.

The project will contain:

1. `index.js` — main JavaScript file containing an `add()` function.
2. `test.js` — test file to check whether the function works correctly.
3. `package.json` — Node.js project configuration file.
4. `.github/workflows/node-ci.yml` — GitHub Actions workflow file.

Whenever code is pushed to GitHub, GitHub Actions will automatically run the test.

---

## 2. Final Correct Project Structure

At the end, your project should look exactly like this:

```text
node-ci-demo/
│
├── index.js
├── test.js
├── package.json
│
└── .github/
    └── workflows/
        └── node-ci.yml
```

---

## 3. Very Important Beginner Note

Do **not** type the GitHub repository URL directly in CMD.

### Wrong command

```cmd
https://github.com/alokmisra/node-ci-demo.git
```

This will give this error:

```text
'https:' is not recognized as an internal or external command
```

### Correct command

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

A URL must be used with a Git command. A URL alone is not a Windows CMD command.

---

# 4. Exact Commands from Scratch

Open **Command Prompt** and run the following commands line by line.

---

## Step 1: Go to Your Working Folder

```cmd
cd C:\Users\Mamta\unit5\scenario1
```

### Explanation

This command moves you to the folder where you want to create the project.

---

## Step 2: Create Project Folder

```cmd
mkdir node-ci-demo
```

### Explanation

This creates a new folder named `node-ci-demo`.

---

## Step 3: Move Inside Project Folder

```cmd
cd node-ci-demo
```

### Explanation

This moves CMD inside the newly created project folder.

---

## Step 4: Initialize Node.js Project

```cmd
npm init -y
```

### Explanation

This creates a `package.json` file automatically using default values.

---

# 5. Create Correct Project Files

You may create files using Notepad, VS Code, or CMD.

The easiest method is to use VS Code:

```cmd
code .
```

If VS Code does not open, create files manually using Notepad.

---

## Step 5: Create `index.js`

Create a file named:

```text
index.js
```

Paste this exact code:

```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

### Explanation

This file creates a function named `add`.

```javascript
function add(a, b) {
  return a + b;
}
```

This function adds two numbers.

```javascript
module.exports = add;
```

This exports the function so that another file can import and use it.

This line is very important. Without it, you may get this error:

```text
TypeError: add is not a function
```

---

## Step 6: Create `test.js`

Create a file named:

```text
test.js
```

Paste this exact code:

```javascript
const add = require("./index");

if (add(2, 3) === 5) {
  console.log("Test passed: add(2, 3) is equal to 5");
} else {
  throw new Error("Test failed: add(2, 3) is not equal to 5");
}
```

### Explanation

```javascript
const add = require("./index");
```

This imports the `add` function from `index.js`.

```javascript
if (add(2, 3) === 5)
```

This checks whether `2 + 3` gives `5`.

If the answer is correct, the test passes.

If the answer is wrong, the test fails.

---

## Step 7: Correct `package.json`

Open `package.json`.

Replace the complete content with this exact code:

```json
{
  "name": "node-ci-demo",
  "version": "1.0.0",
  "description": "Basic Node.js CI example using GitHub Actions",
  "main": "index.js",
  "scripts": {
    "test": "node test.js"
  },
  "keywords": [],
  "author": "alokmisra",
  "license": "ISC"
}
```

### Explanation

This section is very important:

```json
"scripts": {
  "test": "node test.js"
}
```

When you run:

```cmd
npm test
```

Node.js will execute:

```cmd
node test.js
```

---

# 6. Test Project Locally

Before pushing to GitHub, test the project locally.

```cmd
npm test
```

Expected output:

```text
> node-ci-demo@1.0.0 test
> node test.js

Test passed: add(2, 3) is equal to 5
```

If you get:

```text
TypeError: add is not a function
```

then check that your `index.js` has this exact line:

```javascript
module.exports = add;
```

and your `test.js` has this exact line:

```javascript
const add = require("./index");
```

---

# 7. Initialize Git and Configure Identity

## Step 8: Initialize Git Repository

```cmd
git init
```

### Explanation

This starts Git tracking in your project folder.

---

## Step 9: Set Git Username

```cmd
git config --global user.name "alokmisra"
```

### Explanation

This sets your Git author name globally.

---

## Step 10: Set Git Email

```cmd
git config --global user.email "alokalokmm@gmail.com"
```

### Explanation

This sets your Git email globally.

---

## Step 11: Verify Git Identity

```cmd
git config --global user.name
git config --global user.email
```

Expected output:

```text
alokmisra
alokalokmm@gmail.com
```

---

# 8. Create GitHub Actions Workflow

## Step 12: Create `.github` Folder

```cmd
mkdir .github
```

### Explanation

This folder is used for GitHub configuration.

---

## Step 13: Create `workflows` Folder

```cmd
mkdir .github\workflows
```

### Explanation

All GitHub Actions workflow files must be inside:

```text
.github/workflows/
```

On Windows CMD, use backslash:

```cmd
.github\workflows
```

---

## Step 14: Create Workflow File

Create this file:

```text
.github/workflows/node-ci.yml
```

In Windows CMD, you can create it using Notepad:

```cmd
notepad .github\workflows\node-ci.yml
```

Paste this exact workflow code:

```yaml
name: Basic Node.js CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Setup Node.js environment
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Run test cases
        run: npm test
```

Save and close Notepad.

---

# 9. Explanation of Workflow File

## 9.1 Workflow Name

```yaml
name: Basic Node.js CI
```

This is the workflow name that appears in the GitHub Actions tab.

---

## 9.2 Trigger

```yaml
on:
  push:
```

This means the workflow will run whenever code is pushed to GitHub.

---

## 9.3 Job

```yaml
jobs:
  test:
```

This creates one job named `test`.

---

## 9.4 Runner

```yaml
runs-on: ubuntu-latest
```

This means the job will run on a temporary Ubuntu machine provided by GitHub.

---

## 9.5 Checkout Action

```yaml
uses: actions/checkout@v4
```

This downloads your GitHub repository code into the runner.

`@v4` means version 4 of the checkout action.

Without this step, the runner will not find your files such as `package.json`, `index.js`, and `test.js`.

---

## 9.6 Setup Node.js Action

```yaml
uses: actions/setup-node@v4
with:
  node-version: 20
```

This installs Node.js version 20 on the GitHub runner.

---

## 9.7 Install Dependencies

```yaml
run: npm install
```

This installs project dependencies.

Even if this project has no external dependency, this step is useful because real-world Node.js projects usually need dependencies.

---

## 9.8 Run Tests

```yaml
run: npm test
```

This runs the test script written in `package.json`.

---

# 10. Commit the Project

## Step 15: Add All Files to Git

```cmd
git add .
```

### Explanation

This stages all project files for commit.

---

## Step 16: Create Commit

```cmd
git commit -m "Initial Node.js project setup"
```

### Explanation

This saves your project snapshot in Git.

---

## Step 17: Rename Branch to Main

```cmd
git branch -M main
```

### Explanation

This renames the current branch to `main`.

---

# 11. Create GitHub Repository

Before pushing, create an empty repository on GitHub.

1. Open GitHub in browser.
2. Login using username:

```text
alokmisra
```

3. Click **New repository**.
4. Repository name must be:

```text
node-ci-demo
```

5. Do **not** add README.
6. Do **not** add `.gitignore`.
7. Do **not** add license.
8. Click **Create repository**.

---

# 12. Connect Local Project with GitHub

## Case A: If Remote Does Not Exist

Run:

```cmd
git remote add origin https://github.com/alokmisra/node-ci-demo.git
```

---

## Case B: If You Already Added Wrong Remote

If you previously added this wrong remote:

```text
https://github.com/YOUR-USERNAME/node-ci-demo.git
```

then run:

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

---

## Step 18: Verify Remote URL

```cmd
git remote -v
```

Expected output:

```text
origin  https://github.com/alokmisra/node-ci-demo.git (fetch)
origin  https://github.com/alokmisra/node-ci-demo.git (push)
```

If you still see:

```text
YOUR-USERNAME
```

then fix it again using:

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

---

# 13. Push Code to GitHub

```cmd
git push -u origin main
```

### Explanation

This uploads your local project to GitHub.

During push, Git may ask you to authenticate in the browser.

Complete the browser authentication.

---

# 14. Check GitHub Actions Result

After pushing:

1. Open repository:

```text
https://github.com/alokmisra/node-ci-demo
```

2. Click **Actions**.
3. Click **Basic Node.js CI**.
4. Open the latest run.
5. Click the `test` job.
6. Check the logs.

Expected successful output:

```text
Test passed: add(2, 3) is equal to 5
```

You should see a green tick mark.

---

# 15. Complete Correct Command Sequence

Use this complete sequence if you are starting fresh:

```cmd
cd C:\Users\Mamta\unit5\scenario1
mkdir node-ci-demo
cd node-ci-demo
npm init -y
notepad index.js
notepad test.js
notepad package.json
npm test
git init
git config --global user.name "alokmisra"
git config --global user.email "alokalokmm@gmail.com"
mkdir .github
mkdir .github\workflows
notepad .github\workflows\node-ci.yml
git add .
git commit -m "Initial Node.js project setup"
git branch -M main
git remote add origin https://github.com/alokmisra/node-ci-demo.git
git remote -v
git push -u origin main
```

---

# 16. Correct Code Summary

## 16.1 `index.js`

```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

---

## 16.2 `test.js`

```javascript
const add = require("./index");

if (add(2, 3) === 5) {
  console.log("Test passed: add(2, 3) is equal to 5");
} else {
  throw new Error("Test failed: add(2, 3) is not equal to 5");
}
```

---

## 16.3 `package.json`

```json
{
  "name": "node-ci-demo",
  "version": "1.0.0",
  "description": "Basic Node.js CI example using GitHub Actions",
  "main": "index.js",
  "scripts": {
    "test": "node test.js"
  },
  "keywords": [],
  "author": "alokmisra",
  "license": "ISC"
}
```

---

## 16.4 `.github/workflows/node-ci.yml`

```yaml
name: Basic Node.js CI

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Setup Node.js environment
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Run test cases
        run: npm test
```

---

# 17. Common Errors and Fixes

## Error 1: `TypeError: add is not a function`

### Cause

`index.js` is not exporting the function correctly.

### Fix

Use this in `index.js`:

```javascript
module.exports = add;
```

Use this in `test.js`:

```javascript
const add = require("./index");
```

---

## Error 2: `Author identity unknown`

### Cause

Git username and email are not configured.

### Fix

```cmd
git config --global user.name "alokmisra"
git config --global user.email "alokalokmm@gmail.com"
```

---

## Error 3: `remote origin already exists`

### Cause

Remote URL was already added earlier.

### Fix

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

---

## Error 4: Repository Not Found

### Cause 1

GitHub repository was not created.

### Fix

Create repository named:

```text
node-ci-demo
```

under GitHub username:

```text
alokmisra
```

### Cause 2

Wrong remote URL was used.

### Fix

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

---

## Error 5: `'https:' is not recognized as an internal or external command`

### Cause

You typed the GitHub URL directly in CMD.

Wrong:

```cmd
https://github.com/alokmisra/node-ci-demo.git
```

Correct:

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

---

## Error 6: Workflow Not Visible in GitHub Actions

### Cause

Workflow file is not inside the correct folder.

### Correct Path

```text
.github/workflows/node-ci.yml
```

---

## Error 7: `npm test` fails in GitHub Actions

### Cause

The local test may already be failing.

### Fix

Always run locally first:

```cmd
npm test
```

Only push after local test passes.

---

# 18. Demonstrating CI Failure Intentionally

To show students how CI detects bugs, edit `index.js`.

Change:

```javascript
return a + b;
```

to:

```javascript
return a - b;
```

Now run:

```cmd
npm test
```

Expected result:

```text
Error: Test failed: add(2, 3) is not equal to 5
```

Now commit and push:

```cmd
git add .
git commit -m "Introduce test failure for CI demonstration"
git push
```

GitHub Actions will fail and show a red cross.

---

# 19. Fixing the CI Failure

Change `index.js` back to:

```javascript
function add(a, b) {
  return a + b;
}

module.exports = add;
```

Run:

```cmd
npm test
```

If test passes, commit and push:

```cmd
git add .
git commit -m "Fix add function and restore passing CI"
git push
```

GitHub Actions should pass again.

---

# 20. Final Learning Outcome

After completing this scenario, you should understand:

1. How to create a basic Node.js project.
2. How to write and run a simple test.
3. How to configure Git.
4. How to push code to GitHub.
5. How to create a GitHub Actions workflow.
6. How to trigger workflow on push.
7. How to debug common Git, Node.js, and GitHub Actions errors.
8. How CI automatically validates code changes.

---

# 21. Short Viva Questions

## Q1. What is CI?

CI means Continuous Integration. It automatically checks code whenever developers push changes.

## Q2. What does `on: push` mean?

It means the workflow runs whenever code is pushed to GitHub.

## Q3. What is `actions/checkout@v4`?

It is a GitHub Action that downloads repository code into the runner.

## Q4. What does `@v4` mean?

It means version 4 of the action.

## Q5. What is a runner?

A runner is a machine that executes the workflow job.

## Q6. Why do we use `npm test`?

It runs the test script defined in `package.json`.

## Q7. Why should we run `npm test` locally before pushing?

Because if it fails locally, it will also fail in GitHub Actions.

## Q8. Why should GitHub Secrets be used in real projects?

Secrets protect sensitive information such as passwords, tokens, and SSH keys.

---

# 22. Student Submission Requirements

Students should submit:

1. GitHub repository link.
2. Screenshot of local `npm test` success.
3. Screenshot of correct project structure.
4. Screenshot of `node-ci.yml`.
5. Screenshot of GitHub Actions successful run.
6. Explanation of each workflow step.
7. Screenshot of one intentional failure and final fix.

---

# 23. Evaluation Rubric

| Parameter | Marks |
|---|---:|
| Correct Node.js project creation | 2 |
| Correct `index.js` file | 2 |
| Correct `test.js` file | 2 |
| Correct `package.json` test script | 2 |
| Correct Git configuration and commit | 2 |
| Correct GitHub remote URL | 2 |
| Correct workflow folder structure | 2 |
| Correct workflow YAML | 3 |
| Successful GitHub Actions execution | 3 |
| Explanation of errors and fixes | 2 |
| **Total** | **22** |

---

# 24. Final Conclusion

This scenario provides a complete beginner-friendly implementation of a basic Node.js CI pipeline using GitHub Actions.

The most important points are:

1. Use the correct function export:

```javascript
module.exports = add;
```

2. Use the correct import:

```javascript
const add = require("./index");
```

3. Use the correct GitHub remote:

```cmd
git remote set-url origin https://github.com/alokmisra/node-ci-demo.git
```

4. Use the correct workflow location:

```text
.github/workflows/node-ci.yml
```

5. Test locally before pushing:

```cmd
npm test
```

If local testing passes, GitHub Actions will also pass unless the workflow file has a syntax or environment issue.
