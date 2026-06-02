# Scenario 06: Multi Job Build Test Deploy Pipeline

## 1. Scenario

You are working on a project named `multi-job-demo`. The task is to implement GitHub Actions for **job dependency with needs**. This scenario focuses on **build, test, deploy** and helps students understand a real CI/CD use case through actual YAML implementation.

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
multi-job-demo/
└── .github/
    └── workflows/
        └── multi-job.yml
```

---


## Package Script Example

```json
{
  "scripts": {
    "build": "echo Build completed",
    "test": "echo Tests passed"
  }
}
```

## Workflow File

```yaml
name: Multi Job Build Test Deploy

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

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm install

      - name: Build application
        run: npm run build

  test:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: npm install

      - name: Run test cases
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: test

    steps:
      - name: Deploy application
        run: echo "Application deployed successfully"
```

## Step-by-Step Implementation

1. Create package scripts for `build` and `test`.
2. Add the workflow file.
3. Push to `main`.
4. Open Actions and observe job order.

## Expected Output

```text
Build application - success
Run test cases - success
Deploy application - success
```

## Explanation

By default, jobs can run in parallel. The `needs` keyword controls dependency. Here, testing waits for build, and deployment waits for testing.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Not using `needs` | Jobs may run in parallel | Add `needs` |
| Missing build script | Build fails | Add `npm run build` script |
| Deploying without testing | Poor CI/CD design | Make deploy depend on test |


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
