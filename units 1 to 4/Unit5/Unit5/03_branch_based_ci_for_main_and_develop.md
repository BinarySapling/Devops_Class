# Scenario 03: Branch Based CI for Main and Develop

## 1. Scenario

You are working on a project named `branch-ci-demo`. The task is to implement GitHub Actions for **push branch filters**. This scenario focuses on **main and develop** and helps students understand a real CI/CD use case through actual YAML implementation.

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
branch-ci-demo/
└── .github/
    └── workflows/
        └── branch-ci.yml
```

---


## Source Code Required

This scenario can use the same simple Node.js files from Scenario 01.

## Workflow File

```yaml
name: Branch Based CI

on:
  push:
    branches:
      - main
      - develop

jobs:
  branch-check:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Display current branch
        run: echo "Workflow running on branch: ${{ github.ref_name }}"

      - name: Run basic validation
        run: echo "Code validation completed"
```

## Step-by-Step Implementation

1. Create the project and workflow file.
2. Push the workflow to `main`.
3. Create and push a `develop` branch:

```bash
git checkout -b develop
git push origin develop
```

4. Create and push a feature branch:

```bash
git checkout -b feature-login
git push origin feature-login
```

5. Observe that the workflow runs only on `main` and `develop`, not on `feature-login`.

## Expected Output

```text
Workflow running on branch: main
Code validation completed
```

## Explanation

The `branches` filter restricts workflow execution to selected branches. This is useful when teams do not want CI to run on every temporary branch.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Writing `branch` instead of `branches` | Workflow syntax issue | Use `branches` |
| Wrong branch name | Workflow does not run | Match exact branch names |
| Bad indentation | YAML error | Use two spaces |


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
