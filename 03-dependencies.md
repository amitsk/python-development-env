# Chapter 3: Managing Dependencies

[← Previous: Virtual Environments](./02-virtual-environments.md) | [Back to README](./README.md) | [Next: Version Control →](./04-version-control.md)

## What are Dependencies?

Dependencies are packages your project does not contain but cannot run without. Other people wrote them, tested them, and published them. You install them instead of reinventing HTTP, date parsing, or a web framework on a Tuesday.

Need HTTP? `requests` or `httpx`. Dates? `python-dateutil`. A web API? `fastapi` (or Flask, Django, Litestar). Data frames? `pandas` and `numpy`. Those names are dependencies.

## Understanding Dependencies with a Simple Example

Fetching JSON from an API with only the standard library works. It is also more code than you want in a tutorial — or in production, once timeouts, encodings, and error bodies show up.

**Without a third-party HTTP library:**

```python
import urllib.request
import json

# Works. You will grow to resent it.
response = urllib.request.urlopen('https://api.example.com/data')
data = json.loads(response.read().decode('utf-8'))
```

**With `requests`:**

```python
import requests

response = requests.get('https://api.example.com/data', timeout=10)
data = response.json()
```

`requests` is a dependency: code you did not write, doing the uninteresting part well.

## Why Manage Dependencies?

Install names and versions on purpose, not by accident.

- **Reproducibility** — if you developed against `requests` 2.34, the next machine should not silently get a different major version.
- **Collaboration** — teammates (and CI) need a declared list, not a tour of your laptop.
- **Deployment** — production installs from the same specification you tested.
- **Isolation** — project A and project B can disagree about versions because they do not share an environment.

The mechanism is a project metadata file plus a lockfile. pip and `requirements.txt` are the background. uv and `pyproject.toml` are the workflow.

## The Python Package Index (PyPI)

[PyPI](https://pypi.org) is the public repository of Python packages. When you `uv add requests`, you are asking an index (usually PyPI) for a wheel or an sdist, then recording that choice.

Look at a project page before you depend on it. Downloads, recent releases, a license you can live with, and a repository that is not a ghost town are all cheaper than debugging a package that peaked in 2018.

## Introduction to pip

`pip` is the standard installer. It ships with Python. You will see it in every older tutorial, every Dockerfile from 2021, and every “just run this” snippet. You should recognize it. You should not make it your project manager.

### Basic pip Usage

```bash
# Install a package
pip install requests

# Exact version
pip install requests==2.34.2

# Latest allowed
pip install --upgrade requests

pip uninstall requests
pip list
pip show requests
```

Run these *inside* a virtual environment. Global `pip install` is how system Python becomes a junk drawer.

### Requirements Files

The traditional declaration is a `requirements.txt`:

**requirements.txt:**

```
requests==2.34.2
pandas==3.1.1
numpy==2.5.4
```

```bash
pip install -r requirements.txt
```

### Limitations of pip

pip installs packages. It does not run a project.

- It is slow on large trees.
- A hand-written `requirements.txt` usually lists direct dependencies only. Transitive versions drift unless you freeze everything.
- Resolution can pick a combination that later explodes at import time.
- Virtualenvs are a separate tool.

That is the gap uv fills: one project workflow, a lockfile, and an installer that does not take the afternoon off.

## Managing Dependencies with uv

Chapter 2 introduced uv as an environment tool. It is also the dependency tool.

Speed matters (often 10–100× pip), but the practical reasons to use it are a real resolver, `uv.lock`, environments that match the project, and a CLI that updates `pyproject.toml` for you. It still understands pip and `requirements.txt` when you are inheriting someone else’s repo.

### The daily path: `uv add`, `uv remove`, `uv sync`

Once you have a project (`uv init` from Chapter 2), you do not “pip install” into it. You declare a dependency and sync.

```bash
# Runtime dependency — updates pyproject.toml and the lockfile, installs into .venv
uv add requests

# Development tools — writes the `dev` group ([dependency-groups], PEP 735)
uv add --dev pytest

# Named group if you want to split lint from test
uv add --group lint ruff

# Constraint at add time
uv add "requests>=2.34.0"

# Take it back out
uv remove requests
uv remove --dev pytest
```

Install (or reinstall) from the lockfile:

```bash
# Project + the default `dev` group
uv sync

# Production-like: runtime only, no dev tools
uv sync --no-dev
```

`uv add --dev pytest` is how local tools get into `[dependency-groups]`. `uv sync` installs them by default so `uv run pytest` works on a fresh clone. Production images and deploy steps should use `uv sync --no-dev`.

Run the project without activating the environment:

```bash
uv run python main.py
uv run pytest
uvx ruff check .     # one-off, not a project dependency
```

Inspect what you actually got:

```bash
uv tree
```

`uv tree` is the view you want: your packages, and what they pulled in. `uv pip list` exists if you want a flat list. Prefer the tree.

### Using pyproject.toml (Modern Approach)

The modern declaration is `pyproject.toml`.

**Runtime dependencies** go in `[project.dependencies]`.
**Local development tools** (pytest, Ruff, type checkers) go in **`[dependency-groups]`** ([PEP 735](https://peps.python.org/pep-0735/)), not in published extras.

**pyproject.toml:**

```toml
[project]
name = "myapp"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.34.0",
    "pandas>=3.1.0",
    "numpy>=2.5.0",
]

[dependency-groups]
dev = [
    "pytest>=9.2.0",
    "ruff>=0.17.0",
    "ty",
]
```

> **Optional dependencies vs dependency groups:**
> `[project.optional-dependencies]` defines *published extras* (for example `pip install mylib[excel]` or `fastapi[standard]`).
> `[dependency-groups]` is for tools you need while developing and should not ship as package extras. Prefer groups for pytest, Ruff, and ty.

You rarely hand-edit the lists. `uv add` / `uv add --dev` / `uv remove` keep `pyproject.toml` and `uv.lock` in agreement. Edit by hand when you are tightening constraints or adding a comment.

### Lock Files

`uv.lock` records exact versions of everything, including transitive dependencies:

```bash
uv sync              # install exact versions from the lockfile
uv lock --upgrade    # refresh the lockfile toward latest allowed versions
```

**Commit `uv.lock`.** That is how every developer and every CI job install the same tree. `pyproject.toml` says what is allowed; the lockfile says what you actually shipped.

### pip-compatible interface (short note)

uv also speaks pip, which is handy for a throwaway environment or a `requirements.txt` you have not migrated yet:

```bash
uv pip install requests
uv pip install "requests>=2.34.0,<3.0.0"
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
```

For a project you own, `uv add` / `uv remove` / `uv sync` are the source of truth. `uv pip install` bypasses `pyproject.toml` unless you are careful. Use it as compatibility, not as the workflow.

If some other system still wants a requirements file, export from the lockfile rather than freezing an ad-hoc environment:

```bash
uv export --format requirements.txt -o requirements.txt
```

## Practical Example: Building a Web Scraper

A small scraper is enough to see the whole loop: init, add, sync, run.

### Step 1: Set Up Project

```bash
uv python install 3.14   # if you do not already have it
uv init --python 3.14 web-scraper
cd web-scraper
```

### Step 2: Add Dependencies

```bash
uv add "requests>=2.34.0" "beautifulsoup4>=4.15.0"
uv add --dev "pytest>=9.2.0" "pytest-cov>=7.1.0"
```

Or write `pyproject.toml` by hand (then `uv sync`):

```toml
[project]
name = "web-scraper"
version = "0.1.0"
description = "A simple web scraper"
requires-python = ">=3.13"
dependencies = [
    "requests>=2.34.0",
    "beautifulsoup4>=4.15.0",
]

[dependency-groups]
dev = [
    "pytest>=9.2.0",
    "pytest-cov>=7.1.0",
]
```

### Step 3: Install Dependencies

```bash
uv sync
uv tree
```

### Step 4: Write the Scraper

**scraper.py:**

```python
import requests
from bs4 import BeautifulSoup


def fetch_page_title(url: str) -> str:
    """Fetch and return the title of a web page.

    Args:
        url: The URL to fetch

    Returns:
        The page title

    Raises:
        requests.RequestException: If the request fails
    """
    response = requests.get(url, timeout=10)
    response.raise_for_status()

    soup = BeautifulSoup(response.content, "html.parser")
    title = soup.find("title")

    return title.string if title else "No title found"


if __name__ == "__main__":
    url = "https://www.python.org"
    title = fetch_page_title(url)
    print(f"Page title: {title}")
```

### Step 5: Test It

```bash
uv run python scraper.py
```

Output:

```
Page title: Welcome to Python.org
```

`requests` and `beautifulsoup4` did the heavy lifting. That is the point of a dependency: less code of yours sitting between you and the page title.

## Dependency Version Constraints

Constraints live in `pyproject.toml`. The lockfile then pins the exact build.

```toml
[project]
dependencies = [
    "requests==2.34.2",        # Exact version
    "pandas>=3.1.0",           # Minimum version
    "numpy>=2.5.0,<3.0.0",     # Version range
    "flask~=3.1.0",            # Compatible release (~= 3.1.0 means >=3.1.0, <3.2.0)
]
```

### Semantic Versioning (SemVer)

Most Python packages use `MAJOR.MINOR.PATCH`:

- **MAJOR** — breaking changes
- **MINOR** — features, backward compatible in theory
- **PATCH** — bug fixes

`requests 2.34.2` is major 2, minor 34, patch 2. Treat a major bump as a deliberate upgrade, not a surprise from `uv lock --upgrade`.

### Best Practices for Version Constraints

```toml
# Good — upper bound on the next breaking major
"requests>=2.34.0,<3.0.0"

# Good — mature libraries where you trust the next minor
"numpy>=2.5.0"

# Use carefully — exact pin in pyproject.toml
"pandas==3.1.1"

# Avoid — no lower bound, no upper bound, just a name
"requests"
```

Exact pins belong in `uv.lock`, which you commit. `pyproject.toml` should say what you can accept so `uv lock --upgrade` has room to move.

## Development vs. Production Dependencies

Split them.

**Development:** pytest, Ruff, type checkers, coverage, docs builders. You need them to work on the code. Production does not.

**Production (runtime):** the libraries the application imports to run — web framework, database driver, HTTP client.

**Example pyproject.toml:**

```toml
[project]
name = "myapp"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    # Runtime only
    "fastapi[standard]>=0.162.0",
    "sqlalchemy>=2.1.0",
    "pydantic>=2.14.0",
]

[dependency-groups]
dev = [
    "pytest>=9.2.0",
    "pytest-cov>=7.1.0",
    "ruff>=0.17.0",
    "ty",
]
```

`fastapi[standard]` is the extra you want for a web app: FastAPI plus the usual server and tooling. Bare `fastapi` is the library only.

```bash
# Development clone (includes the default dev group)
uv sync

# Production-like (no pytest, no Ruff)
uv sync --no-dev
```

## Exploring Packages

Do a little homework before `uv add`.

### Check on PyPI

[pypi.org/project/requests](https://pypi.org/project/requests) shows documentation links, versions, download stats, license, and the package’s own dependencies.

### Check the Repository

Most project pages link to source (often GitHub):

- Read the README
- Look at recent commits and release cadence
- Skim open issues for “abandoned?” energy
- Check the actual docs, not just the README headline

Ruff, for example, lives at [https://github.com/astral-sh/ruff](https://github.com/astral-sh/ruff) — not the old `charliermarsh/ruff` URL you will still find in stale posts.

### Consider Alternatives

HTTP clients, as a sample:

- `requests` — simple, popular, mature
- `httpx` — modern, async
- `urllib3` — lower level, fewer extras

Pick for the job, not for the blog post that ranked first this week.

## Updating Dependencies

Update on purpose. Security fixes and stale locks both accumulate.

```bash
# Refresh the lockfile toward the latest versions your constraints allow
uv lock --upgrade

# Install what the new lockfile says
uv sync

# One package only
uv lock --upgrade-package requests
uv sync
```

Then run the tests. An upgrade that you did not verify is just a future outage with extra steps.

The pip-shaped variants still work (`uv pip list --outdated`, `uv pip install --upgrade requests`). They are not how you update a uv project. Change the lockfile, then sync.

## Dealing with Dependency Conflicts

Sometimes two packages demand incompatible versions of a third:

```
Package A requires: numpy>=2.4.0,<2.5.0
Package B requires: numpy>=2.5.0
```

Options, in order of sanity:

1. Upgrade (or, rarely, pin) one of the packages until their requirements overlap.
2. Replace one of them.
3. Keep the projects in separate environments — which you already do.
4. File an issue upstream if a maintained package is stuck on a dead major.

uv’s resolver will refuse an impossible tree instead of installing one that imports with a warning and dies in production. Read `uv tree` when the error message is a wall of versions.

## Security Considerations

### Audit Dependencies

Check installed packages against known vulnerability records:

```bash
uv add --dev pip-audit
uv run pip-audit
```

Run this in CI, not only when you remember.

### Keep Dependencies Updated

Unpatched libraries stay unpatched until you refresh the lockfile:

```bash
uv lock --upgrade
uv sync
uv run pip-audit
```

Do this on a schedule, then run the test suite. “Latest” is not a strategy if you never look at the diff.

### Review Before Installing

- Prefer packages with real usage, a license, and recent maintenance.
- For anything that will see secrets or user data, read more than the README.
- Check the *name*. Typosquatting is a thing; `reqeusts` is not `requests`.

## Best Practices

**Let the lockfile pin production.** Ranges in `pyproject.toml`, exact versions in `uv.lock`, and that lockfile committed. That is how you get both “we can upgrade” and “CI installed what we tested.”

```bash
git add pyproject.toml uv.lock
git commit -m "Add requests and lock the tree"
```

**Keep runtime and dev dependencies apart.** pytest does not belong in the production image.

```toml
[dependency-groups]
dev = ["pytest>=9.2.0", "ruff>=0.17.0", "ty"]
```

**Leave a breadcrumb in `pyproject.toml`** when the name is not self-explanatory:

```toml
dependencies = [
    "requests>=2.34.0",  # HTTP client for API calls
    "pandas>=3.1.0",     # Data analysis and manipulation
]
```

**Maintain it.** Weekly: security-relevant bumps. Monthly: a look at `uv lock --upgrade` plus tests. Quarterly: delete unused dependencies. Unused code is a gift compared to unused dependencies; the latter still install, still break, and still get CVEs.

**Add as little as you can.** Every extra package is more install time, more conflict surface, and more code you did not review. The standard library is allowed to win.

## Common Dependency Patterns

### Web Applications

```toml
dependencies = [
    "fastapi[standard]>=0.162.0",  # API framework plus the usual server extras
    "sqlalchemy>=2.1.0",           # Database ORM
    "pydantic>=2.14.0",            # Data validation (also pulled in by FastAPI)
]
```

### Data Science

```toml
dependencies = [
    "pandas>=3.1.0",         # Data manipulation
    "numpy>=2.5.0",          # Numerical computing
    "matplotlib>=3.11.0",    # Visualization
    "scikit-learn>=1.9.0",   # Machine learning
]
```

### CLI Tools

```toml
dependencies = [
    "typer>=0.29.0",         # CLI from type hints
    "rich>=15.0.0",          # Terminal output
]
```

## Migrating from requirements.txt to pyproject.toml

Existing project, classic `requirements.txt`:

**Step 1: Create pyproject.toml** if you do not have one:

```bash
uv init --python 3.14
```

Or the minimal file:

```bash
cat > pyproject.toml << 'EOF'
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = []
EOF
```

**Step 2: Import the requirements**

```bash
uv add -r requirements.txt
# Dev-only file, if you had one:
# uv add --dev -r requirements-dev.txt
```

**Step 3: Test**

```bash
uv sync
uv run pytest
```

**Step 4: Drop `requirements.txt` once `uv.lock` is committed** (optional; keep it if another tool still consumes it):

```bash
git rm requirements.txt
```

## Quick Reference

```bash
# pip basics (you will still see these)
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt

# uv project workflow (preferred)
uv init --python 3.14 myapp
uv add requests
uv add --dev pytest ruff
uv remove requests
uv sync
uv sync --no-dev
uv lock --upgrade
uv tree
uv run pytest
uvx ruff check .

# pip-compat, when you need it
uv pip install -r requirements.txt
uv add -r requirements.txt
uv export --format requirements.txt -o requirements.txt

# pyproject.toml structure
[project]
requires-python = ">=3.13"
dependencies = ["requests>=2.34.0"]

[dependency-groups]
dev = ["pytest>=9.2.0", "ruff>=0.17.0"]
```

## Troubleshooting

### Package Not Found

Confirm the name on [PyPI](https://pypi.org). A failed resolve is often a typo, a yanked version, or `requires-python` that excludes the interpreter in `.venv`. `uv python install 3.14` and `uv venv --python 3.14` if the environment is on an older interpreter than the package supports.

### Version Conflicts

```bash
uv tree
uv lock          # the resolver explains the clash
```

`uv pip check` is there if you are in a pip-compat environment and want a quick consistency scan.

### Installation Fails

```bash
uv add -v requests
uv cache clean
```

Verbose output plus a clean cache fixes more “it failed on my machine” installs than any other two commands in this chapter.

## What's Next?

Dependencies are declared, locked, and installable on a second machine. Next is version control: Git, so the code *and* `pyproject.toml` *and* `uv.lock` have a history you can trust.

[← Previous: Virtual Environments](./02-virtual-environments.md) | [Back to README](./README.md) | [Next: Version Control →](./04-version-control.md)
