# Scenario 02: Pull Request CI Validation

## 1. Scenario

A team does not want untested code to be merged into the `main` branch. Whenever a developer creates or updates a pull request targeting `main`, GitHub Actions should automatically run tests.

This helps maintain code quality before merging.

---

## 2. Learning Objectives

Students will learn to:

- Use the `pull_request` trigger.
- Validate code before merging.
- Understand branch protection use cases.
- Run CI checks on pull requests.

---

## 3. Project Structure

```text
pr-validation-demo/
├── package.json
├── app.js
├── test.js
└── .github/
    └── workflows/
        └── pr-ci.yml
```

---

## 4. Source Code

### `app.js`

```javascript
function multiply(a, b) {
  return a * b;
}

module.exports = multiply;
```

### `test.js`

```javascript
const multiply = require("./app");

if (multiply(4, 5) === 20) {
  console.log("Pull request validation test passed");
} else {
  throw new Error("Pull request validation test failed");
}
```

### `package.json`

```json
{
  "name": "pr-validation-demo",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js"
  }
}
```

---

## 5. Workflow File

Create:

```text
.github/workflows/pr-ci.yml
```

```yaml
name: Pull Request Validation CI

on:
  pull_request:
    branches:
      - main

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout pull request code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install project dependencies
        run: npm install

      - name: Run automated tests
        run: npm test
```

---

## 6. Implementation Steps

### Step 1: Create repository and add files

Add `app.js`, `test.js`, and `package.json`.

### Step 2: Add workflow

Create `.github/workflows/pr-ci.yml` and paste YAML.

### Step 3: Push initial code to `main`

```bash
git add .
git commit -m "Add PR validation workflow"
git push origin main
```

### Step 4: Create feature branch

```bash
git checkout -b feature-multiply-change
```

### Step 5: Make a small change and push

```bash
git add .
git commit -m "Update multiplication logic"
git push origin feature-multiply-change
```

### Step 6: Create pull request

On GitHub, create a pull request from:

```text
feature-multiply-change → main
```

### Step 7: Observe checks

GitHub will show CI status on the pull request page.

---

## 7. Expected Output

```text
Pull request validation test passed
```

The pull request check should pass.

---

## 8. Explanation

The workflow runs only when a pull request targets the `main` branch. This ensures that before any code enters the production branch, automated validation happens.

---

## 9. Common Errors and Fixes

| Error | Reason | Fix |
|---|---|---|
| Workflow not running on PR | Wrong target branch | Ensure PR targets `main` |
| Workflow runs on push but not PR | Missing `pull_request` trigger | Add `pull_request` trigger |
| Test fails | Wrong logic in code | Correct code or test case |

---

## 10. Student Tasks

1. Create one passing pull request.
2. Create one intentionally failing pull request.
3. Compare both workflow outputs.

---

## 11. Viva Questions

1. What is a pull request?
2. Why should CI run before merging?
3. Difference between `push` and `pull_request` triggers?
4. What happens if a test fails in PR?
