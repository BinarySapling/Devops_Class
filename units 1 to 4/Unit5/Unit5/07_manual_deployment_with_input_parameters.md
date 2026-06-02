# Scenario 07: Manual Deployment with Input Parameters

## 1. Scenario

You are working on a project named `manual-deploy-demo`. The task is to implement GitHub Actions for **workflow_dispatch inputs**. This scenario focuses on **dev staging production** and helps students understand a real CI/CD use case through actual YAML implementation.

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
manual-deploy-demo/
└── .github/
    └── workflows/
        └── manual-deploy.yml
```

---


## Workflow File

```yaml
name: Manual Environment Deployment

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Select deployment environment"
        required: true
        default: "dev"
        type: choice
        options:
          - dev
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Display selected environment
        run: echo "Selected environment is ${{ inputs.environment }}"

      - name: Deploy to selected environment
        run: |
          if [ "${{ inputs.environment }}" = "production" ]; then
            echo "Deploying to production server"
          elif [ "${{ inputs.environment }}" = "staging" ]; then
            echo "Deploying to staging server"
          else
            echo "Deploying to development server"
          fi
```

## Step-by-Step Implementation

1. Add the workflow file.
2. Push it to GitHub.
3. Go to repository `Actions` tab.
4. Select `Manual Environment Deployment`.
5. Click `Run workflow`.
6. Select `dev`, `staging`, or `production`.
7. Run and check logs.

## Expected Output

```text
Selected environment is staging
Deploying to staging server
```

## Explanation

`workflow_dispatch` makes the workflow manually executable. Inputs allow users to provide values during execution.

## Practical Use Case

Production deployment should not always happen automatically. Manual approval reduces accidental production releases.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Expecting auto trigger | Workflow only runs manually | Use Actions tab |
| Wrong input reference | Empty value | Use `${{ inputs.environment }}` |
| No choice options | Typing mistakes possible | Use `type: choice` |


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
