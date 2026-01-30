# Chapter 10: Putting It All Together

[← Previous: LLMs](./09-llms.md) | [Back to README](./README.md)

## Overview

You've learned about all the essential tools for professional Python development:
- Virtual environments
- Version control
- Linting and formatting
- Unit testing
- Static type checking
- CI/CD
- AI-assisted development

Now let's put it all together in a complete, real-world workflow.

## Creating a New Project with uv

Let's start a new Python project using modern best practices.

### 1. Create Project Directory

```bash
# Create and navigate to project directory
mkdir myapp
cd myapp
```

### 2. Initialize Git

```bash
# Initialize git repository
git init

# Create .gitignore
cat > .gitignore << 'EOF'
# Virtual environments
.venv/
venv/
env/

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# Testing
.pytest_cache/
.coverage
htmlcov/
*.cover

# Type checking
.mypy_cache/
.pytype/

# IDEs
.vscode/
.idea/
*.swp

# Distribution
dist/
build/
*.egg-info/

# Environment
.env
.env.local
EOF
```

### 3. Create Virtual Environment with uv

```bash
# Install uv if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment
uv venv

# Activate it
source .venv/bin/activate  # macOS/Linux
# or
.venv\Scripts\activate  # Windows
```

### 4. Create Project Structure

```bash
# Create directory structure
mkdir -p src/myapp tests docs

# Create __init__ files
touch src/myapp/__init__.py
touch tests/__init__.py

# Create initial files
touch src/myapp/main.py
touch tests/test_main.py
touch README.md
```

Your structure should look like:
```
myapp/
├── .git/
├── .gitignore
├── .venv/
├── src/
│   └── myapp/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
├── docs/
├── pyproject.toml
└── README.md
```

### 5. Create pyproject.toml

```bash
cat > pyproject.toml << 'EOF'
[project]
name = "myapp"
version = "0.1.0"
description = "A sample Python application"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
readme = "README.md"
requires-python = ">=3.9"
dependencies = [
    # Add your dependencies here
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "ruff>=0.1.0",
    "mypy>=1.5.0",
    "pre-commit>=3.5.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 88
target-version = "py311"
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]
exclude = [
    ".git",
    ".venv",
    "__pycache__",
    "build",
    "dist",
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--tb=short",
]

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
plugins = []

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
]
EOF
```

### 6. Install Development Dependencies

```bash
# Install the project in editable mode with dev dependencies
uv pip install -e ".[dev]"
```

## Using Cookiecutter Templates

[Cookiecutter](https://github.com/cookiecutter/cookiecutter) is a tool that creates projects from templates. There are many Python project templates available.

### Installing Cookiecutter

```bash
uv pip install cookiecutter
```

### Using a Python Library Template

Let's use a professional Python library template:

```bash
# Use a cookiecutter template
cookiecutter https://github.com/amitsk/cookiecutter-python-library

# You'll be prompted for project details:
# - project_name: My Amazing Library
# - project_slug: my-amazing-library
# - author_name: Your Name
# - email: your.email@example.com
# - etc.
```

This creates a complete project structure with:
- Proper directory layout
- pyproject.toml configuration
- Testing setup
- Documentation structure
- CI/CD configuration
- Pre-commit hooks
- Example code

### Exploring the Generated Project

```bash
cd my-amazing-library
ls -la
```

You'll see:
```
my-amazing-library/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .gitignore
├── .pre-commit-config.yaml
├── src/
│   └── my_amazing_library/
│       ├── __init__.py
│       └── main.py
├── tests/
│   ├── __init__.py
│   └── test_main.py
├── docs/
├── pyproject.toml
├── README.md
└── LICENSE
```

## Complete Project Workflow

Now let's walk through a complete workflow using all the tools.

### Step 1: Write Initial Code

**src/myapp/calculator.py:**
```python
"""A simple calculator module."""


def add(a: int, b: int) -> int:
    """Add two numbers.

    Args:
        a: First number
        b: Second number

    Returns:
        Sum of a and b
    """
    return a + b


def subtract(a: int, b: int) -> int:
    """Subtract b from a."""
    return a - b


def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


def divide(a: float, b: float) -> float:
    """Divide a by b.

    Args:
        a: Numerator
        b: Denominator

    Returns:
        Result of division

    Raises:
        ValueError: If b is zero
    """
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

### Step 2: Write Tests

**tests/test_calculator.py:**
```python
"""Tests for calculator module."""
import pytest
from myapp.calculator import add, subtract, multiply, divide


def test_add():
    """Test addition."""
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0


def test_subtract():
    """Test subtraction."""
    assert subtract(5, 3) == 2
    assert subtract(0, 5) == -5


def test_multiply():
    """Test multiplication."""
    assert multiply(3, 4) == 12
    assert multiply(0, 100) == 0


def test_divide():
    """Test division."""
    assert divide(10, 2) == 5
    assert divide(9, 3) == 3


def test_divide_by_zero():
    """Test division by zero raises error."""
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)


@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_add_parametrized(a, b, expected):
    """Test addition with multiple inputs."""
    assert add(a, b) == expected
```

### Step 3: Run Quality Checks

```bash
# Format code
ruff format .

# Lint code
ruff check --fix .

# Type check
mypy src/

# Run tests
pytest

# Run tests with coverage
pytest --cov=src --cov-report=term-missing
```

### Step 4: Set Up Pre-commit Hooks

Create **.pre-commit-config.yaml:**
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: []
        args: [--ignore-missing-imports]

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
```

Install the hooks:
```bash
pre-commit install
```

Now quality checks run automatically on commit!

### Step 5: Set Up GitHub Repository

```bash
# Create initial commit
git add .
git commit -m "Initial project setup

- Add calculator module with basic operations
- Add comprehensive tests with pytest
- Configure ruff for linting and formatting
- Set up mypy for type checking
- Add pre-commit hooks

🤖 Generated with Claude Code
"

# Create GitHub repository (using gh CLI)
gh repo create myapp --public --source=. --remote=origin

# Push code
git push -u origin main
```

### Step 6: Set Up CI/CD

Create **.github/workflows/ci.yml:**
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install uv
        run: pip install uv

      - name: Install dependencies
        run: |
          uv venv
          uv pip install -e ".[dev]"

      - name: Lint with Ruff
        run: |
          source .venv/bin/activate
          ruff check .

      - name: Check formatting
        run: |
          source .venv/bin/activate
          ruff format --check .

      - name: Type check with mypy
        run: |
          source .venv/bin/activate
          mypy src/

  test:
    name: Test Python ${{ matrix.python-version }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ['3.9', '3.10', '3.11', '3.12']

    steps:
      - uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          pip install uv
          uv pip install -e ".[dev]"

      - name: Run tests
        run: pytest --cov=src --cov-report=xml

      - name: Upload coverage
        if: matrix.os == 'ubuntu-latest' && matrix.python-version == '3.11'
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
```

Commit and push:
```bash
git add .github/workflows/ci.yml
git commit -m "Add CI/CD workflow"
git push
```

### Step 7: Working on a Feature

```bash
# Create a feature branch
git checkout -b add-power-function

# Write the code
cat >> src/myapp/calculator.py << 'EOF'


def power(base: float, exponent: float) -> float:
    """Raise base to the power of exponent.

    Args:
        base: Base number
        exponent: Exponent

    Returns:
        base raised to exponent
    """
    return base ** exponent
EOF

# Write tests
cat >> tests/test_calculator.py << 'EOF'


def test_power():
    """Test power function."""
    assert power(2, 3) == 8
    assert power(5, 2) == 25
    assert power(10, 0) == 1
    assert power(2, -1) == 0.5
EOF

# Run quality checks
ruff format .
ruff check --fix .
mypy src/
pytest

# Commit (pre-commit hooks run automatically)
git add .
git commit -m "Add power function to calculator

- Implement power function with type hints
- Add comprehensive tests
- Update documentation
"

# Push and create PR
git push -u origin add-power-function
gh pr create --title "Add power function" --body "Adds exponentiation support to calculator"
```

### Step 8: Code Review and Merge

1. CI runs automatically on the PR
2. Review the code on GitHub
3. Merge when CI passes and review is approved
4. Delete the feature branch

```bash
# After merge, update local main
git checkout main
git pull origin main
git branch -d add-power-function
```

## Daily Development Workflow

Here's your typical daily workflow:

### Morning Routine

```bash
# Update your local repository
git checkout main
git pull origin main

# Start working on a task
git checkout -b feature-name

# Set up environment
source .venv/bin/activate
```

### Development Loop

```bash
# 1. Write code
# 2. Run tests frequently
pytest

# 3. Check code quality
ruff check --fix .
ruff format .
mypy src/

# 4. Commit frequently
git add .
git commit -m "Descriptive message"

# 5. Push when ready
git push -u origin feature-name
```

### Before Submitting PR

```bash
# Final checks
ruff check .
ruff format --check .
mypy src/
pytest --cov=src

# Ensure all tests pass
# Create PR
gh pr create
```

## Project Maintenance

### Adding a Dependency

```bash
# Add to pyproject.toml dependencies
# Then install
uv pip install requests

# Or install directly (uv will update pyproject.toml)
uv add requests
```

### Updating Dependencies

```bash
# Update all dependencies
uv pip install --upgrade -e ".[dev]"

# Or use uv's update command
uv pip list --outdated
```

### Running Different Checks

```bash
# Format only
ruff format .

# Lint only
ruff check .

# Fix linting issues
ruff check --fix .

# Type check
mypy src/

# Test with coverage
pytest --cov=src --cov-report=html
open htmlcov/index.html  # View coverage report

# All checks
ruff check --fix . && ruff format . && mypy src/ && pytest
```

## Troubleshooting Common Issues

### Pre-commit Hooks Failing

```bash
# Run manually to see errors
pre-commit run --all-files

# Update hooks
pre-commit autoupdate

# Skip hooks if needed (not recommended)
git commit --no-verify
```

### CI Failing Locally Passing

```bash
# Run in clean environment
deactivate
rm -rf .venv
uv venv
uv pip install -e ".[dev]"
pytest
```

### Merge Conflicts

```bash
# Update your branch with main
git checkout main
git pull origin main
git checkout your-branch
git merge main

# Resolve conflicts manually
# Then commit
git add .
git commit -m "Merge main and resolve conflicts"
```

## Best Practices Summary

### ✅ Do

- Use virtual environments for every project
- Commit frequently with descriptive messages
- Write tests for new code
- Run quality checks before pushing
- Use feature branches
- Review code before merging
- Keep dependencies updated
- Document complex code

### ❌ Don't

- Commit directly to main
- Push without running tests
- Ignore linter warnings
- Skip code review
- Commit secrets or sensitive data
- Leave commented-out code
- Merge broken CI
- Use `--no-verify` habitually

## Quick Reference Commands

```bash
# Project setup
uv venv
uv pip install -e ".[dev]"

# Quality checks
ruff check --fix .
ruff format .
mypy src/
pytest --cov=src

# Git workflow
git checkout -b feature-name
git add .
git commit -m "message"
git push -u origin feature-name
gh pr create

# Maintenance
uv pip install --upgrade -e ".[dev]"
pre-commit autoupdate
pytest --cov=src --cov-report=html
```

## Next Steps

Congratulations! You now have a complete, professional Python development setup. Here's what you can do next:

### 1. Practice

Create a small project using everything you've learned:
- Set up the project structure
- Write some code
- Add tests
- Set up CI/CD
- Make a few commits
- Create and merge a PR

### 2. Customize

Adapt these tools to your needs:
- Adjust ruff rules
- Configure mypy strictness
- Customize pre-commit hooks
- Add project-specific tools

### 3. Learn More

Explore advanced topics:
- Advanced pytest features (fixtures, markers, plugins)
- Type system advanced features (generics, protocols)
- GitHub Actions advanced workflows
- Performance profiling and optimization
- Security best practices

### 4. Share

Help others learn:
- Share your setup with teammates
- Contribute to open source
- Write about your experience
- Mentor newcomers

## Resources

### Official Documentation

- [Python](https://docs.python.org/)
- [uv](https://docs.astral.sh/uv/)
- [Ruff](https://docs.astral.sh/ruff/)
- [pytest](https://docs.pytest.org/)
- [mypy](https://mypy.readthedocs.io/)
- [GitHub Actions](https://docs.github.com/actions)

### Community

- [Python Discord](https://discord.gg/python)
- [r/Python](https://reddit.com/r/Python)
- [Python User Groups](https://wiki.python.org/moin/LocalUserGroups)

### Templates

- [cookiecutter-python-library](https://github.com/amitsk/cookiecutter-python-library)
- [PyPA sample project](https://github.com/pypa/sampleproject)

## Conclusion

You've learned the complete toolkit for modern Python development:

1. **Virtual Environments**: Isolate project dependencies
2. **Version Control**: Track changes and collaborate
3. **Linting**: Catch errors automatically
4. **Formatting**: Maintain consistent style
5. **Testing**: Verify code correctness
6. **Type Checking**: Catch type errors early
7. **CI/CD**: Automate quality checks
8. **AI Tools**: Accelerate development
9. **Complete Workflow**: Put it all together

These tools and practices will serve you throughout your Python development journey. Start simple, add tools gradually, and soon this workflow will become second nature.

Happy coding!

[← Previous: LLMs](./09-llms.md) | [Back to README](./README.md)
