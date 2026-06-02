# Scenario 14: Python CI with Pytest and Matrix

## 1. Scenario

You are working on a project named `python-ci-demo`. The task is to implement GitHub Actions for **Python pytest matrix**. This scenario focuses on **Python 3.10, 3.11, 3.12** and helps students understand a real CI/CD use case through actual YAML implementation.

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
python-ci-demo/
└── .github/
    └── workflows/
        └── python-ci.yml
```

---


## Project Structure

```text
python-ci-demo/
├── calculator.py
├── test_calculator.py
├── requirements.txt
└── .github/
    └── workflows/
        └── python-ci.yml
```

## Source Code

### `calculator.py`

```python
def add(a, b):
    return a + b
```

### `test_calculator.py`

```python
from calculator import add

def test_add():
    assert add(2, 3) == 5
```

### `requirements.txt`

```text
pytest
```

## Workflow File

```yaml
name: Python CI

on:
  push:
  pull_request:

jobs:
  python-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - name: Checkout repository code
        uses: actions/checkout@v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Python dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run pytest
        run: pytest
```

## Step-by-Step Implementation

1. Create Python files.
2. Add `pytest` in `requirements.txt`.
3. Add workflow.
4. Push code.
5. Check matrix jobs for Python 3.10, 3.11, and 3.12.

## Expected Output

```text
1 passed
```

## Explanation

The workflow tests the same Python code on multiple Python versions using matrix strategy.

## Common Mistakes

| Mistake | Result | Fix |
|---|---|---|
| Missing `pytest` | Command not found | Add `pytest` to requirements |
| Test file not named correctly | No tests discovered | Use `test_*.py` |
| Wrong Python version | Setup fails | Use supported version |


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
