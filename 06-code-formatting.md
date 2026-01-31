# Chapter 6: Code Formatting

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing →](./07-unit-testing.md)

## What is Code Formatting?

Code formatting is the practice of structuring your code according to specific style conventions. While linting checks for errors and potential bugs, formatting focuses on how code looks - indentation, spacing, line breaks, and quote styles.

Consider these two versions of the same function:

**Unformatted:**
```python
def calculate_total(items,tax_rate=0.1,discount=0):
    subtotal=sum([item['price']for item in items])
    tax=subtotal*tax_rate
    total=subtotal+tax-discount
    return {"subtotal":subtotal,"tax":tax,"total":total}
```

**Formatted:**
```python
def calculate_total(items, tax_rate=0.1, discount=0):
    subtotal = sum([item["price"] for item in items])
    tax = subtotal * tax_rate
    total = subtotal + tax - discount
    return {"subtotal": subtotal, "tax": tax, "total": total}
```

Both work identically, but the formatted version is much easier to read.

## Why Format Your Code?

### 1. Consistency
Consistent formatting makes code predictable and easier to scan. Your brain doesn't have to adjust to different styles across files.

### 2. Reduced Cognitive Load
When code looks the same everywhere, you can focus on logic rather than syntax and style.

### 3. Eliminate Style Debates
Automated formatting ends discussions about tabs vs spaces, quote styles, and line breaks. The formatter decides.

### 4. Easier Code Reviews
Reviews focus on functionality rather than style nitpicks.

### 5. Better Diffs
Consistent formatting produces cleaner git diffs, making it easier to see what actually changed.

### 6. Professional Quality
Well-formatted code signals professionalism and attention to detail.

## Overview of Python Formatters

### Black

[Black](https://github.com/psf/black) is "the uncompromising Python code formatter." It's opinionated and offers minimal configuration.

```bash
# Install
uv pip install black

# Format files
black .

# Check without modifying
black --check .
```

**Philosophy:**
- "Any color you want, as long as it's black"
- Minimal configuration by design
- Consistent output every time

**Pros:**
- Zero-config by default
- Widely adopted
- Produces very consistent results
- Works well with most codebases

**Cons:**
- Limited customization
- Some style choices are controversial (e.g., line breaking)
- Separate tool needed for import sorting

### autopep8

Automatically formats Python code to conform to PEP 8.

```bash
# Install
uv pip install autopep8

# Format
autopep8 --in-place --aggressive file.py
```

**Pros**: Gentle, follows PEP 8 closely
**Cons**: Less comprehensive than Black, slower

### YAPF (Yet Another Python Formatter)

Google's Python formatter.

```bash
# Install
uv pip install yapf

# Format
yapf -i file.py
```

**Pros**: Highly configurable
**Cons**: Configuration complexity, less widely adopted

## Deep Dive: Formatting with Ruff

Ruff recently added a formatter that's compatible with Black but much faster. It's quickly becoming the go-to choice for Python formatting.

### Why Use Ruff for Formatting?

1. **Speed**: 10-100x faster than Black
2. **All-in-One**: One tool for linting AND formatting
3. **Black-Compatible**: Drop-in replacement for Black
4. **Simple**: Easy to configure and use
5. **Active Development**: Rapidly improving

### Installing Ruff

If you followed Chapter 4, you already have Ruff installed:

```bash
# Install if you haven't already
uv pip install ruff

# Verify
ruff --version
```

### Basic Formatting Usage

```bash
# Format all Python files in current directory
ruff format .

# Format specific files or directories
ruff format myproject/
ruff format file.py

# Check formatting without modifying (like black --check)
ruff format --check .

# Show diff of what would change
ruff format --diff .
```

### Configuration

Ruff's formatter is configured in `pyproject.toml`:

```toml
[tool.ruff]
# Set line length (default: 88, same as Black)
line-length = 88

# Assume Python 3.11
target-version = "py311"

[tool.ruff.format]
# Use double quotes for strings (default)
quote-style = "double"

# Indent with spaces (default)
indent-style = "space"

# Use Unix line endings (default is auto)
line-ending = "lf"

# Enable auto-formatting of code examples in docstrings
docstring-code-format = true
```

### Practical Example

Let's format a messy Python file:

**1. Create an unformatted file (messy.py):**
```python
import sys,os
import json

def process_data(data,multiplier=2,offset=0):
    """Process data with multiplier and offset"""
    results=[]
    for item in data:
        value=item*multiplier+offset
        results.append(value)
    return results

class DataProcessor:
    def __init__(self,name,config=None):
        self.name=name
        self.config=config if config else {}

    def process(self,items):
        return [self.transform(x) for x in items]

    def transform(self,value):
        return value**2

# Main execution
if __name__=='__main__':
    processor=DataProcessor('default')
    data=[1,2,3,4,5]
    result=processor.process(data)
    print(f"Results: {result}")
```

**2. Format with Ruff:**
```bash
ruff format messy.py
```

**3. Result:**
```python
import sys, os
import json


def process_data(data, multiplier=2, offset=0):
    """Process data with multiplier and offset"""
    results = []
    for item in data:
        value = item * multiplier + offset
        results.append(value)
    return results


class DataProcessor:
    def __init__(self, name, config=None):
        self.name = name
        self.config = config if config else {}

    def process(self, items):
        return [self.transform(x) for x in items]

    def transform(self, value):
        return value**2


# Main execution
if __name__ == "__main__":
    processor = DataProcessor("default")
    data = [1, 2, 3, 4, 5]
    result = processor.process(data)
    print(f"Results: {result}")
```

Notice the improvements:
- Proper spacing around operators
- Consistent indentation
- Proper blank lines between functions and classes
- Spaces after commas
- Consistent quote style

### Combining Linting and Formatting

Ruff can do both linting and formatting in one pass:

```bash
# Lint and format
ruff check --fix . && ruff format .

# Or create a simple script (format_and_lint.sh)
#!/bin/bash
ruff check --fix .
ruff format .
```

Add to your `pyproject.toml`:
```toml
[tool.ruff]
# Linting config
select = ["E", "F", "I", "B", "C4", "UP"]
line-length = 88

[tool.ruff.format]
# Formatting config
quote-style = "double"
```

### Integration with uv

Run Ruff formatter through uv:

```bash
# Install as development dependency
uv pip install --dev ruff

# Format using uv
uv run ruff format .

# Or use uvx to run without installing
uvx ruff format .
```

### VSCode Integration

The Ruff VSCode extension handles both linting and formatting:

**1. Install the Ruff extension**

**2. Configure in `.vscode/settings.json`:**
```json
{
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll.ruff": true,
            "source.organizeImports.ruff": true
        }
    },
    "ruff.enable": true
}
```

Now VSCode will:
- Format on save
- Fix linting issues automatically
- Organize imports

### Format on Save vs Format on Commit

You have two main options:

**Option 1: Format on Save (Recommended for solo work)**
- Immediate feedback
- Always working with formatted code
- Configure in your editor

**Option 2: Format on Commit (Recommended for teams)**
- Consistent formatting across team
- Works regardless of editor choice
- Use pre-commit hooks

### Pre-commit Hook for Formatting

**1. Install pre-commit:**
```bash
uv pip install pre-commit
```

**2. Create `.pre-commit-config.yaml`:**
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9  # Check for latest version
    hooks:
      # Run the linter
      - id: ruff
        args: [--fix]
      # Run the formatter
      - id: ruff-format
```

**3. Install the hook:**
```bash
pre-commit install
```

Now formatting happens automatically before each commit!

## Ruff vs Black: A Comparison

| Feature | Ruff | Black |
|---------|------|-------|
| Speed | 10-100x faster | Baseline |
| Black compatibility | Yes, near 100% | N/A |
| Linting included | Yes | No (separate tool needed) |
| Maturity | Newer (2022) | Mature (2018) |
| Configuration | More options | Minimal |
| Adoption | Rapidly growing | Very widespread |

For new projects, Ruff is recommended. For existing projects using Black, Ruff is a drop-in replacement.

## Best Practices

### 1. Format Early and Often
Add formatting to your project from day one. It's much easier than reformatting later.

### 2. Commit Formatting Changes Separately
When adding formatting to an existing project:
```bash
# 1. Format all code
ruff format .

# 2. Commit ONLY formatting changes
git add .
git commit -m "Apply Ruff formatting to entire codebase"

# 3. Continue with feature work
```

This keeps git history clean and makes it easy to ignore formatting commits when using `git blame`.

### 3. Use Consistent Line Length
Pick a line length and stick to it:
- 88 (Black/Ruff default) - Good balance
- 79 (PEP 8) - Traditional Python
- 100 or 120 - Common in modern projects

### 4. Document Your Formatting Choices
Add formatting configuration to your project's README:
```markdown
## Code Style

This project uses Ruff for both linting and formatting.

Format code: `ruff format .`
Lint code: `ruff check --fix .`
```

### 5. Automate Everything
Don't rely on manual formatting. Use:
- Format-on-save in your editor
- Pre-commit hooks
- CI/CD checks

### 6. Be Consistent Across Projects
If you work on multiple Python projects, use the same formatter and configuration. It reduces context switching.

## Common Formatting Scenarios

### Long Lines

**Before:**
```python
def create_user(username, email, first_name, last_name, date_of_birth, address, phone_number, is_active=True):
    pass
```

**After (Ruff format):**
```python
def create_user(
    username,
    email,
    first_name,
    last_name,
    date_of_birth,
    address,
    phone_number,
    is_active=True,
):
    pass
```

### Dictionary and List Formatting

**Before:**
```python
config = {"database": "postgresql", "host": "localhost", "port": 5432, "username": "admin", "password": "secret"}
```

**After:**
```python
config = {
    "database": "postgresql",
    "host": "localhost",
    "port": 5432,
    "username": "admin",
    "password": "secret",
}
```

### Import Sorting

Ruff also sorts imports:

**Before:**
```python
import os
from mypackage import helpers
import sys
from typing import List, Dict
import json
```

**After:**
```python
import json
import os
import sys
from typing import Dict, List

from mypackage import helpers
```

## Handling Existing Code

For large existing codebases:

```bash
# 1. See what would change
ruff format --diff . | less

# 2. Format a subset first
ruff format src/module1/

# 3. Test thoroughly
pytest

# 4. Format everything
ruff format .

# 5. Commit
git add .
git commit -m "Apply Ruff formatting"
```

## Quick Reference

```bash
# Format files
ruff format .                # Format all Python files
ruff format file.py          # Format specific file
ruff format src/             # Format directory

# Check formatting (don't modify)
ruff format --check .        # Exit 1 if formatting needed
ruff format --diff .         # Show what would change

# Combined linting and formatting
ruff check --fix . && ruff format .

# Configuration (pyproject.toml)
[tool.ruff]
line-length = 88

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

## What's Next?

With linting and formatting in place, your code is clean and consistent. Next, we'll learn how to verify that your code actually works correctly using unit testing.

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing →](./07-unit-testing.md)
