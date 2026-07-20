# Chapter 10: CI/CD

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: LLMs →](./11-llms.md)

## What is CI/CD?

CI/CD stands for Continuous Integration and Continuous Deployment (or Delivery). It's a set of practices that automate the building, testing, and deployment of software.

### Continuous Integration (CI)

Automatically build and test code whenever changes are pushed to version control. This ensures:
- Code always compiles/builds
- Tests always pass
- Quality standards are maintained
- Integration issues are caught early

### Continuous Deployment (CD)

Automatically deploy code to production after passing all checks. For projects not ready for full automation, Continuous Delivery means code is always ready to deploy but requires manual approval.

## Why Use CI/CD?

### 1. Catch Bugs Early
Automated tests run on every change, catching issues before they reach production.

### 2. Consistent Quality
Every code change goes through the same quality checks - linting, formatting, type checking, tests.

### 3. Fast Feedback
Developers know within minutes if their changes broke something.

### 4. Reduced Manual Work
No more remembering to run tests manually before pushing code.

### 5. Documentation
CI configuration documents your build and test process.

### 6. Confidence
With automated checks, you can merge and deploy with confidence.

## Introduction to GitHub Actions

[GitHub Actions](https://github.com/features/actions) is GitHub's built-in CI/CD platform. It's free for public repositories and includes free minutes for private repositories.

### Why GitHub Actions?

1. **Integrated**: Built into GitHub, no external service needed
2. **Free**: Generous free tier for open source
3. **Simple**: Easy YAML configuration
4. **Powerful**: Extensive marketplace of pre-built actions
5. **Flexible**: Supports Linux, Windows, and macOS runners

### Alternatives

- **GitLab CI/CD**: Integrated with GitLab
- **CircleCI**: Popular third-party service
- **Travis CI**: Early CI/CD pioneer
- **Jenkins**: Self-hosted, highly customizable
- **Azure Pipelines**: Microsoft's CI/CD service

## GitHub Actions Concepts

### Workflows

A workflow is an automated process defined in a YAML file. Workflows are stored in `.github/workflows/` directory.

### Events

Events trigger workflows:
- `push`: Code pushed to repository
- `pull_request`: PR opened or updated
- `schedule`: Run on a schedule (cron)
- `workflow_dispatch`: Manual trigger

### Jobs

A workflow contains one or more jobs that run in parallel (by default) or sequentially.

### Steps

Jobs contain steps - individual tasks like checking out code, installing dependencies, or running tests.

### Runners

Runners are servers that execute workflows. GitHub provides hosted runners with Ubuntu, Windows, and macOS.

## Creating Your First GitHub Actions Workflow

Let's create a workflow that runs linting, formatting checks, type checking, and tests.

### Directory Structure

```
myproject/
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
├── pyproject.toml
└── README.md
```

### Basic CI Workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true

      - name: Set up Python
        run: uv python install 3.12

      - name: Install dependencies
        run: uv sync

      - name: Lint with Ruff
        run: uv run ruff check .

      - name: Check formatting with Ruff
        run: uv run ruff format --check .

      - name: Type check
        run: uv run ty check src/

      - name: Run tests
        run: uv run pytest --cov=src --cov-report=xml

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v5
        with:
          files: ./coverage.xml
```

### Understanding the Workflow

1. **name**: Workflow name shown in GitHub UI
2. **on**: Events that trigger the workflow
3. **jobs**: One or more jobs to run
4. **runs-on**: Operating system for the runner
5. **steps**: Sequential tasks in the job

### Caching

`astral-sh/setup-uv` with `enable-cache: true` caches uv’s download cache. Combined with `uv.lock`, installs stay fast and reproducible.

## Matrix Testing

Test against multiple Python versions (skip EOL releases such as 3.9):

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.11', '3.12', '3.13']

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: uv sync

      - name: Run tests
        run: uv run pytest
```

## Multiple Jobs

Separate concerns into different jobs:

```yaml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv python install 3.12
      - run: uv sync
      - run: uv run ruff check .
      - run: uv run ruff format --check .

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv python install 3.12
      - run: uv sync
      - run: uv run ty check src/

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv python install 3.12
      - run: uv sync
      - run: uv run pytest --cov=src

  all-checks-passed:
    needs: [lint, typecheck, test]
    runs-on: ubuntu-latest
    steps:
      - run: echo "All checks passed!"
```

## Real-World Example

Complete CI/CD workflow for a Python package:

```yaml
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  PYTHON_VERSION: '3.12'

jobs:
  quality:
    name: Code quality checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
      - run: uv python install ${{ env.PYTHON_VERSION }}
      - run: uv sync
      - run: uv run ruff check . --output-format=github
      - run: uv run ruff format --check .
      - run: uv run ty check src/

  test:
    name: Test Python ${{ matrix.python-version }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.11', '3.12', '3.13']

    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
      - run: uv python install ${{ matrix.python-version }}
      - run: uv sync
      - run: uv run pytest -v --cov=src --cov-report=xml --cov-report=term

      - name: Upload coverage
        if: matrix.os == 'ubuntu-latest' && matrix.python-version == '3.12'
        uses: codecov/codecov-action@v5
        with:
          files: ./coverage.xml

  publish:
    name: Publish to PyPI
    needs: [quality, test]
    runs-on: ubuntu-latest
    if: github.event_name == 'release'
    permissions:
      id-token: write  # Trusted Publishing (OIDC)

    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv python install ${{ env.PYTHON_VERSION }}
      - run: uv build
      # Configure Trusted Publishing on PyPI, then:
      - name: Publish package distributions to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
```

## Status Badges

Add status badges to your README:

```markdown
# My Project

![CI](https://github.com/username/repo/workflows/CI/badge.svg)
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

## Best Practices

### 1. Fail Fast

```yaml
strategy:
  fail-fast: true  # Stop all jobs if one fails
  matrix:
    python-version: ['3.11', '3.12', '3.13']
```

### 2. Use Specific Action Versions

```yaml
# Good - pinned version
- uses: actions/checkout@v4

# Bad - unpinned
- uses: actions/checkout@main
```

### 3. Cache Dependencies

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/pyproject.toml') }}
```

### 4. Set Timeout

```yaml
jobs:
  test:
    timeout-minutes: 10  # Kill job after 10 minutes
```

### 5. Use Secrets for Sensitive Data

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

Add secrets: Settings → Secrets and variables → Actions

### 6. Only Run When Needed

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'tests/**'
      - 'pyproject.toml'
```

### 7. Conditional Steps

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

## Local Testing with act

Test GitHub Actions locally with [act](https://github.com/nektos/act):

```bash
# Install act
brew install act  # macOS
# or
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Run workflows locally
act

# Run specific job
act -j test

# Use specific event
act pull_request
```

## Troubleshooting

### Workflow Not Running

Check:
- Workflow file is in `.github/workflows/`
- YAML syntax is valid
- Event triggers match your actions
- Branch name is correct

### Authentication Issues

For private dependencies:

```yaml
- name: Install private package
  env:
    PRIVATE_TOKEN: ${{ secrets.PRIVATE_TOKEN }}
  run: pip install git+https://${PRIVATE_TOKEN}@github.com/user/repo.git
```

### Slow Workflows

Optimize:
- Cache dependencies
- Run jobs in parallel
- Use matrix testing strategically
- Only install necessary dependencies

## Monitoring and Notifications

### Email Notifications

GitHub automatically emails on workflow failures if you're watching the repository.

### Slack Integration

```yaml
- name: Notify Slack
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## Required Status Checks

Make CI checks required before merging:

1. Go to repository Settings
2. Click Branches
3. Add branch protection rule for `main`
4. Check "Require status checks to pass before merging"
5. Select your CI workflow jobs

Now PRs can't be merged until all checks pass!

## Quick Reference

```yaml
# Basic CI with uv
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with:
          enable-cache: true
      - run: uv python install 3.12
      - run: uv sync
      - run: uv run pytest

# Matrix testing
strategy:
  matrix:
    python-version: ['3.11', '3.12', '3.13']
    os: [ubuntu-latest, macos-latest]

# Conditional execution
if: github.ref == 'refs/heads/main'

env:
  PYTHON_VERSION: '3.12'

# Secrets
${{ secrets.SECRET_NAME }}
```

## What's Next?

With CI/CD in place, your code is automatically tested and validated. Next, we'll explore how AI-powered development tools can accelerate your Python development.

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: LLMs →](./11-llms.md)
