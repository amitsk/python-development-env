# Chapter 10: CI/CD

[← Previous: Logging](./09-logging.md) | [Back to README](./README.md) | [Next: LLMs →](./11-llms.md)

## What is CI/CD?

CI/CD stands for Continuous Integration and Continuous Deployment (or Delivery). It is the practice of letting a machine build, test, and (when you are ready) ship your software so humans do not have to remember the checklist.

### Continuous Integration (CI)

Automatically build and test code whenever changes are pushed to version control:

- The project still installs
- Tests still pass
- Lint, format, and type checks still agree with the repo
- Integration surprises show up on the pull request, not on Friday afternoon

### Continuous Deployment (CD)

Automatically deploy to production after every green pipeline. If that sentence makes you nervous, use **Continuous Delivery** instead: every main-branch commit is *ready* to deploy, and a human still presses the button.

## Why Use CI/CD?

1. **Catch bugs early.** Tests run on every change, including the one you were "just tweaking."
2. **Consistent quality.** Linting, formatting, types, and tests run the same way for everyone.
3. **Fast feedback.** Minutes, not "I'll run the suite later."
4. **Less manual work.** The workflow file *is* the checklist.
5. **Documentation.** New contributors can read `.github/workflows/ci.yml` instead of a wiki page last updated during a different Python.
6. **Confidence.** Merging is less of a coin flip.

## Introduction to GitHub Actions

[GitHub Actions](https://github.com/features/actions) is GitHub's built-in CI/CD platform. Public repositories get a generous free tier; private repositories get a monthly allotment of minutes.

### Why GitHub Actions?

1. **Integrated** — lives next to the code, no extra service to provision
2. **Free enough** — especially for open source
3. **YAML, not a GUI maze** — the pipeline is versioned with the repo
4. **Marketplace** — pre-built actions for checkout, caching, publishing
5. **Flexible runners** — Linux, Windows, and macOS

### Alternatives

- **GitLab CI/CD** — if the repo already lives on GitLab
- **CircleCI** — third-party, still widely used
- **Travis CI** — the early default; less central than it used to be
- **Jenkins** — self-hosted, infinitely configurable, your problem to keep alive
- **Azure Pipelines** — Microsoft's CI service

This chapter uses GitHub Actions because that is where most of the Python projects you will clone already run.

## GitHub Actions Concepts

### Workflows

A workflow is an automated process defined in YAML under `.github/workflows/`.

### Events

Events trigger workflows:

- `push` — code landed on a branch
- `pull_request` — a PR opened or updated
- `schedule` — cron, for the jobs nobody remembers to run
- `workflow_dispatch` — a button in the Actions tab
- `release` — a GitHub Release was published

### Jobs, Steps, Runners

A workflow contains **jobs**. Jobs run in parallel unless you say otherwise with `needs:`.

Each job is a list of **steps**: checkout, install uv, run Ruff, run pytest.

**Runners** are the machines. `ubuntu-latest` is the default that actually matches most Python packaging.

## Creating Your First GitHub Actions Workflow

A workflow that lints, checks formatting, type-checks, tests, and uploads coverage:

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
        uses: actions/checkout@v7

      - name: Install uv
        uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true

      - name: Set up Python
        run: uv python install 3.14

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
        uses: codecov/codecov-action@v7
        with:
          files: ./coverage.xml
```

### Understanding the Workflow

1. **name** — shown in the GitHub UI and used by some badge URLs
2. **on** — which events start a run
3. **jobs** — one or more jobs
4. **runs-on** — the runner image
5. **steps** — sequential tasks inside a job

`actions/checkout@v7` is the current checkout action. Older `@v4` examples still float around the internet; they run on Node 20, and GitHub is retiring that runtime. Prefer v7.

`astral-sh/setup-uv` **dropped moving major tags**. `@v5` or `@v10` will not quietly track new releases. Pin a full version such as `@v10.3.3`. Check [the setup-uv releases](https://github.com/astral-sh/setup-uv/releases) when you copy this file six months from now.

If you want to be extra careful about supply chain risk, pin third-party actions to a full commit SHA and leave the human-readable version in a comment:

```yaml
# Resolve the SHA from the action's GitHub release; do not copy a hash from memory
# - uses: astral-sh/setup-uv@<40-character-sha>  # v10.3.3
```

Readable tutorials use version tags. Production repos that handle secrets often pin SHAs and let Dependabot bump both the SHA and the comment.

### Caching

`astral-sh/setup-uv` with `enable-cache: true` caches uv's download cache. Combined with `uv.lock`, installs stay fast and reproducible. You do not need a separate `actions/cache` step for pip's `~/.cache/pip` — this tutorial does not use pip as the CI installer.

## Matrix Testing

Test the versions you claim to support. Skip EOL releases; as of this writing that means treating 3.13–3.15 as the interesting matrix:

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
        python-version: ['3.13', '3.14', '3.15']

    steps:
      - uses: actions/checkout@v7

      - name: Install uv
        uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: uv sync

      - name: Run tests
        run: uv run pytest
```

`fail-fast: false` lets the 3.15 / Windows cell finish even if macOS 3.13 already failed. That extra signal is usually worth the minutes.

## Multiple Jobs

Split lint, types, and tests so a formatting nit does not hide a test failure — and so they can run in parallel:

```yaml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install 3.14
      - run: uv sync
      - run: uv run ruff check .
      - run: uv run ruff format --check .

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install 3.14
      - run: uv sync
      - run: uv run ty check src/

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install 3.14
      - run: uv sync
      - run: uv run pytest --cov=src

  all-checks-passed:
    needs: [lint, typecheck, test]
    runs-on: ubuntu-latest
    steps:
      - run: echo "All checks passed!"
```

The `all-checks-passed` job is a convenient single required status check: branch protection can require that one name instead of three.

## Real-World Example

A package workflow: quality on 3.14, tests on a version × OS matrix, publish to PyPI with Trusted Publishing when a GitHub Release is published.

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
  PYTHON_VERSION: '3.14'

jobs:
  quality:
    name: Code quality checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
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
        python-version: ['3.13', '3.14', '3.15']

    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install ${{ matrix.python-version }}
      - run: uv sync
      - run: uv run pytest -v --cov=src --cov-report=xml --cov-report=term

      - name: Upload coverage
        if: matrix.os == 'ubuntu-latest' && matrix.python-version == '3.14'
        uses: codecov/codecov-action@v7
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
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install ${{ env.PYTHON_VERSION }}
      - run: uv build
      - name: Publish package distributions to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
```

**Trusted Publishing** is the current PyPI story: you prove GitHub identity with OIDC instead of stuffing a long-lived `PYPI_TOKEN` into Actions secrets. Configure the publisher on PyPI (project → Publishing), then the `id-token: write` permission above is enough. [PyPI's trusted publishers docs](https://docs.pypi.org/trusted-publishers/) are the source of truth for the click-path.

`pypa/gh-action-pypi-publish@release/v1` is the maintained tag line; still prefer a SHA pin if this workflow is the one thing standing between you and an unexpected package upload.

## Status Badges

Put the boring green check where people actually look — the README:

```markdown
# My Project

![CI](https://github.com/username/repo/actions/workflows/ci.yml/badge.svg)
[![codecov](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)](https://codecov.io/gh/username/repo)
```

Point the CI badge at the workflow *file* (`ci.yml`). The older `/workflows/CI/badge.svg` form still appears in copy-pasted READMEs; the file-based URL is the one GitHub documents now.

## Best Practices

### 1. Fail Fast (When You Mean It)

```yaml
strategy:
  fail-fast: true  # cancel remaining matrix cells if one fails
  matrix:
    python-version: ['3.13', '3.14', '3.15']
```

Use `true` when a single failure makes the rest of the matrix a waste of minutes. Use `false` (as in the full example above) when you want every Python version's opinion.

### 2. Pin Action Versions (or SHAs)

```yaml
# Good enough for a tutorial and most repos — a released tag
- uses: actions/checkout@v7
- uses: astral-sh/setup-uv@v10.3.3

# Extra careful — immutable commit SHA (resolve from the release)
# - uses: astral-sh/setup-uv@<full-sha>  # v10.3.3

# Bad — a branch that moves under you
- uses: actions/checkout@main
```

Tags can be retargeted. SHAs cannot. First-party GitHub actions (`actions/checkout`) are a milder risk than a random Marketplace action with access to `secrets.*`.

### 3. Cache Dependencies

Prefer uv's own cache via `enable-cache: true`. Reach for `actions/cache` only if you are caching something uv does not already handle.

### 4. Set a Timeout

```yaml
jobs:
  test:
    timeout-minutes: 10  # hung pytest is still a failure
```

A job that waits forever is indistinguishable from a job that works, until your bill arrives.

### 5. Use Secrets for Sensitive Data

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
  run: ./deploy.sh
```

Add secrets at **Settings → Secrets and variables → Actions**. Never commit them. Never print them. GitHub masks values that exactly match a secret; it cannot mask a secret you `echo` after concatenating it with a greeting.

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

Handy for monorepos. Easy to get wrong: forget `pyproject.toml` and a dependency bump will skip CI. If in doubt, run on every change.

### 7. Conditional Steps

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

## Local Testing with act

[act](https://github.com/nektos/act) runs GitHub Actions workflows on your machine. It is not a perfect replica of `ubuntu-latest`, but it catches YAML typos and missing env vars before you push.

```bash
# Install act
brew install act  # macOS
# or
curl -s https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Run workflows locally
act

# Run a specific job
act -j test

# Simulate a pull_request event
act pull_request
```

The first run downloads a runner image. After that it is merely slow in the ordinary Docker way.

## Troubleshooting

### Workflow Not Running

Check:

- The file is under `.github/workflows/` and ends in `.yml` or `.yaml`
- YAML is valid (two spaces, not a creative mix of tabs)
- The `on:` events match what you actually did
- The branch name is really `main` (or whatever you wrote)

### Authentication Issues

For private dependencies:

```yaml
- name: Install private package
  env:
    PRIVATE_TOKEN: ${{ secrets.PRIVATE_TOKEN }}
  run: uv add "git+https://${PRIVATE_TOKEN}@github.com/user/repo.git"
```

Prefer deploy keys or a fine-grained token with the smallest possible scope. A personal access token with `repo` on your whole account is how private packages become public incidents.

### Slow Workflows

- Keep `enable-cache: true` on setup-uv
- Run independent jobs in parallel
- Do not install docs extras just to run pytest
- Upload coverage from one matrix cell, not twelve

## Monitoring and Notifications

### Email

GitHub emails workflow failures if you watch the repository. That is enough until it is not.

### Slack

Third-party notify actions exist (for example `8398a7/action-slack`). Treat them like any other third-party action: pin a version or SHA, store the webhook as a secret, and only fire on `failure()` unless you enjoy channel noise.

```yaml
- name: Notify Slack
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## Required Status Checks

A green CI run that nobody is required to wait for is a suggestion. Make it a gate:

1. Repository **Settings**
2. **Rules** → **Rulesets**, or the older **Branches** → branch protection rule
3. Target `main`
4. Enable **Require status checks to pass before merging**
5. Select the job names from your workflow (`test`, `quality`, or `all-checks-passed`)

Pull requests then stay unmergeable until those checks report success. GitHub's UI labels move around; the idea does not: named checks, matching job names, limited bypass.

## Quick Reference

```yaml
# Basic CI with uv
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: astral-sh/setup-uv@v10.3.3
        with:
          enable-cache: true
      - run: uv python install 3.14
      - run: uv sync
      - run: uv run ruff check .
      - run: uv run ruff format --check .
      - run: uv run ty check src/
      - run: uv run pytest --cov=src --cov-report=xml

# Matrix testing
strategy:
  matrix:
    python-version: ['3.13', '3.14', '3.15']
    os: [ubuntu-latest, macos-latest]

# Conditional execution
if: github.ref == 'refs/heads/main'

env:
  PYTHON_VERSION: '3.14'

# Secrets
${{ secrets.SECRET_NAME }}

# PyPI Trusted Publishing
permissions:
  id-token: write
# uses: pypa/gh-action-pypi-publish@release/v1
```

## What's Next?

CI will faithfully run whatever you committed, including the function an AI autocomplete invented at 1 a.m. Next we look at LLM-powered tools — Copilot, Cursor, Claude Code, Codex, and Grok Build — and the parts of the job they do not get to skip.

[← Previous: Logging](./09-logging.md) | [Back to README](./README.md) | [Next: LLMs →](./11-llms.md)
