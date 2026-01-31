# Chapter 3: Managing Dependencies

[← Previous: Virtual Environments](./02-virtual-environments.md) | [Back to README](./README.md) | [Next: Version Control →](./04-version-control.md)

## What are Dependencies?

Dependencies are external packages or libraries that your Python project needs to function. Instead of writing everything from scratch, you can use code that others have written, tested, and shared with the community.

Think of dependencies like building blocks. When you need to:
- Make HTTP requests → use `requests`
- Work with dates → use `python-dateutil`
- Build web APIs → use `fastapi` or `flask`
- Analyze data → use `pandas` and `numpy`

You don't have to write these from scratch - you install them as dependencies.

## Understanding Dependencies with a Simple Example

Let's say you want to fetch data from a web API. You could write all the HTTP handling code yourself, or you could use the `requests` library:

**Without dependencies (doing it yourself):**
```python
import urllib.request
import json

# Complex, error-prone code
response = urllib.request.urlopen('https://api.example.com/data')
data = json.loads(response.read().decode('utf-8'))
```

**With the requests library:**
```python
import requests

# Simple, clean code
response = requests.get('https://api.example.com/data')
data = response.json()
```

The `requests` library is a dependency - external code that makes your life easier.

## Why Manage Dependencies?

### 1. Reproducibility
You need to ensure that your code works the same way on different machines. If you use `requests` version 2.31.0, everyone else should use the same version.

### 2. Collaboration
Team members need to know exactly which packages your project requires and which versions to install.

### 3. Deployment
When deploying your application, you need a reliable way to install all required packages.

### 4. Version Control
Different projects may need different versions of the same library. Dependency management helps avoid conflicts.

## The Python Package Index (PyPI)

[PyPI](https://pypi.org) is the official repository of Python packages. It contains hundreds of thousands of packages that you can install and use in your projects.

When you install a package, you're typically downloading it from PyPI.

## Introduction to pip

`pip` is the standard package installer for Python. It comes bundled with Python installations.

### Basic pip Usage

```bash
# Install a package
pip install requests

# Install a specific version
pip install requests==2.31.0

# Install the latest version
pip install --upgrade requests

# Uninstall a package
pip uninstall requests

# List installed packages
pip list

# Show package information
pip show requests
```

### Requirements Files

The traditional way to manage dependencies is with a `requirements.txt` file:

**requirements.txt:**
```
requests==2.31.0
pandas==2.1.0
numpy==1.24.3
```

Install all requirements:
```bash
pip install -r requirements.txt
```

### Limitations of pip

While pip works well, it has some limitations:

1. **Slow**: Installing large dependency trees can be time-consuming
2. **No Lock File**: requirements.txt doesn't capture transitive dependencies
3. **Dependency Resolution**: Can sometimes install incompatible versions
4. **No Built-in Virtual Environment Management**: You need to manage venvs separately

This is where modern tools like `uv` come in.

## Managing Dependencies with uv

As you learned in Chapter 2, `uv` is a fast, modern Python package manager. It excels at dependency management.

### Why Use uv for Dependencies?

1. **Speed**: 10-100x faster than pip
2. **Reliable**: Better dependency resolution
3. **Modern**: Built-in lock file support
4. **Integrated**: Combines package management with virtual environments
5. **Compatible**: Works with existing pip/requirements.txt workflows

### Installing Packages with uv

Basic installation:
```bash
# Install a single package
uv pip install requests

# Install multiple packages
uv pip install requests pandas numpy

# Install specific version
uv pip install requests==2.31.0

# Install with version constraints
uv pip install "requests>=2.30.0,<3.0.0"
```

### Working with requirements.txt

uv is fully compatible with requirements.txt:

```bash
# Install from requirements.txt
uv pip install -r requirements.txt

# Generate requirements.txt from installed packages
uv pip freeze > requirements.txt
```

### Using pyproject.toml (Modern Approach)

The modern way to specify dependencies is using `pyproject.toml`:

**pyproject.toml:**
```toml
[project]
name = "myapp"
version = "0.1.0"
dependencies = [
    "requests>=2.31.0",
    "pandas>=2.1.0",
    "numpy>=1.24.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
]
```

Install project dependencies:
```bash
# Install main dependencies
uv pip install -e .

# Install with dev dependencies
uv pip install -e ".[dev]"
```

### Adding Dependencies with uv

uv provides commands to add dependencies directly:

```bash
# Add a dependency (updates pyproject.toml)
uv add requests

# Add a dev dependency
uv add --dev pytest

# Add with version constraint
uv add "requests>=2.31.0"
```

### Removing Dependencies

```bash
# Remove a package
uv remove requests

# Remove from specific group
uv remove --dev pytest
```

### Lock Files

uv automatically creates and uses lock files (`uv.lock`) that capture the exact versions of all dependencies, including transitive ones:

```bash
# Install from lock file (exact versions)
uv sync

# Update dependencies and lock file
uv lock --upgrade
```

Lock files ensure everyone on your team uses the exact same dependency versions.

## Practical Example: Building a Web Scraper

Let's build a simple web scraper to demonstrate dependency management.

### Step 1: Set Up Project

```bash
# Create project directory
mkdir web-scraper
cd web-scraper

# Create virtual environment
uv venv
source .venv/bin/activate
```

### Step 2: Create pyproject.toml

```bash
cat > pyproject.toml << 'EOF'
[project]
name = "web-scraper"
version = "0.1.0"
description = "A simple web scraper"
dependencies = [
    "requests>=2.31.0",
    "beautifulsoup4>=4.12.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
]
EOF
```

### Step 3: Install Dependencies

```bash
# Install main dependencies
uv pip install -e .

# Check what was installed
uv pip list
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

    soup = BeautifulSoup(response.content, 'html.parser')
    title = soup.find('title')

    return title.string if title else "No title found"


if __name__ == "__main__":
    url = "https://www.python.org"
    title = fetch_page_title(url)
    print(f"Page title: {title}")
```

### Step 5: Test It

```bash
# Run the scraper
uv run python scraper.py
```

Output:
```
Page title: Welcome to Python.org
```

Notice how `requests` and `beautifulsoup4` make this task simple. These are our dependencies doing the heavy lifting.

## Dependency Version Constraints

Understanding version constraints is important:

```toml
[project]
dependencies = [
    "requests==2.31.0",        # Exact version
    "pandas>=2.1.0",           # Minimum version
    "numpy>=1.24.0,<2.0.0",    # Version range
    "flask~=2.3.0",            # Compatible release (~= 2.3.0 means >=2.3.0, <2.4.0)
]
```

### Semantic Versioning (SemVer)

Most Python packages follow semantic versioning: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes

Example: `requests 2.31.0`
- Major version: 2
- Minor version: 31
- Patch version: 0

### Best Practices for Version Constraints

```toml
# ✅ Good - allows updates but prevents breaking changes
"requests>=2.31.0,<3.0.0"

# ✅ Good - for stable libraries
"numpy>=1.24.0"

# ⚠️ Use carefully - pins exact version
"pandas==2.1.0"

# ❌ Avoid - too permissive
"requests"
```

## Development vs. Production Dependencies

Separate dependencies into groups:

**Development dependencies:**
- Testing tools (pytest)
- Linters (ruff)
- Type checkers (mypy)
- Documentation tools

**Production dependencies:**
- Libraries your application needs to run
- Web frameworks
- Database drivers
- API clients

**Example pyproject.toml:**
```toml
[project]
dependencies = [
    # Production dependencies only
    "fastapi>=0.104.0",
    "sqlalchemy>=2.0.0",
    "pydantic>=2.4.0",
]

[project.optional-dependencies]
dev = [
    # Development dependencies
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
]
```

Install for development:
```bash
uv pip install -e ".[dev]"
```

Install for production (no dev deps):
```bash
uv pip install .
```

## Exploring Packages

Before adding a dependency, research it:

### Check on PyPI

Visit [pypi.org/project/requests](https://pypi.org/project/requests) to see:
- Documentation
- Version history
- Download statistics
- License
- Dependencies

### Check the Repository

Most packages link to their source code (often on GitHub):
- Read the README
- Check recent activity
- Look at open issues
- Review documentation

### Consider Alternatives

For example, for HTTP requests:
- `requests` - Simple, popular, mature
- `httpx` - Modern, async support
- `urllib3` - Lower level, fewer dependencies

Choose based on your needs.

## Updating Dependencies

Keep dependencies updated for security and features:

```bash
# See outdated packages
uv pip list --outdated

# Update a specific package
uv pip install --upgrade requests

# Update all packages (careful!)
uv pip install --upgrade -r requirements.txt
```

### Using uv sync for Updates

```bash
# Update lock file with latest versions
uv lock --upgrade

# Install updated versions
uv sync
```

## Dealing with Dependency Conflicts

Sometimes packages require incompatible versions of the same dependency:

```
Package A requires: numpy>=1.24.0,<2.0.0
Package B requires: numpy>=2.0.0
```

Solutions:

1. **Update packages**: Check if newer versions are compatible
2. **Use different packages**: Find alternatives
3. **Isolate projects**: Use separate virtual environments
4. **Contact maintainers**: Report the issue

uv's advanced dependency resolver helps avoid many conflicts.

## Security Considerations

### Audit Dependencies

Check for known vulnerabilities:

```bash
# Using pip-audit
uv pip install pip-audit
uv run pip-audit

# Using safety
uv pip install safety
uv run safety check
```

### Keep Dependencies Updated

Outdated dependencies may have security vulnerabilities:

```bash
# Regular updates
uv pip install --upgrade -r requirements.txt

# Check for updates
uv pip list --outdated
```

### Review Before Installing

- Check package popularity and maintenance
- Read the code if possible (for critical dependencies)
- Verify the package name (typosquatting attacks exist)

## Best Practices

### 1. Pin Versions in Production

```toml
# Development - flexible
"requests>=2.31.0"

# Production - pinned
"requests==2.31.0"
```

### 2. Use Lock Files

Always commit `uv.lock` to version control:
```bash
git add uv.lock
git commit -m "Update dependencies"
```

### 3. Separate Development Dependencies

```toml
[project.optional-dependencies]
dev = ["pytest", "ruff", "mypy"]
```

### 4. Document Dependencies

Add comments in pyproject.toml:
```toml
dependencies = [
    "requests>=2.31.0",  # HTTP client for API calls
    "pandas>=2.1.0",     # Data analysis and manipulation
]
```

### 5. Regular Maintenance

Set a schedule:
- Weekly: Check for security updates
- Monthly: Review and update dependencies
- Quarterly: Audit and remove unused dependencies

### 6. Minimal Dependencies

Only add what you need:
- Fewer dependencies = less maintenance
- Smaller attack surface
- Faster installation
- Fewer conflicts

## Common Dependency Patterns

### Web Applications

```toml
dependencies = [
    "fastapi>=0.104.0",      # Web framework
    "uvicorn>=0.24.0",       # ASGI server
    "sqlalchemy>=2.0.0",     # Database ORM
    "pydantic>=2.4.0",       # Data validation
]
```

### Data Science

```toml
dependencies = [
    "pandas>=2.1.0",         # Data manipulation
    "numpy>=1.24.0",         # Numerical computing
    "matplotlib>=3.8.0",     # Visualization
    "scikit-learn>=1.3.0",   # Machine learning
]
```

### CLI Tools

```toml
dependencies = [
    "click>=8.1.0",          # CLI framework
    "rich>=13.6.0",          # Rich terminal output
    "typer>=0.9.0",          # Modern CLI framework
]
```

## Migrating from requirements.txt to pyproject.toml

If you have an existing project with requirements.txt:

**Step 1: Create pyproject.toml**
```bash
cat > pyproject.toml << 'EOF'
[project]
name = "myproject"
version = "0.1.0"
dependencies = []
EOF
```

**Step 2: Add dependencies from requirements.txt**
```bash
# Read requirements.txt and add each package
while read package; do
    uv add "$package"
done < requirements.txt
```

**Step 3: Test**
```bash
# Create fresh environment
uv venv --force
uv pip install -e .

# Run tests
pytest
```

**Step 4: Remove requirements.txt** (optional)
```bash
git rm requirements.txt
```

## Quick Reference

```bash
# pip basics
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt

# uv dependency management
uv add requests                    # Add dependency
uv add --dev pytest               # Add dev dependency
uv remove requests                # Remove dependency
uv pip install -e .               # Install project
uv pip install -e ".[dev]"        # Install with dev deps
uv pip list                       # List packages
uv pip list --outdated            # Show outdated packages
uv sync                           # Install from lock file
uv lock --upgrade                 # Update lock file

# pyproject.toml structure
[project]
dependencies = ["requests>=2.31.0"]

[project.optional-dependencies]
dev = ["pytest>=7.4.0"]
```

## Troubleshooting

### Package Not Found

```bash
# Update package index
uv pip install --upgrade pip

# Check package name on PyPI
# Visit https://pypi.org
```

### Version Conflicts

```bash
# See dependency tree
uv pip show requests

# Check conflicting requirements
uv pip check
```

### Installation Fails

```bash
# Try with verbose output
uv pip install -v requests

# Clear cache
uv cache clean
```

## What's Next?

Now that you understand how to manage dependencies, let's explore version control with Git - essential for tracking both your code and your dependency specifications.

[← Previous: Virtual Environments](./02-virtual-environments.md) | [Back to README](./README.md) | [Next: Version Control →](./04-version-control.md)
