# Chapter 6: Code Formatting

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing →](./07-unit-testing.md)

## What is Code Formatting?

Formatting is how the code looks: indentation, spacing, line breaks, quotes. Linting asks "is this wrong?" Formatting asks "is this consistent?"

Same function, two presentations:

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

Both run. One is a kindness to whoever reads it next, which is often you.

## Why Format?

Consistent code is easier to scan. Your eyes stop compensating for each author's spacing hobby.

A formatter also ends the tabs-versus-spaces discussion. You can still have the discussion. It just no longer belongs in the pull request.

Reviews stay on behavior. Diffs stay on the change you meant to make, not on a drive-by reindent. None of this is glamorous. It is how a repo stays readable after the third contributor.

## Other Formatters

Ruff is the formatter in this guide. You will still see these names.

**[Black](https://github.com/psf/black)** is the historical "uncompromising" formatter. Almost no knobs, identical output every time, still widely used. Ruff's formatter is designed to be compatible with Black's style, which is why switching is usually a config change rather than a rewrite.

```bash
uvx black --check .
```

**[autopep8](https://github.com/hhatto/autopep8)** nudges code toward PEP 8. Gentler, less complete.

**[YAPF](https://github.com/google/yapf)** (Yet Another Python Formatter) is Google's highly configurable option. Powerful if you want a style committee. Most teams do not.

## Formatting with Ruff

Ruff's formatter is the same `ruff` binary you installed for linting. It is mature, Black-compatible, and fast. Use it as the project formatter unless you have a specific reason not to.

If you followed Chapter 5, you already have it:

```bash
uv add --dev "ruff>=0.16.0"
uv run ruff --version
```

Or without a project install:

```bash
uvx ruff format .
```

### Usage

```bash
# Whole tree
uv run ruff format .

# One path
uv run ruff format src/
uv run ruff format messy.py

# CI: fail if anything would change
uv run ruff format --check .

# Preview the rewrite
uv run ruff format --diff .
```

`--check` is the formatting equivalent of `ruff check`: it reports, it does not write.

### Configuration

```toml
[project]
requires-python = ">=3.12"

[tool.ruff]
line-length = 88
target-version = "py313"

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "lf"
docstring-code-format = true
```

88 characters is Black's default and Ruff's. PEP 8 said 79; plenty of projects use 100 or 120. Pick one number, put it in `pyproject.toml`, and stop talking about it.

### Practical example

**1. A file that has never met a formatter (`messy.py`):**

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

**2. Format it:**

```bash
uv run ruff format messy.py
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

Spacing, blank lines, and quotes are now consistent. Notice `import sys, os` is still one line: the **formatter** does not split imports. That is a lint fix (`E401` / `I001`):

```bash
uv run ruff check --fix messy.py
uv run ruff format messy.py
```

Lint, then format. That order is the whole workflow.

### One tool, two commands

```bash
uv run ruff check --fix . && uv run ruff format .
```

A config that covers both:

```toml
[tool.ruff]
line-length = 88
target-version = "py313"

[tool.ruff.lint]
select = ["E", "W", "F", "I", "B", "C4", "UP"]

[tool.ruff.format]
quote-style = "double"
```

### VS Code

Official Ruff extension from Astral, then `.vscode/settings.json`:

```json
{
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
        }
    }
}
```

Save formats, fixes, and sorts imports. Solo work is nicer when the file is already clean before it hits git.

### Format on save vs format on commit

**On save** (good for you): immediate feedback, you never stare at an unformatted buffer.

**On commit** (good for a team): everyone gets the same result, even from an editor that is not VS Code. Use pre-commit.

Do both. On-save keeps you honest; the hook keeps the repo honest.

### Pre-commit

```bash
uv add --dev pre-commit
```

`.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.16.5
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
      - id: ruff-format
```

```bash
uv run pre-commit install
```

`--exit-non-zero-on-fix` means "I cleaned this up; please look at it and commit again." That is the correct failure.

## Ruff vs Black

| Feature | Ruff | Black |
|---------|------|-------|
| Speed | Much faster | Baseline |
| Black-compatible style | Yes | N/A |
| Linting in the same tool | Yes | No |
| Maturity | Production default (0.16.x) | Long-standing (2018) |
| Knobs | A few (quotes, indent, line length) | Almost none |
| Adoption | Default on new projects | Still everywhere |

New project: Ruff. Existing Black project: Ruff is the usual replacement. Run `ruff format --check` after switching; remaining diffs are rare and small.

## Habits That Stick

Turn formatting on before the repo has a personality. Reformatting 40,000 lines later is a commit nobody wants to review.

When you *do* reformat an existing tree, make that the only change in the commit:

```bash
uv run ruff format .
git add .
git commit -m "Apply Ruff formatting to entire codebase"
```

`git blame` can skip that commit. Future diffs stay readable.

Write the two commands in the README so nobody has to guess:

```markdown
## Code Style

This project uses Ruff for linting and formatting.

- Format: `uv run ruff format .`
- Lint: `uv run ruff check --fix .`
```

Editor on-save, pre-commit, and CI `--check` are the same rule enforced three times. That is the point.

## Common Rewrites

### Long signatures

**Before:**
```python
def create_user(username, email, first_name, last_name, date_of_birth, address, phone_number, is_active=True):
    pass
```

**After:**
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

### Long literals

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

### Import order (linter, not formatter)

`ruff format` will not sort this. `ruff check --fix` with rule `I` will:

**Before:**
```python
import os
from mypackage import helpers
import sys
import json
```

**After:**
```python
import json
import os
import sys

from mypackage import helpers
```

stdlib, third-party, first-party, each block sorted. You do not maintain this by hand.

## Existing Codebases

```bash
# See the blast radius
uv run ruff format --diff . | less

# Format one package, run tests
uv run ruff format src/module1/
uv run pytest

# Then the rest, in its own commit
uv run ruff format .
git add .
git commit -m "Apply Ruff formatting"
```

Do not mix a reformat with a feature. Reviewers deserve that much.

## Quick Reference

```bash
uv add --dev "ruff>=0.16.0"

uv run ruff format .                 # write
uv run ruff format file.py
uv run ruff format --check .         # CI
uv run ruff format --diff .

uv run ruff check --fix . && uv run ruff format .

uvx ruff format .

# [tool.ruff]
# line-length = 88
# target-version = "py313"
# [tool.ruff.format]
# quote-style = "double"
# indent-style = "space"
```

## What's Next?

The code is consistent. Next you prove it does what you think it does: unit tests with pytest.

[← Previous: Linting](./05-linting.md) | [Back to README](./README.md) | [Next: Unit Testing →](./07-unit-testing.md)
