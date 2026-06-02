# Scenario 11: Remote Linux Server Deployment Using SSH

## 1. Scenario

You are working on a project named `ssh-deploy-demo`. The task is to implement GitHub Actions for **SSH deployment**. This scenario focuses on **remote server** and helps students understand a real CI/CD use case through actual YAML implementation.

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
ssh-deploy-demo/
└── .github/
    └── workflows/
        └── ssh-deploy.yml
```

---


## Required Secrets

Create these GitHub Actions secrets:

```text
SERVER_HOST
SERVER_USER
SERVER_SSH_KEY
```

## Server Preparation

On your Linux server:

```bash
cd /var/www
git clone https://github.com/your-username/ssh-deploy-demo.git
cd ssh-deploy-demo
npm install
npm install -g pm2
pm2 start index.js --name ssh-deploy-demo
```

## Workflow File

```yaml
name: Deploy to Linux Server

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy application using SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /var/www/ssh-deploy-demo
            git pull origin main
            npm install
            pm2 restart ssh-deploy-demo
```

## Step-by-Step Implementation

1. Generate SSH key locally:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy"
```

2. Copy public key to server `~/.ssh/authorized_keys`.
3. Copy private key content to GitHub secret `SERVER_SSH_KEY`.
4. Add server IP/host as `SERVER_HOST`.
5. Add Linux username as `SERVER_USER`.
6. Push to `main`.

## Expected Output

```text
Successfully connected to server
git pull origin main
pm2 restart ssh-deploy-demo
Deployment completed
```

## Common Error

```text
Permission denied (publickey)
```

## Fix

| Cause | Fix |
|---|---|
| Wrong private key | Add correct private key to secret |
| Public key not on server | Add public key to `authorized_keys` |
| Wrong server user | Use correct Linux username |
| SSH blocked | Check firewall/security group |


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
