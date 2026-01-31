# Chapter 5: Linting

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)

## What is Linting?

Linting is the automated checking of your source code for programmatic and stylistic errors. A linter is a tool that analyzes your code and flags potential bugs, errors, stylistic issues, and suspicious constructs.

The term "lint" comes from the original Unix utility called `lint` that examined C source code. Today, linters exist for virtually every programming language.

## Why Lint Your Code?

### 1. Catch Bugs Early
Linters can identify many types of errors before you even run your code:
- Undefined variables
- Unused imports
- Syntax errors
- Type mismatches
- Logic errors

### 2. Enforce Code Style
Linters ensure code follows consistent style guidelines:
- Naming conventions
- Indentation and spacing
- Line length limits
- Import organization

### 3. Learn Best Practices
By following linter suggestions, you learn Python best practices and idioms over time.

### 4. Improve Code Reviews
With automated style checking, code reviews can focus on logic and architecture rather than formatting nitpicks.

### 5. Prevent Common Mistakes
Linters warn about common pitfalls:
- Mutable default arguments
- Bare except clauses
- Shadowing built-in names
- Security vulnerabilities

## Overview of Python Linters

The Python ecosystem has several popular linting tools:

### Flake8
[Flake8](https://github.com/PyCQA/flake8) is a wrapper around several tools:
- PyFlakes (logical errors)
- pycodestyle (PEP 8 style guide)
- McCabe (complexity checker)

```bash
# Install
uv pip install flake8

# Run
flake8 myproject/

# Configure in .flake8 or setup.cfg
```

**Pros**: Mature, widely used, extensive plugin ecosystem
**Cons**: Slower than modern alternatives, separate tools needed for formatting

### Pylint
[Pylint](https://github.com/PyCQA/pylint) is a comprehensive linter that checks for errors, enforces coding standards, and looks for code smells.

```bash
# Install
uv pip install pylint

# Run
pylint myproject/
```

**Pros**: Very thorough, catches many issues, customizable
**Cons**: Can be slow, sometimes overly strict, verbose output

### Pyright
[Pyright](https://github.com/microsoft/pyright) is a static type checker that also catches many linting issues.

```bash
# Install
npm install -g pyright

# Run
pyright
```

**Pros**: Fast, excellent type checking, good IDE integration
**Cons**: Requires Node.js, focused primarily on types

### mypy
[mypy](https://github.com/python/mypy) is the original static type checker for Python.

```bash
# Install
uv pip install mypy

# Run
mypy myproject/
```

**Pros**: Official type checker, mature, well-documented
**Cons**: Can be slow on large projects

### Ruff
[Ruff](https://github.com/astral-sh/ruff) is a modern, extremely fast Python linter written in Rust. It can replace Flake8, isort, and many Flake8 plugins.

```bash
# Install
uv pip install ruff

# Run
ruff check .
```

**Pros**: Extremely fast (10-100x faster than alternatives), actively developed, all-in-one tool
**Cons**: Newer tool (though rapidly maturing)

## Deep Dive: Ruff

Ruff has quickly become the modern standard for Python linting. Let's explore it in detail.

### Why Ruff?

1. **Speed**: Written in Rust, it's 10-100x faster than existing linters
2. **Comprehensive**: Implements hundreds of rules from multiple tools
3. **Simple**: One tool replaces many (Flake8, isort, pyupgrade, etc.)
4. **Modern**: Actively developed with frequent updates
5. **Compatible**: Drop-in replacement for Flake8

### Installing Ruff

```bash
# Using uv (recommended)
uv pip install ruff

# Or install globally with uv tools
uv tool install ruff

# Verify installation
ruff --version
```

### Basic Usage

```bash
# Check all Python files in current directory
ruff check .

# Check specific files or directories
ruff check myproject/
ruff check file.py

# Auto-fix issues when possible
ruff check --fix .

# Show all violations (including those that would be fixed)
ruff check --fix --show-fixes .
```

### Configuration

Ruff can be configured via `pyproject.toml`, `ruff.toml`, or `.ruff.toml`.

**Example pyproject.toml:**

```toml
[tool.ruff]
# Set the maximum line length
line-length = 88

# Assume Python 3.11
target-version = "py311"

# Enable specific rule sets
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]

# Disable specific rules
ignore = [
    "E501",  # line too long (handled by formatter)
]

# Exclude directories
exclude = [
    ".git",
    ".venv",
    "__pycache__",
    "build",
    "dist",
]

[tool.ruff.per-file-ignores]
# Ignore specific rules in test files
"tests/**/*.py" = ["S101"]  # Allow assert in tests
```

### Understanding Ruff Rules

Ruff organizes rules into categories:

- **E, W**: pycodestyle errors and warnings
- **F**: Pyflakes (logical errors)
- **I**: isort (import sorting)
- **N**: pep8-naming
- **B**: flake8-bugbear (likely bugs)
- **A**: flake8-builtins (shadowing built-ins)
- **C4**: flake8-comprehensions
- **UP**: pyupgrade (modernize Python code)
- **S**: flake8-bandit (security issues)

View all rules: [Ruff Rules Documentation](https://docs.astral.sh/ruff/rules/)

### Practical Example

Let's create a sample project and lint it with Ruff.

**1. Create a project structure:**
```bash
mkdir my_sample_project
cd my_sample_project
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
```

**2. Create a sample Python file (sample.py):**
```python
import os
import sys
import json

def calculate_sum(numbers):
    """Calculate sum of numbers"""
    total = 0
    for num in numbers:
        total = total + num
    return total

def process_data(data):
    if type(data) == list:
        return calculate_sum(data)
    else:
        return 0

# Unused import example
def main():
    nums = [1,2,3,4,5]
    result = process_data(nums)
    print(f"Sum is {result}")

if __name__ == "__main__":
    main()
```

**3. Install and run Ruff:**
```bash
uv pip install ruff

# Check the file
ruff check sample.py
```

**Expected output (issues found):**
```
sample.py:1:8: F401 [*] `os` imported but unused
sample.py:2:8: F401 [*] `sys` imported but unused
sample.py:3:8: F401 [*] `json` imported but unused
sample.py:13:8: E721 Use `isinstance()` for type comparisons
sample.py:20:15: E231 Missing whitespace after ','
```

**4. Fix automatically where possible:**
```bash
ruff check --fix sample.py
```

**5. Review remaining issues:**
After auto-fix, Ruff removes unused imports. You'll need to manually fix:
- Line 13: Change `type(data) == list` to `isinstance(data, list)`
- Line 20: Add spaces after commas in the list

### Integrating Ruff with uv

You can run Ruff through uv without installing it in your environment:

```bash
# Run ruff using uv (installs if needed)
uv tool run ruff check .

# Or add ruff as a development dependency
uv pip install --dev ruff

# Then run it
uv run ruff check .
```

### Ruff in VSCode

Install the Ruff extension for real-time linting:

1. Open VSCode
2. Go to Extensions (Ctrl+Shift+X)
3. Search for "Ruff"
4. Install the official Ruff extension

Configure in `.vscode/settings.json`:
```json
{
    "ruff.enable": true,
    "ruff.organizeImports": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.ruff": true,
        "source.organizeImports.ruff": true
    }
}
```

### Pre-commit Integration

Run Ruff automatically before each commit:

**1. Install pre-commit:**
```bash
uv pip install pre-commit
```

**2. Create .pre-commit-config.yaml:**
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9  # Use latest version
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
```

**3. Install the hook:**
```bash
pre-commit install
```

Now Ruff runs automatically on `git commit`!

## Best Practices

### 1. Start Early
Add linting to your project from the beginning. It's easier to maintain standards than to retrofit them.

### 2. Fix Issues Incrementally
For existing projects with many issues:
```bash
# See how many issues exist
ruff check . --statistics

# Fix safe issues automatically
ruff check --fix .

# Address remaining issues gradually
```

### 3. Configure for Your Team
Agree on rules with your team and document them in `pyproject.toml`.

### 4. Use in CI/CD
Run linting in your continuous integration pipeline to catch issues before merge.

### 5. Don't Fight the Linter
Most linter suggestions improve code quality. If you disable a rule, document why.

### 6. Combine with Formatting
Use Ruff for both linting and formatting (covered in the next chapter).

## Common Linting Issues and Fixes

### Unused Imports
```python
# Bad
import os
import sys

print("Hello")

# Good - remove unused imports
print("Hello")
```

### Undefined Names
```python
# Bad
def calculate():
    return total  # NameError: total not defined

# Good
def calculate():
    total = 0
    return total
```

### Mutable Default Arguments
```python
# Bad
def add_item(item, items=[]):
    items.append(item)
    return items

# Good
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Type Comparisons
```python
# Bad
if type(data) == list:
    pass

# Good
if isinstance(data, list):
    pass
```

## Quick Reference

```bash
# Install Ruff
uv pip install ruff

# Check files
ruff check .
ruff check file.py

# Auto-fix issues
ruff check --fix .

# Watch mode (re-run on file changes)
ruff check --watch .

# Show statistics
ruff check . --statistics

# Configuration
# Add to pyproject.toml:
[tool.ruff]
line-length = 88
select = ["E", "F", "I"]
```

## What's Next?

Linting helps catch errors and enforce style, but consistent formatting makes code even more readable. In the next chapter, we'll explore code formatting and how Ruff can handle both linting and formatting.

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)
