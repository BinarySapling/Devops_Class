# Continuous Integration (CI) with GitHub Actions
### A Complete Beginner-to-Advanced Guide

---

## Table of Contents

1. [What is Continuous Integration?](#1-what-is-continuous-integration)
2. [Key Components](#2-key-components)
3. [Workflow Directory Structure](#3-workflow-directory-structure)
4. [Workflow Triggers (Events)](#4-workflow-triggers-events)
5. [Jobs & Matrix Strategies](#5-jobs--matrix-strategies)
6. [Steps & Shell Commands](#6-steps--shell-commands)
7. [Using Marketplace Actions](#7-using-marketplace-actions)
8. [Language-Specific Actions](#8-language-specific-actions)
9. [Caching for Faster Builds](#9-caching-for-faster-builds)
10. [Multi-Job Workflows](#10-multi-job-workflows)
11. [Runners: GitHub-Hosted](#11-runners-github-hosted)
12. [Runners: Self-Hosted](#12-runners-self-hosted)
13. [Runner Security & Management](#13-runner-security--management)
14. [Docker & GitHub Actions](#14-docker--github-actions)
15. [Deployments to Servers & Clouds](#15-deployments-to-servers--clouds)

---

## 1. What is Continuous Integration?

CI is the practice of **automatically building and testing your code every time someone pushes changes**. GitHub Actions is GitHub's built-in automation engine. Instead of manually running tests, linting, and builds — GitHub does it for you on every push, pull request, or schedule.

### The CI Flow

```
You push code → GitHub detects event → Workflow triggers → Runner executes jobs → Pass / Fail report
```

### Why CI Matters

- Catches bugs early before they reach production
- Ensures code quality is consistent across the team
- Automates repetitive tasks (linting, testing, building, deploying)
- Gives confidence to merge pull requests
- Creates an audit trail of what ran, when, and what the result was

---

## 2. Key Components

Understanding these six building blocks is essential before writing your first workflow.

### Workflow
A YAML file stored in `.github/workflows/` that defines **what to do** and **when to do it**. Every `.yml` file in that folder is a separate, independent workflow.

### Event / Trigger
What **starts** the workflow. Common triggers include: a push to a branch, opening a pull request, a scheduled time (cron), or clicking a button manually in the GitHub UI.

### Job
A **group of steps** that run sequentially on one runner machine. Multiple jobs in a workflow run **in parallel** by default. You can make jobs depend on each other using `needs:`.

### Step
A **single task** inside a job. A step is either:
- A `run:` — a shell command you write directly
- A `uses:` — a pre-built action from the GitHub Marketplace

### Action
A **reusable, pre-built step** published on the GitHub Marketplace. For example, `actions/checkout@v4` downloads your repository code onto the runner. You don't write the logic — you just `uses:` it.

### Runner
The **virtual machine** that executes your jobs. GitHub provides Ubuntu, Windows, and macOS runners. You can also connect your own machine as a self-hosted runner.

---

### Mental Model

> A workflow file = a **recipe**.
> Events = what **starts cooking**.
> Jobs = **cooking stations**.
> Steps = **individual instructions**.
> Runners = **the kitchen**.
> Actions = **pre-made ingredients**.

---

### Anatomy of a Complete Workflow File

```yaml
# The name shown in the Actions tab on GitHub
name: CI Pipeline

# What triggers this workflow
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

# All the jobs to run
jobs:
  build-and-test:           # job ID (your custom name, no spaces)
    runs-on: ubuntu-latest  # which runner to use

    steps:
      # Step 1: download your repository code onto the runner
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: install and configure Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      # Step 3: a raw shell command
      - name: Install dependencies
        run: npm install

      # Step 4: another shell command
      - name: Run tests
        run: npm test
```

**Key YAML rules to remember:**
- Indentation matters — use 2 spaces, never tabs
- Keys like `name:`, `on:`, `jobs:` are at the root level
- `steps:` is a list — each item starts with `-`
- Strings with special characters should be quoted

---

## 3. Workflow Directory Structure

GitHub Actions workflows live in a **fixed, specific folder** inside your repository. GitHub only reads from `.github/workflows/`.

```
your-project/
├── .github/
│   ├── workflows/              ← GitHub reads ALL .yml files here
│   │   ├── ci.yml              ← runs on every push (tests, lint)
│   │   ├── deploy.yml          ← runs on release tag
│   │   ├── nightly.yml         ← runs on a schedule
│   │   └── docker.yml          ← builds and pushes Docker images
│   ├── CODEOWNERS              ← optional: who reviews which files
│   └── dependabot.yml          ← optional: auto-update dependencies
├── src/
│   └── index.js
├── tests/
│   └── index.test.js
├── package.json
├── package-lock.json
└── Dockerfile
```

### Key Rules

- Every `.yml` or `.yaml` file inside `.github/workflows/` is treated as a **separate workflow**
- You can have as many workflows as you want — they run independently and in parallel
- Workflow files can reference any file in your repo using paths relative to the repo root
- The `.github/` folder must be at the **root** of your repository

### Referencing Files Inside Workflows

```yaml
jobs:
  build:
    steps:
      # Paths are relative to your repository root
      - name: Run deploy script
        run: bash ./scripts/deploy.sh

      # Set working directory for a specific step
      - name: Build frontend
        working-directory: ./frontend
        run: npm run build

      # Reference files using environment variables
      - name: Use config file
        run: cat ./config/production.json
```

### Best Practice: One Workflow Per Concern

| File | Purpose |
|------|---------|
| `ci.yml` | Lint, test, type-check on every push |
| `deploy.yml` | Deploy to staging/production on release |
| `docker.yml` | Build and push Docker images |
| `nightly.yml` | Scheduled health checks, security scans |
| `pr-checks.yml` | PR-specific validations |

---

## 4. Workflow Triggers (Events)

The `on:` key tells GitHub **what events should start your workflow**. You can combine multiple triggers in one workflow.

### Push Trigger

Runs when code is pushed to a branch or tag.

```yaml
on:
  push:
    branches:
      - main
      - develop
      - 'feature/**'        # wildcard: any branch starting with feature/
      - 'release/v[0-9]+.*' # regex-style: release/v1.0, release/v2.3
    paths:                   # only trigger if these specific files changed
      - 'src/**'
      - 'package.json'
      - 'Dockerfile'
    paths-ignore:            # alternatively, ignore certain paths
      - '**.md'              # ignore markdown files (docs changes)
      - 'docs/**'
    tags:                    # also trigger on version tags
      - 'v*'                 # matches v1.0.0, v2.3.1, etc.
```

### Pull Request Trigger

Runs when pull request activity happens.

```yaml
on:
  pull_request:
    branches:
      - main                 # only PRs targeting main
    types:                   # which PR events to react to
      - opened               # when PR is first created
      - synchronize          # when new commits are pushed to PR
      - reopened             # when a closed PR is reopened
      - ready_for_review     # when draft PR is marked ready
    paths:
      - 'src/**'             # only run if source files changed
```

### Schedule Trigger (Cron)

Runs on a time schedule, defined using standard cron syntax.

```yaml
on:
  schedule:
    # Runs at 2:30 AM every day (UTC timezone)
    - cron: '30 2 * * *'

    # Cron format breakdown:
    # ┌─── minute (0-59)
    # │  ┌─── hour (0-23)
    # │  │  ┌─── day of month (1-31)
    # │  │  │  ┌─── month (1-12)
    # │  │  │  │  ┌─── day of week (0-6, 0=Sunday)
    # *  *  *  *  *

    # Every Monday at 9:00 AM UTC
    - cron: '0 9 * * 1'

    # Every 15 minutes
    - cron: '*/15 * * * *'

    # First day of every month at midnight
    - cron: '0 0 1 * *'
```

> **Note:** Schedules on inactive repositories (no activity for 60 days) are automatically disabled by GitHub.

### Manual Workflow (workflow_dispatch)

Adds a "Run workflow" button in the GitHub Actions UI, optionally accepting user-provided inputs.

```yaml
on:
  workflow_dispatch:          # adds "Run workflow" button in GitHub UI
    inputs:
      environment:
        description: 'Deploy to which environment?'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
          - preview
      run_tests:
        description: 'Run tests before deploying?'
        type: boolean
        default: true
      version_tag:
        description: 'Version tag to deploy (e.g. v1.2.3)'
        required: false
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Show inputs
        run: |
          echo "Deploying to: ${{ github.event.inputs.environment }}"
          echo "Run tests: ${{ github.event.inputs.run_tests }}"
          echo "Version: ${{ github.event.inputs.version_tag }}"

      - name: Run tests conditionally
        if: github.event.inputs.run_tests == 'true'
        run: npm test
```

### Other Useful Triggers

```yaml
on:
  # When a GitHub Release is published
  release:
    types: [ published ]

  # When an issue or PR comment is created
  issue_comment:
    types: [ created ]

  # Called by another workflow (reusable workflow)
  workflow_call:
    inputs:
      environment:
        required: true
        type: string

  # Triggered via the GitHub REST API (for external systems)
  repository_dispatch:
    types: [ deploy-command ]

  # When a branch or tag is created
  create: {}

  # When code is deleted (branch/tag deleted)
  delete: {}
```

### Combining Multiple Triggers

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch: {}
```

---

## 5. Jobs & Matrix Strategies

A **job** is a set of steps that run sequentially on the same runner. Multiple jobs run **in parallel** by default unless you use `needs:` to create dependencies.

### Basic Job Configuration

```yaml
jobs:
  test:
    name: Run Tests                    # display name shown in GitHub UI
    runs-on: ubuntu-latest
    timeout-minutes: 30                # cancel job if it exceeds 30 minutes
    continue-on-error: false           # fail the entire workflow if this job fails

    env:                               # environment variables for the entire job
      NODE_ENV: test
      DATABASE_URL: postgresql://localhost/testdb
      LOG_LEVEL: debug

    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

### Job Dependencies with `needs:`

```yaml
jobs:
  # Job 1 — runs first (no dependencies)
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  # Job 2 — also runs first, in parallel with lint
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  # Job 3 — waits for BOTH lint and test to pass
  build:
    runs-on: ubuntu-latest
    needs: [ lint, test ]
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

  # Job 4 — waits for build
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "All checks passed, deploying!"
```

### Matrix Strategy — Test Across Multiple Versions

Instead of writing 9 separate jobs for Node 18/20/22 on Ubuntu/Windows/macOS, define a matrix and GitHub creates all combinations automatically.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}           # dynamically set from matrix

    strategy:
      fail-fast: false                  # if one combo fails, keep running others
      matrix:
        os: [ ubuntu-latest, windows-latest, macos-latest ]
        node: [ 18, 20, 22 ]
        # ↑ creates 3×3 = 9 parallel jobs total!

        # Exclude specific combinations
        exclude:
          - os: windows-latest
            node: 18                    # skip Windows + Node 18

        # Add extra variables to specific combinations
        include:
          - os: ubuntu-latest
            node: 20
            coverage: true             # only run coverage on this combo

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}   # inject matrix value
          cache: 'npm'

      - name: Install and test
        run: npm ci && npm test

      # Conditional step — only runs for the combo with coverage: true
      - name: Generate coverage report
        if: matrix.coverage == true
        run: npm run coverage

      - name: Upload coverage
        if: matrix.coverage == true
        uses: codecov/codecov-action@v4
```

### Passing Data Between Jobs Using Outputs

Jobs run on separate machines, so you can't share files directly. Use `outputs:` to pass small values (strings, version numbers, flags).

```yaml
jobs:
  get-version:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get_ver.outputs.version }}
      sha_short: ${{ steps.get_sha.outputs.sha_short }}
    steps:
      - uses: actions/checkout@v4

      - name: Get version from package.json
        id: get_ver
        run: echo "version=$(cat package.json | jq -r .version)" >> $GITHUB_OUTPUT

      - name: Get short SHA
        id: get_sha
        run: echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

  build:
    needs: get-version
    runs-on: ubuntu-latest
    steps:
      - name: Use outputs from previous job
        run: |
          echo "Building version: ${{ needs.get-version.outputs.version }}"
          echo "Commit SHA: ${{ needs.get-version.outputs.sha_short }}"
```

---

## 6. Steps & Shell Commands

Steps are the **individual units of work** inside a job. A step is either a `run:` (shell command) or `uses:` (a marketplace action).

### Shell Commands with `run:`

```yaml
steps:
  # Single line command
  - name: Print current date
    run: date

  # Multi-line commands using the pipe symbol (|)
  - name: Setup and build
    run: |
      echo "Starting build process..."
      npm ci
      npm run lint
      npm run build
      echo "Build complete!"

  # Environment variable scoped to a single step
  - name: Deploy with API key
    env:
      API_KEY: ${{ secrets.DEPLOY_API_KEY }}
      DEPLOY_ENV: production
    run: ./scripts/deploy.sh

  # Conditional step — only runs on the main branch
  - name: Only run on main
    if: github.ref == 'refs/heads/main'
    run: echo "This is the main branch!"

  # Use a different shell (default is bash on Linux/macOS, powershell on Windows)
  - name: PowerShell step
    shell: pwsh
    run: Write-Output "Hello from PowerShell"

  # Python shell
  - name: Python script
    shell: python
    run: |
      import os
      print("Hello from Python:", os.getcwd())

  # Continue workflow even if this step fails
  - name: Optional quality check
    continue-on-error: true
    run: npm run optional-check

  # Capture command output as a step output
  - name: Get git log
    id: git_log
    run: echo "commit_msg=$(git log -1 --pretty=%B)" >> $GITHUB_OUTPUT

  - name: Use the output
    run: echo "Last commit was: ${{ steps.git_log.outputs.commit_msg }}"
```

### Common Condition Expressions for `if:`

```yaml
# Run only on the main branch
if: github.ref == 'refs/heads/main'

# Run only on push events (not pull requests)
if: github.event_name == 'push'

# Run only on pull request events
if: github.event_name == 'pull_request'

# Run only if ALL previous steps succeeded
if: success()

# Run always, even if previous steps failed (e.g. cleanup)
if: always()

# Run only if a previous step/job failed (e.g. send failure notification)
if: failure()

# Run only if the job was cancelled
if: cancelled()

# Check branch name
if: startsWith(github.ref, 'refs/tags/v')

# Check commit message contains a keyword
if: contains(github.event.head_commit.message, '[deploy]')

# Check if a matrix variable is set
if: matrix.coverage == true

# Combine conditions with AND / OR
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
```

### Setting Environment Variables

```yaml
# Workflow-level (available to all jobs)
env:
  NODE_ENV: production
  APP_NAME: my-app

jobs:
  build:
    # Job-level (available to all steps in this job)
    env:
      BUILD_DIR: ./dist

    steps:
      # Step-level (only available in this step)
      - name: Deploy
        env:
          SECRET_KEY: ${{ secrets.MY_SECRET }}
        run: ./deploy.sh

      # Dynamic env var using GITHUB_ENV
      - name: Set dynamic variable
        run: echo "TIMESTAMP=$(date +%s)" >> $GITHUB_ENV

      - name: Use dynamic variable
        run: echo "Build started at $TIMESTAMP"
```

---

## 7. Using Marketplace Actions

The GitHub Marketplace has thousands of pre-built actions. Using `uses:` lets you leverage community-built functionality without writing the code yourself.

### Action Versioning

```yaml
steps:
  # Pin to a major version (auto-gets latest patch/minor)
  - uses: actions/checkout@v4

  # Pin to exact version (most secure — no surprise updates)
  - uses: actions/setup-node@v4.2.0

  # Pin to a specific commit SHA (most secure for third-party actions)
  - uses: some-org/some-action@a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4e5f6a1b2

  # Use from a branch (NOT recommended for production)
  - uses: actions/checkout@main
```

> **Security tip:** For third-party actions (not `actions/` namespace), always pin to a full commit SHA. This prevents supply-chain attacks where a maintainer tags a new version with malicious code.

### Using Actions with Inputs (`with:`)

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0        # fetch full git history (default: 1 = shallow)
      ref: main             # checkout a specific branch
      token: ${{ secrets.GITHUB_TOKEN }}

  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      node-version-file: '.nvmrc'   # alternatively, read from file
      cache: 'npm'                   # built-in caching
      registry-url: 'https://registry.npmjs.org'  # for npm publish

  - uses: actions/upload-artifact@v4
    with:
      name: test-results
      path: ./coverage/
      retention-days: 30            # keep for 30 days
      if-no-files-found: error      # fail if nothing to upload
```

### Referencing Step Outputs

```yaml
steps:
  # Give the step an id so you can reference its outputs
  - id: my_step
    uses: some-action/that-has-outputs@v1

  # Reference outputs from the step above
  - run: echo "The result was: ${{ steps.my_step.outputs.result }}"

  # Reference outputs in an if condition
  - name: Conditional on output
    if: steps.my_step.outputs.changed == 'true'
    run: echo "Something changed!"
```

---

## 8. Language-Specific Actions

### Node.js / JavaScript

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'        # LTS version
    cache: 'npm'              # or 'yarn' or 'pnpm'
    registry-url: 'https://registry.npmjs.org'   # needed for npm publish

# Install dependencies the right way for CI
- run: npm ci                 # clean install (respects lock file, faster)

# Common Node.js CI commands
- run: npm run lint
- run: npm test
- run: npm run build

# Publish to npm
- name: Publish to NPM
  run: npm publish --access public
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Python

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'              # caches ~/.cache/pip

- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt

- name: Run tests with pytest
  run: pytest tests/ -v --cov=src --cov-report=xml

- name: Lint with flake8
  run: flake8 src/ --max-line-length=88

# Publish to PyPI
- name: Build and publish
  uses: pypa/gh-action-pypi-publish@release/v1
  with:
    user: __token__
    password: ${{ secrets.PYPI_API_TOKEN }}
```

### Java (Maven / Gradle)

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'   # or 'zulu', 'corretto', 'microsoft'
    cache: 'maven'            # or 'gradle'

# Maven
- name: Build and test with Maven
  run: mvn -B package --file pom.xml

# Gradle
- name: Build and test with Gradle
  run: ./gradlew build test
```

### Go

```yaml
- name: Setup Go
  uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true               # caches Go module downloads

- name: Download dependencies
  run: go mod download

- name: Build
  run: go build -v ./...

- name: Test
  run: go test -v ./...

- name: Lint
  uses: golangci/golangci-lint-action@v6
  with:
    version: latest
```

### Rust

```yaml
- name: Setup Rust
  uses: dtolnay/rust-toolchain@stable
  with:
    toolchain: stable
    components: clippy, rustfmt

- name: Cache Rust dependencies
  uses: Swatinem/rust-cache@v2

- name: Build
  run: cargo build --verbose

- name: Test
  run: cargo test --verbose

- name: Lint with Clippy
  run: cargo clippy -- -D warnings

- name: Check formatting
  run: cargo fmt --all -- --check
```

### PHP

```yaml
- name: Setup PHP
  uses: shivammathur/setup-php@v2
  with:
    php-version: '8.3'
    extensions: mbstring, xml, ctype, pdo, sqlite3
    coverage: xdebug

- name: Install dependencies
  run: composer install --prefer-dist --no-progress

- name: Run tests
  run: ./vendor/bin/phpunit
```

---

## 9. Caching for Faster Builds

Every time a runner starts, it has **zero dependencies installed**. Without caching, `npm install` downloads thousands of packages on every run — taking 2-5 minutes. Caching saves packages between runs so installs take seconds.

### How Caching Works

1. You define a **cache key** — a string that uniquely identifies your cache (usually a hash of your lock file)
2. On first run: no cache exists, dependencies are downloaded and saved with the key
3. On subsequent runs: GitHub checks if the key matches an existing cache
4. **Cache hit:** the cached folder is restored in seconds (skips download)
5. **Cache miss:** dependencies are re-downloaded and the new cache is saved

GitHub caches up to **10 GB** per repository. Caches older than 7 days are automatically evicted.

### Built-in Caching via Setup Actions (Easiest Method)

```yaml
# setup-node has built-in cache support — just add cache: 'npm'
- name: Setup Node.js with cache
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'              # automatically caches ~/.npm

# Same for yarn
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'yarn'

# Python pip caching
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'

# Go module caching
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true
```

### Manual Caching with `actions/cache` (Full Control)

```yaml
steps:
  - uses: actions/checkout@v4

  # Cache node_modules manually
  - name: Cache node_modules
    id: cache-node-modules
    uses: actions/cache@v4
    with:
      path: ~/.npm                          # what directory to cache
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      # ↑ key changes ONLY when package-lock.json changes
      restore-keys: |
        ${{ runner.os }}-node-             # fallback: use any previous node cache

  - name: Install dependencies
    run: npm ci                             # fast if cache hit

  # Cache multiple directories at once
  - name: Cache Next.js build
    uses: actions/cache@v4
    with:
      path: |
        ~/.npm
        ${{ github.workspace }}/.next/cache
      key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.ts', '**/*.tsx') }}
      restore-keys: |
        ${{ runner.os }}-nextjs-
```

### Cache Key Strategies

```yaml
# Best practice: include OS + tool version + lock file hash
key: ${{ runner.os }}-node-20-${{ hashFiles('**/package-lock.json') }}

# For Yarn (uses yarn.lock)
key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}

# For Python pip
key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}

# For Maven
key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}

# For Gradle
key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

# For Cargo (Rust)
key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
```

### What to Cache Per Language

| Language | Cache Path | Lock File |
|----------|-----------|-----------|
| Node.js (npm) | `~/.npm` | `package-lock.json` |
| Node.js (yarn) | `~/.yarn/cache` | `yarn.lock` |
| Python (pip) | `~/.cache/pip` | `requirements.txt` |
| Java (Maven) | `~/.m2` | `pom.xml` |
| Java (Gradle) | `~/.gradle` | `*.gradle` |
| Go | `~/go/pkg/mod` | `go.sum` |
| Rust (Cargo) | `~/.cargo` | `Cargo.lock` |
| Ruby (Bundler) | `vendor/bundle` | `Gemfile.lock` |

### Time Savings Example

| Scenario | Without Cache | With Cache | Savings |
|----------|--------------|------------|---------|
| `npm install` (500 packages) | 3-4 minutes | 10-30 seconds | ~85% |
| `pip install` (50 packages) | 1-2 minutes | 5-15 seconds | ~90% |
| `mvn install` (Java) | 5-8 minutes | 20-45 seconds | ~90% |

---

## 10. Multi-Job Workflows

Real-world CI pipelines have many jobs working together. Testing, linting, building, security scanning, and deploying are often separate jobs. **Artifacts** let jobs share files between them.

### Complete Multi-Job Pipeline

```yaml
name: Full CI/CD Pipeline
on:
  push:
    branches: [ main ]

jobs:
  # ─── Phase 1: Parallel checks ───────────────────────────────
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
      - name: Upload coverage report
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run npm audit
        run: npm audit --audit-level=moderate
      - name: Run CodeQL analysis
        uses: github/codeql-action/analyze@v3

  # ─── Phase 2: Build (waits for all Phase 1 jobs) ─────────────
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [ lint, test, security ]   # all three must pass first
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run build

      # Save build output so deploy job can use it
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: production-build
          path: ./dist
          retention-days: 7
          if-no-files-found: error

  # ─── Phase 3: Deploy (waits for build) ───────────────────────
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    environment: staging              # uses GitHub Environment protection
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: production-build
          path: ./dist

      - name: Deploy to staging
        run: ./scripts/deploy.sh staging
        env:
          DEPLOY_TOKEN: ${{ secrets.STAGING_DEPLOY_TOKEN }}

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production           # can require manual approval in GitHub UI
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: production-build
          path: ./dist

      - name: Deploy to production
        run: ./scripts/deploy.sh production
        env:
          DEPLOY_TOKEN: ${{ secrets.PROD_DEPLOY_TOKEN }}
```

### Environment Protection Rules

GitHub Environments let you require **manual approval** before a job runs — essential for production deployments.

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment: production        # links to a named GitHub Environment
    # In GitHub: Settings → Environments → production → add Required reviewers
    # The job pauses and waits for a designated reviewer to approve
    steps:
      - run: echo "Deploying to production!"
        env:
          # Secrets scoped to this environment only
          DB_PASSWORD: ${{ secrets.PROD_DB_PASSWORD }}
```

### Failure Notifications

```yaml
jobs:
  notify-on-failure:
    runs-on: ubuntu-latest
    needs: [ lint, test, build ]
    if: failure()                  # only runs if any previous job failed
    steps:
      - name: Notify Slack
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {"text": "CI failed on ${{ github.ref }}! Check: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## 11. Runners: GitHub-Hosted

A runner is the **machine that actually runs your workflow jobs**. GitHub-hosted runners are virtual machines provided and managed entirely by GitHub.

### Key Characteristics

- **Zero setup** — no configuration needed
- **Fresh environment** every time — each job starts with a clean VM
- **Pre-installed software** — Node.js, Python, Java, Docker, Git, and hundreds of other tools are already available
- **Automatically updated** — GitHub keeps them patched

### Available Runner Labels

```yaml
# Ubuntu (Linux) — most popular, cheapest, fastest
runs-on: ubuntu-latest        # Ubuntu 24.04 (current latest)
runs-on: ubuntu-22.04         # Ubuntu 22.04 (LTS)
runs-on: ubuntu-20.04         # Ubuntu 20.04 (older LTS)

# Windows
runs-on: windows-latest       # Windows Server 2022
runs-on: windows-2022         # Windows Server 2022
runs-on: windows-2019         # Windows Server 2019

# macOS — most expensive (10x billing multiplier)
runs-on: macos-latest         # macOS 14 (Sonoma) on Apple Silicon
runs-on: macos-14             # macOS 14 (Apple M1 chip)
runs-on: macos-13             # macOS 13 (Intel chip)
runs-on: macos-12             # macOS 12 (Intel chip)

# Larger runners (paid plans only)
runs-on: ubuntu-latest-4-cores    # 4 vCPUs, 16 GB RAM
runs-on: ubuntu-latest-8-cores    # 8 vCPUs, 32 GB RAM
runs-on: ubuntu-latest-16-cores   # 16 vCPUs, 64 GB RAM
```

### Standard Runner Specs (Free Tier)

| OS | CPU | RAM | Storage | Billing Multiplier |
|----|-----|-----|---------|-------------------|
| Ubuntu | 2 vCPU | 7 GB | 14 GB SSD | 1× (baseline) |
| Windows | 2 vCPU | 7 GB | 14 GB SSD | 2× |
| macOS | 3 CPU (M1) | 7 GB | 14 GB SSD | 10× |

### GitHub Actions Free Tier Limits

| Plan | Minutes/Month |
|------|--------------|
| Free (personal) | 2,000 minutes |
| Pro | 3,000 minutes |
| Team | 3,000 minutes |
| Enterprise | 50,000 minutes |

> Minutes are calculated based on the billing multiplier. Using macOS for 100 minutes consumes 1,000 of your free minutes.

### Pre-installed Software

GitHub-hosted Ubuntu runners include (among many others):
- Node.js (multiple versions via nvm)
- Python 3.x
- Java (JDK 8, 11, 17, 21)
- Go
- Ruby
- Docker + Docker Compose
- Git, curl, wget, jq, unzip
- AWS CLI, Azure CLI, Google Cloud SDK

Full list: https://github.com/actions/runner-images

---

## 12. Runners: Self-Hosted

Self-hosted runners are **your own machines** (a server, cloud VM, Raspberry Pi, or even a local laptop) connected to GitHub to run workflows.

### When to Use Self-Hosted Runners

- Need GPU access (ML model training, GPU compute)
- Need access to private network resources (internal databases, VPNs)
- Need specific hardware or OS not available on GitHub runners
- Want unlimited minutes without per-minute billing
- Have very long-running jobs (>6 hours)
- Need large amounts of RAM or storage

### Setup: Installing the Runner on Your Machine

```bash
# Step 1: In GitHub — go to:
# Repo → Settings → Actions → Runners → New self-hosted runner
# Choose your OS, then follow the download and configure steps shown

# Step 2: Create directory and download runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.317.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.317.0/actions-runner-linux-x64-2.317.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.317.0.tar.gz

# Step 3: Configure — paste the token shown in GitHub UI
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO \
            --token YOUR_REGISTRATION_TOKEN \
            --name "my-production-server" \
            --labels "self-hosted,linux,x64,production"

# Step 4: Start the runner (foreground mode for testing)
./run.sh

# Step 5: Install as a persistent background service
sudo ./svc.sh install     # creates systemd service
sudo ./svc.sh start       # starts the service
sudo ./svc.sh status      # check if running
sudo ./svc.sh stop        # stop when needed
```

### Using Self-Hosted Runners in Workflows

```yaml
jobs:
  build:
    # Use any self-hosted runner
    runs-on: self-hosted

  build-linux:
    # Filter by OS and architecture labels
    runs-on: [ self-hosted, linux, x64 ]

  build-gpu:
    # Use a runner with a custom label you assigned during setup
    runs-on: [ self-hosted, gpu, production ]

  build-windows:
    # Self-hosted Windows runner
    runs-on: [ self-hosted, windows ]
```

### Removing a Self-Hosted Runner

```bash
# Stop the service first
sudo ./svc.sh stop
sudo ./svc.sh uninstall

# Remove the runner registration from GitHub
./config.sh remove --token YOUR_REMOVAL_TOKEN
```

---

## 13. Runner Security & Management

### Critical Security Warning

> **Never use self-hosted runners on public repositories.**
>
> Anyone can fork a public repo and submit a pull request. If a workflow runs on PRs, the attacker's code will execute on your machine. This can compromise your server, steal secrets, and access your private network.

Self-hosted runners should only be used with **private repositories** or with workflows that only trigger on `push` events from trusted branches (never on `pull_request` from forks).

### Security Best Practices

```yaml
# Restrict which events can trigger workflows on self-hosted runners
on:
  push:
    branches: [ main ]       # only trusted pushes
  # Never: pull_request on public repos with self-hosted runners
```

**Operational security checklist:**
- Create a **dedicated low-privilege user** for the runner (not root)
- Run the runner **inside a Docker container** or VM for isolation
- **Never store credentials or secrets** as files on the runner disk
- Regularly **update** the runner software (`./config.sh --version`)
- Set up **automatic runner cleanup** between jobs for ephemeral isolation
- Use **network policies** to restrict what the runner can reach
- Review runner **audit logs** in Settings → Actions → Runners

### Ephemeral Runners (Best Practice for Security)

```bash
# Configure runner to shut down after completing one job
./config.sh --ephemeral \
            --url https://github.com/ORG/REPO \
            --token TOKEN
# A fresh runner starts for every job — no state leaks between runs
```

### Runner Groups (Organization Level)

Organization admins can create runner groups to:
- Share runners across multiple repositories
- Restrict which repositories can access which runners
- Assign runners with special hardware to specific teams

```yaml
jobs:
  build:
    # Target a specific runner group by label
    runs-on: [ self-hosted, production-runners ]
```

### Concurrency Control

Prevent multiple deployments from running at the same time:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true    # cancel older runs when new one starts
```

---

## 14. Docker & GitHub Actions

GitHub Actions is excellent for Docker workflows — building images, tagging them with semantic versions or commit SHAs, and pushing to any registry.

### Core Actions for Docker

| Action | Purpose |
|--------|---------|
| `docker/setup-buildx-action` | Enable BuildKit for multi-platform builds |
| `docker/metadata-action` | Auto-generate tags and labels |
| `docker/login-action` | Authenticate to any registry |
| `docker/build-push-action` | Build and push images |

### Building a Docker Image (Without Pushing)

```yaml
name: Build Docker Image
on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Enable Docker BuildKit (required for --cache-from, multi-platform)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Auto-generate image tags based on branch, version, SHA
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: myorg/myapp
          tags: |
            type=ref,event=branch          # branch name: main, develop
            type=semver,pattern={{version}} # v1.2.3
            type=semver,pattern={{major}}.{{minor}} # v1.2
            type=sha,prefix=sha-           # sha-abc1234

      # Build image (push: false = build only, don't push)
      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .                       # Dockerfile location
          file: ./Dockerfile               # explicit Dockerfile path
          push: false
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          build-args: |
            NODE_VERSION=20
            APP_ENV=production
          cache-from: type=gha             # use GitHub Actions cache for layers
          cache-to: type=gha,mode=max      # save layers to cache
```

### Pushing to Docker Hub

**Prerequisites:**
1. Create an account at hub.docker.com
2. Go to Account Settings → Security → New Access Token
3. In GitHub repo: Settings → Secrets → Add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`

```yaml
name: Push to Docker Hub
on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]            # also trigger on version tags

jobs:
  push-dockerhub:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU (for multi-platform builds)
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Authenticate to Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Auto-generate tags
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/myapp

      # Build for multiple CPU architectures and push
      - name: Build and push multi-platform image
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64    # supports both Intel and Apple M1
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Pushing to GitHub Container Registry (GHCR)

**Advantages over Docker Hub:**
- No separate account — uses your GitHub token
- Free for public images
- Integrated with GitHub permissions (repo access = image access)
- Images stored at `ghcr.io/username/image-name`

**No secret setup needed** — uses the automatically available `secrets.GITHUB_TOKEN`.

```yaml
name: Push to GHCR
on:
  push:
    branches: [ main ]
    tags: [ 'v*.*.*' ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}     # e.g. myorg/myapp

jobs:
  push-ghcr:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write                      # required permission to push to GHCR

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Login using the built-in GitHub token — no secrets needed!
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha,prefix=sha-
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push to GHCR
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          # Open Container Initiative labels for discoverability
          annotations: |
            org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
            org.opencontainers.image.description=My application
            org.opencontainers.image.licenses=MIT
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Docker Multi-Stage Build Example

Use multi-stage Dockerfiles to keep final images small:

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production image (only the built output)
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

```yaml
# Build targeting a specific stage
- name: Build production image
  uses: docker/build-push-action@v5
  with:
    context: .
    target: production         # only build up to the 'production' stage
    push: true
    tags: myapp:latest
```

---

## 15. Deployments to Servers & Clouds

### GitHub Secrets — Storing Sensitive Values

Never hardcode API keys, passwords, or tokens in your YAML. Use GitHub Secrets.

**To add a secret:** Repo → Settings → Secrets and variables → Actions → New repository secret

```yaml
# Access secrets in workflows
steps:
  - run: ./deploy.sh
    env:
      API_KEY: ${{ secrets.MY_API_KEY }}
      DB_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}
```

### Built-in Variables (No Setup Required)

```yaml
${{ secrets.GITHUB_TOKEN }}      # auto-generated token for the workflow run
${{ github.sha }}                 # full commit SHA (e.g. a1b2c3d4...)
${{ github.ref }}                 # full ref (e.g. refs/heads/main)
${{ github.ref_name }}            # short name (e.g. main, v1.0.0)
${{ github.actor }}               # who triggered the workflow
${{ github.repository }}          # owner/repo-name
${{ github.event_name }}          # push, pull_request, schedule, etc.
${{ github.run_id }}              # unique ID for this workflow run
${{ github.run_number }}          # incrementing number per workflow
${{ github.server_url }}          # https://github.com
${{ runner.os }}                  # Linux, Windows, macOS
```

### Deploying to a Linux VPS via SSH

**Setup steps:**
1. Generate an SSH key pair: `ssh-keygen -t ed25519 -C "github-actions"`
2. Add the **private key** content as secret `SSH_PRIVATE_KEY` in GitHub
3. Append the **public key** to your server's `~/.ssh/authorized_keys`
4. Add server host/IP as `SERVER_HOST` secret, username as `SERVER_USER`

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: npm ci && npm run build

      # Method 1: Using appleboy/ssh-action (simplest)
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci --production
            npm run build
            pm2 restart myapp
            echo "Deployment complete!"

      # Method 2: Manual SSH + rsync (more control)
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      - name: Sync files to server
        run: |
          rsync -avz --delete \
            --exclude='.git' \
            --exclude='node_modules' \
            ./dist/ ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }}:/var/www/myapp/

      - name: Restart application on server
        run: |
          ssh ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} << 'EOF'
            cd /var/www/myapp
            npm ci --production
            pm2 restart myapp || pm2 start dist/index.js --name myapp
          EOF
```

### Deploying to AWS

**Setup:** Create an IAM user with appropriate permissions, store `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as secrets.

#### AWS S3 (Static Sites / Frontend)

```yaml
jobs:
  deploy-s3:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm run build

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1                    # or your region

      - name: Deploy to S3
        run: |
          aws s3 sync ./dist s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --cache-control "max-age=31536000" \
            --exclude "*.html"
          # HTML files should not be cached
          aws s3 sync ./dist s3://${{ secrets.S3_BUCKET_NAME }} \
            --exclude "*" \
            --include "*.html" \
            --cache-control "no-cache"

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

#### AWS ECR + ECS (Containerized Applications)

```yaml
jobs:
  deploy-ecs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1

      # Login to Amazon ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # Build and push image to ECR
      - name: Build, tag, and push image to ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: myapp
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      # Update ECS task definition with new image
      - name: Download existing task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition myapp-task \
            --query taskDefinition > task-definition.json

      - name: Update task definition with new image
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: myapp-container
          image: ${{ steps.build-image.outputs.image }}

      # Deploy updated task definition to ECS
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: myapp-service
          cluster: myapp-cluster
          wait-for-service-stability: true         # wait until deployment completes
```

#### AWS Elastic Beanstalk

```yaml
jobs:
  deploy-eb:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1

      - name: Deploy to Elastic Beanstalk
        uses: einaregilsson/beanstalk-deploy@v22
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: myapp
          environment_name: myapp-production
          version_label: ${{ github.sha }}
          region: ap-south-1
          deployment_package: deploy.zip
```

### Deploying to Google Cloud Platform (GCP)

```yaml
jobs:
  deploy-gcp:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Authenticate to GCP using service account JSON
      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      # Deploy to Cloud Run (serverless containers)
      - name: Deploy to Cloud Run
        uses: google-github-actions/deploy-cloudrun@v2
        with:
          service: myapp
          region: asia-south1
          image: gcr.io/${{ secrets.GCP_PROJECT_ID }}/myapp:${{ github.sha }}
          env_vars: |
            NODE_ENV=production
            PORT=8080

      # Deploy to Google Kubernetes Engine (GKE)
      - name: Get GKE credentials
        uses: google-github-actions/get-gke-credentials@v2
        with:
          cluster_name: my-cluster
          location: asia-south1-a

      - name: Deploy to GKE
        run: |
          kubectl set image deployment/myapp myapp=gcr.io/${{ secrets.GCP_PROJECT_ID }}/myapp:${{ github.sha }}
          kubectl rollout status deployment/myapp
```

### Deploying to Azure

```yaml
jobs:
  deploy-azure:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Deploy to Azure App Service
      - name: Deploy to Azure App Service
        uses: azure/webapps-deploy@v3
        with:
          app-name: my-azure-app
          slot-name: production
          package: ./dist

      # Deploy to Azure Container Apps
      - name: Deploy to Azure Container Apps
        uses: azure/container-apps-deploy-action@v1
        with:
          appSourcePath: ${{ github.workspace }}
          acrName: myregistry
          containerAppName: myapp
          resourceGroup: my-resource-group
```

### Deploying to Vercel

```yaml
jobs:
  deploy-vercel:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'              # deploy to production
          working-directory: ./             # project root
```

### Deploying to Netlify

```yaml
jobs:
  deploy-netlify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm run build

      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: './dist'
          production-branch: main          # auto-detect production vs preview
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions - ${{ github.sha }}"
          enable-pull-request-comment: true
          enable-commit-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Deploying to Railway

```yaml
jobs:
  deploy-railway:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Railway
        uses: bervProject/railway-deploy@main
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: my-service
```

---

## Complete Real-World Example: Full CI/CD with Docker + GHCR + VPS Deploy

This combines everything into a production-grade pipeline:

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [ main ]
    tags: [ 'v*.*.*' ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─── Phase 1: Quality Checks ────────────────────────────────
  lint-and-test:
    name: Lint & Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

      - name: Test with coverage
        run: npm test -- --coverage

      - name: Upload coverage
        if: always()
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # ─── Phase 2: Docker Build & Push ───────────────────────────
  docker-build:
    name: Build & Push Docker Image
    runs-on: ubuntu-latest
    needs: lint-and-test
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha,prefix=sha-

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─── Phase 3: Deploy to Staging ─────────────────────────────
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: docker-build
    if: github.ref == 'refs/heads/main'
    environment: staging

    steps:
      - name: Deploy to staging server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker pull ghcr.io/${{ github.repository }}:main
            docker stop myapp-staging || true
            docker rm myapp-staging || true
            docker run -d \
              --name myapp-staging \
              --restart unless-stopped \
              -p 3001:3000 \
              -e NODE_ENV=staging \
              -e DATABASE_URL=${{ secrets.STAGING_DB_URL }} \
              ghcr.io/${{ github.repository }}:main
            echo "Staging deployment complete!"

  # ─── Phase 4: Deploy to Production (manual approval) ────────
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production                   # requires manual approval in GitHub

    steps:
      - name: Deploy to production server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            IMAGE_TAG=${{ needs.docker-build.outputs.image-tag }}
            echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker pull ghcr.io/${{ github.repository }}:${IMAGE_TAG}
            docker stop myapp-prod || true
            docker rm myapp-prod || true
            docker run -d \
              --name myapp-prod \
              --restart unless-stopped \
              -p 80:3000 \
              -e NODE_ENV=production \
              -e DATABASE_URL=${{ secrets.PROD_DB_URL }} \
              ghcr.io/${{ github.repository }}:${IMAGE_TAG}

      - name: Notify Slack on success
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "Production deployment successful!",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Production Deploy Successful!* :rocket:\nVersion: `${{ github.ref_name }}`\nTriggered by: ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}

  # ─── Failure Notification ───────────────────────────────────
  notify-failure:
    name: Notify on Failure
    runs-on: ubuntu-latest
    needs: [ lint-and-test, docker-build, deploy-staging ]
    if: failure()
    steps:
      - name: Send failure notification
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {"text": "CI/CD FAILED on ${{ github.ref_name }} — ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Quick Reference Cheat Sheet

### Common GitHub Contexts

| Context | Example Value | Description |
|---------|--------------|-------------|
| `github.ref` | `refs/heads/main` | Full ref of the branch/tag |
| `github.ref_name` | `main` | Short branch or tag name |
| `github.sha` | `a1b2c3d4...` | Full commit SHA |
| `github.actor` | `ravneet-dev` | Who triggered the workflow |
| `github.repository` | `myorg/myapp` | owner/repo |
| `github.event_name` | `push` | Event that triggered workflow |
| `github.run_id` | `12345678` | Unique run ID |
| `runner.os` | `Linux` | Runner operating system |

### Common Action Versions (as of 2024)

| Action | Latest Version |
|--------|---------------|
| `actions/checkout` | v4 |
| `actions/setup-node` | v4 |
| `actions/setup-python` | v5 |
| `actions/setup-java` | v4 |
| `actions/setup-go` | v5 |
| `actions/cache` | v4 |
| `actions/upload-artifact` | v4 |
| `actions/download-artifact` | v4 |
| `docker/build-push-action` | v5 |
| `docker/login-action` | v3 |
| `docker/metadata-action` | v5 |
| `docker/setup-buildx-action` | v3 |

### Workflow Syntax Quick Reference

```yaml
name: Workflow Name
on: [push, pull_request]             # simple list form

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'

env:
  GLOBAL_VAR: value

jobs:
  my-job:
    runs-on: ubuntu-latest
    needs: [other-job]
    if: github.ref == 'refs/heads/main'
    timeout-minutes: 30
    continue-on-error: false
    environment: production
    strategy:
      matrix:
        node: [18, 20, 22]
    outputs:
      my-output: ${{ steps.step-id.outputs.value }}
    env:
      JOB_VAR: value
    steps:
      - name: Step name
        id: step-id
        uses: actions/checkout@v4
        with:
          key: value
        env:
          STEP_VAR: value
        if: success()
        continue-on-error: true
        timeout-minutes: 5
        run: echo "hello"
        shell: bash
        working-directory: ./subdir
```

---

*Guide compiled for: GitHub Actions CI/CD — Complete Reference*
*Covers: Workflows, Events, Triggers, Jobs, Matrix, Steps, Actions, Runners, Caching, Docker, Deployments*
