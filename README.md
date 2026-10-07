# Python CI Pipeline with GitHub Actions and Email Notifications

A production-style continuous integration pipeline for Python projects, built with GitHub Actions. The pipeline enforces code quality, formatting consistency, test coverage, and security standards on every push and pull request -- then delivers the results directly to your inbox via automated email notifications.

---

## Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [CI Stages](#ci-stages)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Running Locally](#running-locally)
- [Configuring Email Notifications](#configuring-email-notifications)
- [How Notifications Work](#how-notifications-work)
- [Coverage Threshold](#coverage-threshold)
- [Testing a Failed Pipeline](#testing-a-failed-pipeline)
- [License](#license)

---

## Overview

This project demonstrates how to build a robust CI pipeline for a Python codebase using GitHub Actions. It is designed as a reference implementation that can be adapted for real-world projects.

The pipeline performs the following on every push to `main` or pull request targeting `main`:

1. **Linting** -- Flake8 checks code against PEP 8 and catches common programming errors.
2. **Formatting** -- Black verifies that all source code follows a consistent formatting standard.
3. **Testing** -- Pytest runs the full unit test suite with line-level coverage reporting.
4. **Security** -- Bandit scans application source code for potentially insecure patterns.
5. **Dependency Audit** -- pip-audit checks installed packages against known vulnerability databases.
6. **Artifact Upload** -- Coverage reports (XML and HTML) are uploaded as downloadable GitHub Actions artifacts.
7. **Email Notification** -- A summary email is sent after the job completes, regardless of whether it passed or failed.

The pipeline can also be triggered manually through the GitHub Actions UI via `workflow_dispatch`.

---

## Pipeline Architecture

```
Push / Pull Request / Manual Trigger
                |
                v
       GitHub Actions (ubuntu-latest, Python 3.12)
                |
                v
      +--------------------+
      | Checkout Repository |
      +--------------------+
                |
                v
      +--------------------+
      | Install Dependencies|
      +--------------------+
                |
                v
      +--------------------+
      |   Flake8 Linting   |  -- Code quality and PEP 8 compliance
      +--------------------+
                |
                v
      +--------------------+
      | Black Format Check |  -- Consistent code formatting
      +--------------------+
                |
                v
      +--------------------+
      | Pytest + Coverage  |  -- Unit tests with 90% minimum coverage
      +--------------------+
                |
                v
      +--------------------+
      | Bandit Security    |  -- Static security analysis
      +--------------------+
                |
                v
      +--------------------+
      |    pip-audit        |  -- Dependency vulnerability scan
      +--------------------+
                |
                v
      +--------------------+
      | Upload Coverage    |  -- Artifact upload (always runs)
      +--------------------+
                |
                v
      +--------------------+
      | Send Email Report  |  -- Notification (always runs)
      +--------------------+
```

---

## CI Stages

### Flake8 -- Linting

Flake8 analyzes Python source files for style violations, unused imports, undefined variables, and other common code quality issues. The project uses a custom `.flake8` configuration that sets the maximum line length to 88 characters (aligned with Black) and ignores `E203` and `W503` for compatibility.

```bash
flake8 src tests scripts
```

### Black -- Formatting

Black is an opinionated code formatter. In CI, it runs in `--check` mode, which reports formatting violations without modifying files. Developers are expected to run `black .` locally before pushing.

```bash
black --check src tests scripts
```

### Pytest -- Unit Testing and Coverage

Pytest executes the test suite located in `tests/`. Coverage is measured against the `src/` directory and the pipeline enforces a minimum threshold of 90%. Reports are generated in both terminal, XML, and HTML formats.

```bash
pytest --cov=src --cov-report=term-missing --cov-report=xml:coverage.xml --cov-report=html:htmlcov --cov-fail-under=90
```

### Bandit -- Security Scanning

Bandit performs static analysis on Python source code to detect common security issues such as hard-coded credentials, use of `eval()`, `shell=True` in subprocess calls, and other patterns that may introduce vulnerabilities. The test directory is excluded from scanning.

```bash
bandit -r src scripts -x tests
```

### pip-audit -- Dependency Vulnerability Scan

pip-audit checks all installed Python packages against the Python Packaging Advisory Database and the OSV database for known security vulnerabilities.

```bash
pip-audit
```

### Coverage Report Upload

The HTML and XML coverage reports are uploaded as a GitHub Actions artifact named `coverage-report`. This step runs regardless of whether prior steps passed or failed, ensuring reports are always available for review.

### Email Notification

A custom Python script sends an email summary after the job completes. The notification includes the repository name, branch, commit SHA, the user who triggered the workflow, and a direct link to the workflow run. See [Configuring Email Notifications](#configuring-email-notifications) for setup instructions.

---

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── ci.yml                 # GitHub Actions CI workflow definition
├── scripts/
│   └── send_ci_email.py           # Email notification script (SMTP/TLS)
├── src/
│   ├── __init__.py
│   └── calculator.py              # Application module with arithmetic functions
├── tests/
│   └── test_calculator.py         # Pytest unit tests for the calculator module
├── .flake8                        # Flake8 linter configuration
├── .gitignore
├── pyproject.toml                 # Black and Pytest configuration
├── requirements-dev.txt           # Development and CI dependencies
├── EXPLANATION.md                 # In-depth explanation of tools and workflow
└── README.md
```

---

## Tech Stack

| Category             | Tool / Technology               |
|----------------------|---------------------------------|
| Language             | Python 3.12                     |
| CI Platform          | GitHub Actions                  |
| Linter               | Flake8 7.1.2                    |
| Formatter            | Black >= 26.3.1                 |
| Test Framework       | Pytest >= 9.0.3                 |
| Coverage             | pytest-cov 6.0.0                |
| Security Scanner     | Bandit 1.8.3                    |
| Dependency Audit     | pip-audit 2.8.0                 |
| Email                | Python smtplib (SMTP over TLS)  |

---

## Getting Started

### Prerequisites

- Python 3.12 or later
- pip
- A GitHub repository with Actions enabled

### Clone the Repository

```bash
git clone https://github.com/<your-username>/python-github-actions-ci-email.git
cd python-github-actions-ci-email
```

### Set Up a Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Install Dependencies

```bash
pip install -r requirements-dev.txt
```

---

## Running Locally

Run the full test suite:

```bash
pytest
```

Run tests with coverage reporting:

```bash
pytest --cov=src --cov-report=term-missing
```

Run the linter:

```bash
flake8 src tests scripts
```

Check formatting:

```bash
black --check src tests scripts
```

Auto-format code:

```bash
black src tests scripts
```

Run the security scanner:

```bash
bandit -r src scripts -x tests
```

Run the dependency audit:

```bash
pip-audit
```

---

## Configuring Email Notifications

The email notification step requires SMTP credentials stored as GitHub repository secrets.

### Required Secrets

Navigate to your repository: **Settings > Secrets and variables > Actions > New repository secret**

Create the following secrets:

| Secret Name          | Description                                  |
|----------------------|----------------------------------------------|
| `SMTP_SERVER`        | SMTP server hostname                         |
| `SMTP_PORT`          | SMTP server port (typically `587` for TLS)   |
| `SMTP_USERNAME`      | Email address used to authenticate           |
| `SMTP_PASSWORD`      | SMTP password or app-specific password       |
| `CI_EMAIL_RECIPIENT` | Email address that receives CI notifications |

### Gmail Configuration Example

| Secret Name          | Value                         |
|----------------------|-------------------------------|
| `SMTP_SERVER`        | `smtp.gmail.com`              |
| `SMTP_PORT`          | `587`                         |
| `SMTP_USERNAME`      | `your-email@gmail.com`        |
| `SMTP_PASSWORD`      | Your Google App Password      |
| `CI_EMAIL_RECIPIENT` | `team@example.com`            |

> **Important:** Do not commit SMTP credentials to the repository. For Gmail, generate an [App Password](https://support.google.com/accounts/answer/185833) rather than using your account password.

---

## How Notifications Work

The email notification step uses the `if: always()` conditional, which ensures it executes regardless of whether the preceding CI checks passed or failed. It also uses `continue-on-error: true` so that a misconfigured email setup does not mark an otherwise successful pipeline as failed.

The notification script reads the `CI_STATUS` environment variable (set to the value of `job.status` by the workflow) and composes an email that includes:

- CI result (passed or failed) in the subject line
- Repository name, workflow name, branch, and commit SHA
- The GitHub user who triggered the run
- A direct link to the GitHub Actions workflow run

---

## Coverage Threshold

The pipeline enforces a minimum coverage threshold of **90%**. If the test suite covers less than 90% of the code in `src/`, the Pytest step will fail.

To adjust this threshold, modify the `--cov-fail-under` value in [`.github/workflows/ci.yml`](.github/workflows/ci.yml):

```yaml
--cov-fail-under=90
```

---

## Testing a Failed Pipeline

To observe the failure notification flow, temporarily introduce a failing assertion:

```python
def test_add():
    assert add(2, 3) == 100  # intentionally wrong
```

Commit and push the change. Pytest will fail, and the email notification will report a failed CI status.

Restore the correct assertion and push again to confirm the success notification:

```python
def test_add():
    assert add(2, 3) == 5
```

---

## License

This project is provided as a reference implementation for learning and portfolio purposes.
