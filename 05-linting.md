# Chapter 5: Linting

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)

## What is Linting?

A linter reads your source and flags problems without running the program: unused names, likely bugs, style drift, and the odd construct that *looks* fine until it isn't.

The name comes from the old Unix `lint` tool for C. The idea survived. Most of the original C code did not.

## Why Bother?

Linters catch a useful class of mistakes while the file is still open: undefined names, unused imports, `type(x) == list` instead of `isinstance`, mutable default arguments, bare `except:`. That is cheaper than finding the same thing in a review, or in production.

They also keep a codebase looking like one person wrote it. Naming, import order, and the "please don't do this" list stop being a matter of taste. Reviews can talk about behavior instead of commas.

Follow the suggestions for a while and you pick up Python idioms without reading PEP 8 like a novel.

## Other Linters

Ruff is the default in this guide. These still exist, and you will meet them in older repos.

**[Flake8](https://github.com/PyCQA/flake8)** wraps Pyflakes, pycodestyle, and McCabe. Mature, plugin-heavy, slower than Ruff. You assemble the rest of the toolchain yourself.

**[Pylint](https://github.com/pylint-dev/pylint)** is thorough and configurable, sometimes to a fault. It will have an opinion about everything. Useful when you want that; noisy when you don't.

Type checkers such as [mypy](https://github.com/python/mypy) and [Pyright](https://github.com/microsoft/pyright) overlap with linting on some issues. They are covered in [Chapter 8](./08-static-type-checking.md).

## Ruff

[Ruff](https://github.com/astral-sh/ruff) is a Python linter *and* formatter, written in Rust. It is the modern default: fast, one binary, hundreds of rules ported from Flake8, isort, pyupgrade, flake8-bugbear, and friends.

It is not new. The current release line is **0.16.x**. Pin that line so CI does not surprise you:

```bash
uv add --dev "ruff>=0.16.0"
uv run ruff check .
```

One-off, no project install:

```bash
uvx ruff check .
```

Ruff is fast enough that a long wait usually means you pointed it at the wrong directory.

### Why this tool

One process replaces a pile of Flake8 plugins. `--fix` applies the safe edits. The formatter (`ruff format`, next chapter) is Black-compatible, so you are not maintaining two style systems. Rule codes stay familiar if you already know Flake8.

### Basic usage

```bash
# Whole project
uv run ruff check .

# One path
uv run ruff check src/
uv run ruff check sample.py

# Apply safe fixes
uv run ruff check --fix .

# Show the fix, then apply it
uv run ruff check --fix --show-fixes .

# Counts by rule (handy on a messy repo)
uv run ruff check . --statistics

# Re-run on save
uv run ruff check --watch .
```

### Configuration

Put settings in `pyproject.toml`, `ruff.toml`, or `.ruff.toml`.

Ruff 0.16 expanded the **default** rule set a lot. A new project can start from those defaults and add rules as you go. An explicit `select` is still a good teaching (and team) config: you can see exactly what is on, and it includes the full pycodestyle `E` set, import sorting, bugbear, comprehensions, and pyupgrade.

```toml
[project]
name = "my-sample-project"
version = "0.1.0"
requires-python = ">=3.12"

[tool.ruff]
line-length = 88
target-version = "py313"

exclude = [
    ".git",
    ".venv",
    "__pycache__",
    "build",
    "dist",
]

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # Pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]

# Line length is the formatter's job.
ignore = [
    "E501",
]

[tool.ruff.lint.per-file-ignores]
# If you later enable flake8-bandit (S), keep assert in tests.
"tests/**/*.py" = ["S101"]
```

`target-version = "py313"` tells pyupgrade and friends which language features are fair game. Keep it in sync with the Python you actually run.

### Rule categories

Ruff groups rules by origin:

- **E, W**: pycodestyle
- **F**: Pyflakes (unused imports, undefined names)
- **I**: import sorting
- **N**: pep8-naming
- **B**: flake8-bugbear (the "this is probably a bug" set)
- **A**: flake8-builtins (shadowing `list`, `id`, …)
- **C4**: comprehensions
- **UP**: pyupgrade (modern syntax)
- **S**: flake8-bandit (security)

The catalog lives at [Ruff Rules](https://docs.astral.sh/ruff/rules/). Default-on rules are listed separately under [Default Rules](https://docs.astral.sh/ruff/default-rules/).

### Practical example

**1. Start a small project:**

```bash
mkdir my_sample_project
cd my_sample_project
uv init --python 3.13
uv add --dev "ruff>=0.16.0"
```

**2. Add `sample.py` with a few deliberate problems:**

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

**3. Lint it with the teaching rule set:**

```bash
uv run ruff check --select E,W,F,I,B,C4,UP --output-format=concise sample.py
```

Ruff's default output is more verbose (snippets and suggested fixes). Compact form:

```
sample.py:1:1: I001 [*] Import block is un-sorted or un-formatted
sample.py:1:8: F401 [*] `os` imported but unused
sample.py:2:8: F401 [*] `sys` imported but unused
sample.py:3:8: F401 [*] `json` imported but unused
sample.py:13:8: E721 Use `is` and `is not` for type comparisons, or `isinstance()` for isinstance checks
Found 5 errors.
[*] 4 fixable with the `--fix` option.
```

The missing spaces in `[1,2,3,4,5]` are a **formatter** issue, not a linter one. Ruff leaves those to `ruff format`.

**4. Auto-fix what it can:**

```bash
uv run ruff check --select E,W,F,I,B,C4,UP --fix sample.py
```

Unused imports disappear. `E721` stays: change `type(data) == list` to `isinstance(data, list)` yourself. Then format:

```bash
uv run ruff format sample.py
```

Without an explicit `select`, 0.16 defaults still catch the unused imports and import order. They do **not** include `E721`; that is one reason a written `select` list is easier to teach from.

### VS Code

Install the official **Ruff** extension from Astral (`charliermarsh.ruff`). Then in `.vscode/settings.json`:

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

Save a file and the obvious problems go away before you remember to run the CLI.

### Pre-commit

Run Ruff on every commit so a forgotten `ruff check` never reaches the remote.

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

`--exit-non-zero-on-fix` fails the commit after applying fixes, so you restage and commit the cleaned files instead of silently committing Ruff's edits.

```bash
uv run pre-commit install
```

Pin `rev` to a release tag. Bump it when you bump `ruff` in the project.

## Common Issues

### Unused imports

```python
# Flagged (F401)
import os
import sys

print("Hello")

# Fixed: delete them
print("Hello")
```

Python will let you import the entire standard library and never touch it. Ruff will not.

### Undefined names

```python
# Flagged (F821)
def calculate():
    return total

# Fixed
def calculate():
    total = 0
    return total
```

### Mutable default arguments

```python
# Flagged (B006): this list is shared across calls
def add_item(item, items=[]):
    items.append(item)
    return items

# Fixed
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

This is a real bug, not a style preference. The default is created once, when the function is defined.

### Type comparisons

```python
# Flagged (E721)
if type(data) == list:
    pass

# Fixed
if isinstance(data, list):
    pass
```

`isinstance` respects subclasses. `type(...) ==` does not.

## Habits That Stick

Add linting on day one. Retrofitting a large repo is a weekend you will not enjoy; `ruff check . --statistics` at least tells you how large the weekend is.

Fix in batches: `--fix` first, then the remainder by rule. Put the agreed `select` list in `pyproject.toml`, not in someone's memory. Run the same command in CI that you run locally.

If you disable a rule, leave a comment. "We don't like this" is not a reason; "this false-positives on generated code" is.

Use Ruff for formatting too. That is the next chapter.

## Quick Reference

```bash
# Project install (preferred)
uv add --dev "ruff>=0.16.0"
uv run ruff check .
uv run ruff check --fix .
uv run ruff check --watch .
uv run ruff check . --statistics

# No install
uvx ruff check .

# pyproject.toml sketch
# [tool.ruff]
# line-length = 88
# target-version = "py313"
# [tool.ruff.lint]
# select = ["E", "W", "F", "I", "B", "C4", "UP"]
```

## What's Next?

Linting catches mistakes and nags you toward a house style. Formatting makes the house style automatic. Next: `ruff format`.

[← Previous: Version Control](./04-version-control.md) | [Back to README](./README.md) | [Next: Code Formatting →](./06-code-formatting.md)
