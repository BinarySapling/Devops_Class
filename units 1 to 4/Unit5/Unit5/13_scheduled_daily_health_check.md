# Scenario 13: Scheduled Daily Health Check

## 1. Scenario

You are working on a project named `scheduled-health-check-demo`. The task is to implement GitHub Actions for **cron schedule**. This scenario focuses on **daily midnight UTC** and helps students understand a real CI/CD use case through actual YAML implementation.

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
scheduled-health-check-demo/
└── .github/
    └── workflows/
        └── health-check.yml
```

---


## Workflow File

```yaml
name: Daily Health Check

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest

    steps:
      - name: Call application health endpoint
        run: |
          curl -f https://example.com/health || exit 1

      - name: Print success message
        run: echo "Application health check passed"
```

## Step-by-Step Implementation

1. Replace `https://example.com/health` with your real application health URL.
2. Add workflow file.
3. Push code.
4. Test manually using `Run workflow`.
5. Let it run automatically daily.

## Expected Output

```text
Application health check passed
```

## Explanation

`0 0 * * *` means the workflow runs every day at midnight UTC. `workflow_dispatch` is added so the teacher or student can test it manually.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Thinking cron uses local time | Wrong timing expectation | Remember GitHub uses UTC |
| Wrong URL | Curl fails | Use correct health endpoint |
| No manual trigger | Hard to test immediately | Add `workflow_dispatch` |


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
