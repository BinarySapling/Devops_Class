# Scenario 05: Dependency Caching for Faster Node.js Builds

## 1. Scenario

You are working on a project named `node-cache-demo`. The task is to implement GitHub Actions for **dependency caching**. This scenario focuses on **npm cache** and helps students understand a real CI/CD use case through actual YAML implementation.

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
node-cache-demo/
└── .github/
    └── workflows/
        └── cache-ci.yml
```

---


## Source Code Required

Use a Node.js project containing `package-lock.json`.

## Workflow File

```yaml
name: Node.js CI with Cache

on:
  push:
  pull_request:

jobs:
  test-with-cache:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Setup Node.js with npm cache
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

## Step-by-Step Implementation

1. Run locally to create lock file:

```bash
npm install
```

2. Ensure `package-lock.json` exists.
3. Add the workflow.
4. Push once and observe dependency installation.
5. Push again and compare build time.

## Expected Output

First run:

```text
Cache not found
Installing dependencies
```

Later run:

```text
Cache restored
Installing dependencies
```

## Explanation

Caching stores dependency data so that future workflow runs are faster. `npm ci` is preferred in CI because it installs exactly according to `package-lock.json`.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| No `package-lock.json` | `npm ci` may fail | Generate lock file |
| Expecting first run to be fast | Cache not yet created | Run workflow once |
| Using changing dependencies every run | Cache less effective | Keep lock file stable |


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
