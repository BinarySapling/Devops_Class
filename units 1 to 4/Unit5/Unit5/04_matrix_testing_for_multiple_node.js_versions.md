# Scenario 04: Matrix Testing for Multiple Node.js Versions

## 1. Scenario

You are working on a project named `node-matrix-demo`. The task is to implement GitHub Actions for **matrix strategy**. This scenario focuses on **Node.js 16, 18, and 20** and helps students understand a real CI/CD use case through actual YAML implementation.

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
node-matrix-demo/
└── .github/
    └── workflows/
        └── matrix-ci.yml
```

---


## Source Code Required

Use any Node.js project with a `package.json` and test script.

## Workflow File

```yaml
name: Node.js Matrix Testing

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16, 18, 20]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm install

      - name: Run tests on Node.js ${{ matrix.node-version }}
        run: npm test
```

## Step-by-Step Implementation

1. Add the workflow under `.github/workflows/matrix-ci.yml`.
2. Push the code.
3. Open the GitHub Actions tab.
4. Notice that GitHub creates three job executions automatically.

## Expected Output

```text
Run tests on Node.js 16 - success
Run tests on Node.js 18 - success
Run tests on Node.js 20 - success
```

## Explanation

Matrix strategy avoids writing repeated jobs. One job definition is expanded into multiple jobs using the values in `matrix.node-version`.

## Practical Use Case

A package maintainer can test whether the library works across older and newer Node.js versions.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Forgetting matrix variable | Same Node version used | Use `${{ matrix.node-version }}` |
| Unsupported version | Setup fails | Use supported Node versions |
| Wrong indentation under `strategy` | Workflow fails | Check YAML indentation |


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
