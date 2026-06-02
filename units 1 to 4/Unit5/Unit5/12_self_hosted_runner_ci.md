# Scenario 12: Self Hosted Runner CI

## 1. Scenario

You are working on a project named `self-hosted-runner-demo`. The task is to implement GitHub Actions for **self-hosted runner**. This scenario focuses on **internal company machine** and helps students understand a real CI/CD use case through actual YAML implementation.

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
self-hosted-runner-demo/
└── .github/
    └── workflows/
        └── self-hosted.yml
```

---


## Runner Setup

In GitHub:

```text
Repository → Settings → Actions → Runners → New self-hosted runner
```

GitHub will provide commands similar to:

```bash
mkdir actions-runner
cd actions-runner
curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/...
tar xzf ./actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/username/repository --token TOKEN
./run.sh
```

## Workflow File

```yaml
name: Self Hosted Runner CI

on:
  push:

jobs:
  internal-build:
    runs-on: self-hosted

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Display runner machine name
        run: hostname

      - name: Check installed tools
        run: |
          node -v || true
          docker --version || true

      - name: Run internal build
        run: echo "Build is running on company self-hosted runner"
```

## Step-by-Step Implementation

1. Configure self-hosted runner from GitHub settings.
2. Keep the runner active using `./run.sh` or install it as a service.
3. Add workflow file.
4. Push code.
5. Check whether the job is picked by the self-hosted machine.

## Expected Output

```text
Build is running on company self-hosted runner
```

## Explanation

`runs-on: self-hosted` tells GitHub Actions to run the job on your own machine instead of GitHub's virtual machine.

## Security Note

Do not run untrusted pull request code on sensitive self-hosted runners because malicious code may access the runner system.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Runner not running | Job remains queued | Start runner |
| Wrong label | Job not assigned | Use correct runner label |
| Missing tools | Build fails | Install required tools |


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
