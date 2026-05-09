# Master Viva & Technical Placement Q&A Bank: GitHub Actions
## Target Audience: University Practical Exams & Software Engineering Recruitment Rounds

This comprehensive compilation consolidates high-yield questions, conceptual checkups, and advanced scenario-based questions from all five study days of the GitHub Actions syllabus. Use this guide for last-minute revision before practical exams or technical placement interviews.

---

## Category 1: Easy & Conceptual (Core Architecture & Syntax)

### Q1: What is GitHub Actions, and how does it fit into the DevOps lifecycle?
* **Answer**: GitHub Actions is a cloud-native, serverless automation platform integrated directly into GitHub. It enables developers to implement Continuous Integration and Continuous Deployment (CI/CD) pipelines by defining workflows as code inside their repositories, automating compilation, testing, and deployment processes.

### Q2: What is the exact directory and file naming convention for GitHub Actions?
* **Answer**: All workflow definitions must be written as YAML files (with `.yml` or `.yaml` extensions) and stored in the **`.github/workflows/`** directory located at the root of the repository. The directories `.github` and `workflows` are completely case-sensitive and must be named exactly as shown.

### Q3: Define the core hierarchical components of a GitHub Actions execution model.
* **Answer**:
  * **Event / Trigger**: An activity in the repository (e.g., code push, pull request, manual click) that initiates execution.
  * **Workflow**: The top-level automated process defined in a YAML file.
  * **Job**: A set of steps executed on a single runner. Multiple jobs run in parallel by default.
  * **Step**: An individual task within a job that runs sequentially. It can execute raw shell commands (`run`) or call an action (`uses`).
  * **Action**: A reusable, packaged automation component.
  * **Runner**: The underlying virtual machine or physical server running the jobs.

### Q4: Explain the difference between the `run` and `uses` keywords in a workflow step.
* **Answer**:
  * `run` is used to execute **command-line scripts or commands** inside the runner's native shell (e.g., bash on Linux/macOS, PowerShell on Windows).
  * `uses` is used to import and run a **reusable prepackaged action** (an external application) hosted on the GitHub Marketplace, a separate repository, or a local directory (e.g., `uses: actions/checkout@v4`).

### Q5: What is the purpose of the `actions/checkout` action? Why is it usually the first step?
* **Answer**: When a runner VM starts, its workspace directory is completely empty. `actions/checkout` checks out your repository's source code into the runner's workspace (`$GITHUB_WORKSPACE`), allowing subsequent steps (compilation, testing, building) to access and process the project files.

### Q6: Can a workflow have multiple triggers? How are they declared?
* **Answer**: Yes. You can define multiple triggers under the `on` block. For example:
  ```yaml
  on:
    push:
      branches: [main]
    pull_request:
      branches: [main]
    workflow_dispatch:
  ```
  This workflow triggers on pushes, pull requests, and manual triggers.

---

## Category 2: Medium & Scenario-Based (Triggers, Matrices & Performance)

### Q7: What is crontab scheduling syntax in GitHub Actions? Write a cron expression to execute a build every Sunday at midnight UTC.
* **Answer**: GitHub Actions uses POSIX crontab syntax consisting of 5 space-separated fields: `minute hour day_of_month month day_of_week`.
  To run every Sunday at midnight UTC:
  ```yaml
  on:
    schedule:
      - cron: '0 0 * * 0'
  ```
  *(Minute: 0, Hour: 0, DOM: *, Month: *, DOW: 0 for Sunday)*.

### Q8: Contrast `branches` filtering with `branches-ignore` inside a push event configuration. Can they be declared together?
* **Answer**:
  * `branches` defines a whitelist: only pushes to these branches trigger execution.
  * `branches-ignore` defines a blacklist: pushes to these branches are excluded from execution.
  * **No**, you cannot declare both `branches` and `branches-ignore` under the same event trigger in a workflow file. Attempting to do so will result in a syntax validation error. Use glob negation inside `branches` if exclusions are needed alongside inclusions.

### Q9: How do you configure jobs to execute sequentially instead of their default parallel execution?
* **Answer**: You use the **`needs`** keyword to define dependencies. For example, if `job2` must wait for `job1` to complete:
  ```yaml
  jobs:
    job1:
      runs-on: ubuntu-latest
    job2:
      needs: job1
      runs-on: ubuntu-latest
  ```

### Q10: How do you pass variable values or build metadata from one Job to another?
* **Answer**:
  1. In the source job (Job A), write the output value to the special environment file: `echo "key_name=value" >> "$GITHUB_OUTPUT"`.
  2. Declare the output variable at the Job A level under the `outputs` block.
  3. In the destination job (Job B), set `needs: JobA` and access the variable using: `${{ needs.JobA.outputs.key_name }}`.

### Q11: Explain how `actions/upload-artifact` differs from `actions/cache`. When should you use each?
* **Answer**:
  * **Artifacts (`upload-artifact` / `download-artifact`)**: Used to store physical files (e.g. binaries, zip archives, test coverage reports) generated by a job to share them with subsequent jobs or download them manually from the GitHub UI. Artifacts are fresh for every workflow run.
  * **Caching (`actions/cache`)**: Used to persist static dependencies (e.g. `node_modules`, `pip` packages, `maven` libraries) **across different workflow runs** to speed up dependency installation times. It retrieves files compiled in past executions using keys based on lockfiles.

### Q12: What is a Matrix Strategy? How do the `include` and `exclude` keys modify its behavior?
* **Answer**: A matrix strategy allows you to run a single job configuration across multiple combinations of variables (e.g., OS platforms, runtime versions).
  * `include` allows you to append additional specific combinations (e.g., adding an extra variable for a specific OS) or run a custom standalone combination outside the main grid.
  * `exclude` removes specific undesirable or incompatible combinations from the generated grid to save build minutes.

### Q13: What does the `fail-fast: false` strategy setting do?
* **Answer**: By default, if any job in a matrix strategy fails, GitHub immediately cancels all other active and queued matrix jobs in that run (`fail-fast: true`). Setting `fail-fast: false` ensures all matrix combinations execute to completion, regardless of individual job failures, which is essential for comprehensive multi-platform test suites.

---

## Category 3: Hard & Enterprise (Security, Runners & Container Deployments)

### Q14: Explain the network security architecture of Self-Hosted Runners. Why don't they require open inbound ports?
* **Answer**: Self-hosted runners utilize an **outbound-only pull architecture**. The runner agent software installed on the machine initiates a secure long-poll HTTPS connection (over port 443) to GitHub’s API servers. It keeps this connection open to listen for jobs. When a job is available, the runner pulls the workload, executes it locally, and streams logs back over the outbound connection. Because communication is initiated from inside the private network outward, no inbound firewall ports (like port 80 or 443) need to be opened on the runner host.

### Q15: Why is it highly dangerous to configure a Self-Hosted Runner on a Public GitHub Repository?
* **Answer**: Anyone can fork a public repository and open a Pull Request. If the PR modifies the workflow YAML files, those modifications will execute automatically on the self-hosted runner. A malicious user can write steps that scan your private corporate network, access private database assets, consume system resources (e.g., for crypto-mining), or steal credentials.

### Q16: How do you implement secure password/credential handling in runner logs?
* **Answer**:
  1. Store all sensitive credentials as **Repository Secrets** in GitHub.
  2. Access them in workflows via `${{ secrets.MY_SECRET }}` (GitHub automatically masks these in log outputs).
  3. For dynamically generated values, use the runner command to explicitly mask the string:
     `echo "::add-mask::MY_DYNAMIC_VALUE"`
     This instructs the runner's logging engine to replace all occurrences of that string with `***` in output logs.

### Q17: Contrast authenticating with GitHub Container Registry (GHCR) using `GITHUB_TOKEN` versus a Personal Access Token (PAT). Which is preferred?
* **Answer**:
  * **`GITHUB_TOKEN` (Preferred)**: Automatically generated for each individual workflow run. It is short-lived, expires when the job finishes, and its scope is strictly limited to the hosting repository. It is highly secure and prevents credential leaks.
  * **Personal Access Token (Classic PAT)**: Tied directly to a user's global GitHub account. It persists until revoked and has broad permissions across all repos. PATs pose a severe risk if exposed in logs or scripts.

### Q18: Explain how to configure Docker layer caching inside GitHub Actions to prevent rebuilding unchanged layers.
* **Answer**:
  We configure layer caching by using the **`docker/setup-buildx-action`** and configuring the `gha` (GitHub Actions) cache backend in the **`docker/build-push-action`**:
  ```yaml
  - name: Build and Push
    uses: docker/build-push-action@v5
    with:
      push: true
      tags: ghcr.io/owner/app:latest
      cache-from: type=gha
      cache-to: type=gha,mode=max
  ```
  This exports compiled layer caches to GitHub Actions' cache service, allowing subsequent builds to skip unchanged layers and build in seconds instead of minutes.

### Q19: What is OpenID Connect (OIDC), and how does it improve cloud deployment security?
* **Answer**: OIDC allows GitHub Actions workflows to authenticate directly with cloud providers (like AWS, Azure, GCP) without storing long-lived credentials (like AWS Access Keys) as static secrets.
  The workflow requests a temporary JSON Web Token (JWT) from GitHub's OIDC provider. It exchanges this short-lived token with the cloud provider for a temporary, role-scoped session token that expires in minutes. This eliminates static credential leakage risks.

### Q20: A self-hosted runner job hangs indefinitely in the "Queued" status. What are the three most likely causes and how would you resolve them?
* **Answer**:
  1. **Label Mismatch**: The workflow's `runs-on` array lists labels (e.g., `runs-on: [self-hosted, ubunto-latest]`) that do not exactly match the labels registered on any active runner. *Fix*: Verify runner labels in repository settings.
  2. **Runner Agent Offline**: The self-hosted machine or runner service is shut down. *Fix*: Check the host machine and run `./run.sh` or check the systemd service.
  3. **Repository Limits/Queue Throttling**: The organization or account has run out of concurrent execution minutes or reached platform limits. *Fix*: Check billing or plan concurrency limits.
