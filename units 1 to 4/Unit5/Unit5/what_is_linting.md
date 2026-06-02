# What is Linting / Lint?

**Linting** means automatically checking your code for errors, bugs, and bad style — before you even run it.

The word comes from a Unix tool called **lint** (1978) that scanned C code for suspicious patterns. Like how a lint roller picks up small unwanted things from fabric, a linter picks up small unwanted things from your code.

---

## What a Linter Actually Checks

### 1. Errors — things that will definitely break
- Using a variable before declaring it
- Calling a function that doesn't exist
- Missing a required argument

### 2. Potential Bugs — things that probably will break
- `if (x = 5)` instead of `if (x == 5)` (assignment inside condition)
- Unreachable code after a `return` statement
- Comparing with `==` instead of `===` in JavaScript

### 3. Code Style — things that make code inconsistent or hard to read
- Inconsistent indentation (tabs vs spaces)
- Lines that are too long
- Unused variables or imports
- Missing semicolons (or having them when the team agreed not to)

---

## A Real Example (JavaScript with ESLint)

You write this code:

```js
const name = "Ravneet"
let age = 25

console.log(Name)   // wrong variable name — capital N
```

The linter immediately flags:

```
Line 1: 'name' is assigned but never used
Line 3: 'age' is assigned but never used
Line 5: 'Name' is not defined
```

You catch the bug **before** running the code — not after deploying it to production.

---

## Common Linters by Language

| Language | Popular Linter | Install Command |
|----------|---------------|----------------|
| JavaScript / TypeScript | ESLint | `npm install eslint --save-dev` |
| Python | Flake8 | `pip install flake8` |
| Python (faster) | Ruff | `pip install ruff` |
| Python (thorough) | Pylint | `pip install pylint` |
| CSS / SCSS | Stylelint | `npm install stylelint --save-dev` |
| Go | golangci-lint | `brew install golangci-lint` |
| Rust | Clippy | built into Rust toolchain |
| PHP | PHP_CodeSniffer | `composer require squizlabs/php_codesniffer` |
| Ruby | RuboCop | `gem install rubocop` |

---

## Setting Up ESLint (JavaScript / Node.js Example)

### Step 1 — Install ESLint

```bash
npm install eslint --save-dev
```

### Step 2 — Create config file `.eslintrc.json`

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": "eslint:recommended",
  "rules": {
    "no-unused-vars": "error",
    "no-undef": "error",
    "eqeqeq": "warn",
    "semi": ["error", "always"],
    "indent": ["error", 2]
  }
}
```

### Step 3 — Add script to `package.json`

```json
{
  "scripts": {
    "lint": "eslint src/**/*.js",
    "lint:fix": "eslint src/**/*.js --fix"
  }
}
```

### Step 4 — Run the linter

```bash
npm run lint        # check for issues
npm run lint:fix    # auto-fix what it can
```

---

## Setting Up Flake8 (Python Example)

### Install

```bash
pip install flake8
```

### Create config file `.flake8`

```ini
[flake8]
max-line-length = 88
exclude =
    .git,
    __pycache__,
    venv,
    .venv
ignore = E203, W503
```

### Run

```bash
flake8 src/           # lint the src/ folder
flake8 app.py         # lint a single file
```

---

## What Linter Output Looks Like

### ESLint output (JavaScript)

```
/home/ravneet/myapp/src/index.js
  3:7   error  'age' is assigned a value but never used  no-unused-vars
  7:1   error  'Name' is not defined                      no-undef
  12:5  warning  Expected '===' and instead saw '=='      eqeqeq

✖ 3 problems (2 errors, 1 warning)
```

### Flake8 output (Python)

```
src/app.py:5:1: F401 'os' imported but unused
src/app.py:12:80: E501 line too long (95 > 88 characters)
src/app.py:18:1: W291 trailing whitespace
```

Each line follows the format: `file:line:column: CODE message`

---

## Auto-fixing with Linters

Many linters can **automatically fix** simple issues for you:

```bash
# ESLint — fix what it can automatically
eslint src/ --fix

# Prettier (formatter, works alongside ESLint)
prettier --write src/

# Ruff (Python) — auto-fix
ruff check src/ --fix

# Clippy (Rust) — apply suggestions
cargo clippy --fix
```

Auto-fix handles things like:
- Adding/removing semicolons
- Fixing indentation
- Removing trailing whitespace
- Sorting imports

It **cannot** auto-fix things that require logic decisions, like renaming undefined variables.

---

## Linting vs Formatting vs Type Checking

These three are often confused but do different things:

| Tool Type | What it Does | Example Tools |
|-----------|-------------|---------------|
| **Linter** | Finds bugs, bad patterns, unused code | ESLint, Flake8, Clippy |
| **Formatter** | Makes code look consistent (spacing, quotes, line breaks) | Prettier, Black, gofmt |
| **Type Checker** | Verifies types are correct (string vs number, etc.) | TypeScript (`tsc`), mypy (Python) |

In a real project you use **all three together**:

```bash
npm run lint          # ESLint — find bugs
npm run format        # Prettier — fix style
npm run type-check    # TypeScript — check types
```

---

## Why Linting Matters in CI / GitHub Actions

In a CI pipeline, linting runs **first** — before tests, before building — because:

1. It is the fastest check (usually takes 5–30 seconds)
2. If code has obvious errors, there is no point running slower tests
3. It enforces consistent code style across the whole team automatically
4. It blocks bad code from ever reaching the `main` branch

### Example in GitHub Actions

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  lint:
    name: Lint Code
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

      - name: Run ESLint
        run: npm run lint

      # If lint fails, the entire workflow stops here.
      # Tests and deployment never run.

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: lint          # only runs if lint job passes
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm test

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test          # only runs if tests pass
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci && npm run build
```

### The CI pipeline order

```
Lint (fast, 10-30s)
  → Tests (medium, 1-5 min)
    → Build (slow, 2-10 min)
      → Deploy
```

If linting fails, the pipeline stops immediately — saving time and compute resources.

---

## Linting Rules Severity Levels

Most linters support three levels for each rule:

| Level | Meaning | Effect |
|-------|---------|--------|
| `off` / `0` | Rule is disabled | Nothing happens |
| `warn` / `1` | Rule is a warning | Shows message, does NOT fail CI |
| `error` / `2` | Rule is an error | Shows message, FAILS CI |

```json
{
  "rules": {
    "no-unused-vars": "error",    // fails CI
    "no-console": "warn",         // shows warning, CI still passes
    "semi": "off"                 // completely ignored
  }
}
```

---

## Ignoring Specific Lines or Files

Sometimes you need to intentionally bypass a linting rule:

### ESLint (JavaScript)

```js
// Disable for the next line only
// eslint-disable-next-line no-unused-vars
const tempDebugVar = "testing";

// Disable for an entire block
/* eslint-disable no-console */
console.log("debug info");
console.log("more debug");
/* eslint-enable no-console */
```

### Ignore entire files — `.eslintignore`

```
node_modules/
dist/
build/
*.min.js
coverage/
```

### Flake8 (Python)

```python
import os  # noqa: F401          # ignore "imported but unused" for this line

x = 1+1    # noqa                 # ignore ALL rules for this line
```

### Ignore files — `.flake8` config

```ini
[flake8]
exclude =
    migrations/,
    venv/,
    __pycache__/
```

---

## Summary

| Question | Answer |
|----------|--------|
| What is linting? | Automatically scanning code for errors, bugs, and style issues |
| When does it run? | Before tests, on every save (in editor) or on every push (in CI) |
| Does it run the code? | No — it reads and analyzes the code without executing it |
| Can it fix issues? | Yes, for simple style issues. Logic bugs must be fixed manually |
| Why use it in CI? | Catches problems early, enforces team standards, blocks bad code |
| Most popular JS linter? | ESLint |
| Most popular Python linter? | Flake8 or Ruff |

---

*In short: a linter is your automated code reviewer that never sleeps, never misses a thing, and enforces the same standards for every single developer on the team.*
