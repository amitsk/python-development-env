# Chapter 8: Static Type Checking

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: Logging →](./09-logging.md)

## What is Static Type Checking?

Python is a dynamically typed language - you don't have to declare variable types, and types are checked at runtime. This flexibility is powerful but can lead to type-related bugs that only appear when code executes.

Static type checking analyzes your code without running it, catching type errors before your program runs.

**Example without type hints:**
```python
def greet(name):
    return "Hello, " + name

greet("Alice")  # Works
greet(123)      # Runtime error: can only concatenate str (not "int") to str
```

**With type hints:**
```python
def greet(name: str) -> str:
    return "Hello, " + name

greet("Alice")  # OK
greet(123)      # Type checker catches this before runtime!
```

## Why Use Static Type Checking?

### 1. Catch Bugs Early
Type errors are found before code runs, often catching bugs that tests might miss.

### 2. Better Documentation
Type hints serve as inline documentation that's always up-to-date because type checkers verify them.

### 3. Improved IDE Support
IDEs provide better autocomplete, refactoring, and error detection when types are known.

### 4. Easier Refactoring
When changing function signatures, type checkers help you find all places that need updates.

### 5. Enhanced Readability
Type hints make code intentions clearer:
```python
def process_user_data(data: dict[str, Any]) -> User:
    ...
```
You immediately know this function takes a dictionary and returns a User object.

### 6. Gradual Adoption
You can add type hints incrementally - no need to annotate everything at once.

## Python Type Hints Basics

### Basic Types

```python
# Simple types
name: str = "Alice"
age: int = 30
height: float = 5.8
is_active: bool = True

# Function annotations
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}"

def log_message(message: str) -> None:  # Returns nothing
    print(message)
```

### Collection Types

```python
from typing import List, Dict, Set, Tuple, Optional

# Lists
numbers: list[int] = [1, 2, 3, 4]
names: list[str] = ["Alice", "Bob"]

# Dictionaries
user: dict[str, int] = {"age": 30, "score": 100}
config: dict[str, str] = {"host": "localhost", "port": "5432"}

# Sets
tags: set[str] = {"python", "typing", "mypy"}

# Tuples
point: tuple[int, int] = (10, 20)
person: tuple[str, int, bool] = ("Alice", 30, True)

# Optional (can be None)
from typing import Optional
name: Optional[str] = None  # Can be str or None
# Or using the newer syntax (Python 3.10+)
name: str | None = None
```

### Complex Types

```python
from typing import Union, Any, Callable

# Union (multiple possible types)
def process(value: Union[int, str]) -> str:
    return str(value)

# Python 3.10+ syntax
def process(value: int | str) -> str:
    return str(value)

# Any (any type)
from typing import Any
def flexible_function(data: Any) -> Any:
    return data

# Callable (function types)
from typing import Callable
def apply_function(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)
```

### Classes and Custom Types

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
    email: str

def create_user(name: str, age: int, email: str) -> User:
    return User(name, age, email)

def process_users(users: list[User]) -> dict[str, User]:
    return {user.email: user for user in users}
```

## Overview of Type Checkers

### mypy

[mypy](https://github.com/python/mypy) is the original and most popular Python type checker, created by the inventor of Python type hints.

```bash
# Install
uv pip install mypy

# Run
mypy myproject/

# Check specific file
mypy script.py
```

**Pros:**
- Official type checker
- Mature and stable
- Excellent documentation
- Large community

**Cons:**
- Can be slow on large projects
- Sometimes requires complex configuration

### Pyright

[Pyright](https://github.com/microsoft/pyright) is a fast type checker from Microsoft, written in TypeScript.

```bash
# Install (requires Node.js)
npm install -g pyright

# Run
pyright

# Or use via pip wrapper
uv pip install pyright
pyright
```

**Pros:**
- Very fast
- Excellent error messages
- Powers Pylance (VSCode extension)
- Active development

**Cons:**
- Requires Node.js (for standalone)
- Less configurable than mypy

### Pyre

[Pyre](https://github.com/facebook/pyre-check) is Facebook's type checker, focused on performance.

**Pros:** Very fast, incremental checking
**Cons:** Complex setup, primarily designed for large codebases

### Pytype

[Pytype](https://github.com/google/pytype) from Google can infer types even without annotations.

**Pros:** Type inference without annotations
**Cons:** Slower, Linux/macOS only

## mypy

mypy is the most widely used type checker, created by the inventor of Python type hints.

### Installing and Using mypy

```bash
# Using uv
uv pip install mypy

# Check all Python files
mypy .

# Strict mode (recommended for new projects)
mypy --strict .

# Show error codes
mypy --show-error-codes .
```

### Basic Configuration

Create a `pyproject.toml` configuration:

```toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
show_error_codes = true
pretty = true

# Per-module options
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

### Common mypy Errors

```python
# Error: Function is missing a type annotation
def process(data):  # Error
    return data.upper()

def process(data: str) -> str:  # Fix
    return data.upper()

# Error: Incompatible return value type
def get_age() -> int:
    return "25"  # Error: Expected int, got str

def get_age() -> int:
    return 25  # Fix
```

## ty: The Fast Type Checker

[ty](https://github.com/astral-sh/ty) is a new Python type checker from Astral (the team behind uv and Ruff), written in Rust for exceptional performance.

### Why ty?

**Fast and IDE-friendly:**
- Written in Rust for high throughput
- Fine-grained incremental analysis for editor responsiveness
- Much faster than classic mypy on large codebases (always measure for your repo)

**Developer-Friendly:**
- Clear diagnostics
- Configurable rule levels and per-file overrides
- Works well with partially typed codebases

**Modern Features:**
- Built-in language server for IDE integration
- Project-aware configuration via `pyproject.toml`

### Installing and Using ty

```bash
# Run without a permanent install
uvx ty check

# Or add as a dev dependency
uv add --dev ty

uv run ty check
uv run ty check src/
uv run ty check script.py
```

### Configuration

Check [ty’s configuration docs](https://docs.astral.sh/ty/reference/configuration/) for the latest keys. A common shape:

```toml
[tool.ty.environment]
python-version = "3.12"

[tool.ty.src]
include = ["src", "tests"]
exclude = [".venv", "build"]

[tool.ty.rules]
# Example: tune individual rules ("error" | "warn" | "ignore")
# possibly-unresolved-reference = "warn"
```

### ty vs mypy

| Feature | ty | mypy |
|---------|----|----- |
| Speed | Very fast (Rust) | Baseline (Python) |
| Language | Rust | Python |
| IDE Support | Built-in language server | External / plugins |
| Maturity | Newer | Mature (2012+) |
| Community | Growing | Large, established |
| Configuration | Modern `tool.ty.*` | Very mature / plugin-rich |

### Example Usage

**calculator.py:**
```python
def add(a: int, b: int) -> int:
    return a + b

def greet(name: str) -> str:
    return f"Hello, {name}"

# Type error
result = add("1", "2")  # ty will catch this!
```

**Run ty:**
```bash
uvx ty check calculator.py
```

**Output:**
```
calculator.py:8:14 - error: Expected type 'int' but got 'str'
    8 | result = add("1", "2")
                     ^^^
```

### IDE Integration

ty includes a built-in language server that works with:
- **VS Code**: Install the ty extension
- **PyCharm**: Configure as external tool
- **Neovim**: Use built-in LSP client

**VS Code settings.json:**
```json
{
    "ty.enable": true,
    "ty.checkOnSave": true
}
```

### Try ty Online

Experiment with ty without installation at [play.ty.dev](https://play.ty.dev).

### When to Use ty

**Good fit:**
- New projects wanting the fastest type checker
- Large codebases where mypy is slow
- Teams already using uv and Ruff (consistent tooling)
- Projects prioritizing IDE performance

**Considerations:**
- Newer tool with evolving features
- Smaller community compared to mypy
- Some advanced mypy plugins may not be available yet

For more details, visit the [official documentation](https://docs.astral.sh/ty/).

## Pyrefly

[Pyrefly](https://pyrefly.org/) is a Python type checker developed and open-sourced by Meta (Facebook) in 2025. It is written in Rust for high performance and is designed to handle large codebases with fast incremental checking.

### Why Pyrefly?

**Performance:**
- Written in Rust — extremely fast type checking
- Incremental analysis: only re-checks files that changed
- Handles large monorepos that can slow down mypy

**Developer Experience:**
- Clear, actionable error messages with rich context
- Interactive playground at [pyrefly.org](https://pyrefly.org/) — try it without installing
- Built-in language server for IDE integration (LSP)

**Modern Design:**
- Full support for Python's typing system (PEP 484 and beyond)
- Good type inference for unannotated code
- Designed to be compatible with existing typed codebases

### Installing Pyrefly

```bash
# Install with pip or uv
pip install pyrefly

# Or using uv
uv pip install pyrefly

# Or run without installing
uvx pyrefly check
```

For the latest installation options, visit [pyrefly.org](https://pyrefly.org/).

### Basic Usage

```bash
# Check all Python files in the current directory
pyrefly check

# Check a specific file
pyrefly check src/mymodule.py

# Check a directory
pyrefly check src/

# Watch mode — re-check on file changes
pyrefly check --watch
```

### Example

**calculator.py:**
```python
def add(a: int, b: int) -> int:
    return a + b

# Type error: passing strings where ints are expected
result = add("1", "2")
```

**Run Pyrefly:**
```bash
pyrefly check calculator.py
```

**Output:**
```
calculator.py:5:14 error: Expected `int`, got `str` [arg-type]
    5 | result = add("1", "2")
                      ^^^
```

### Configuration

Pyrefly can be configured in `pyproject.toml`:

```toml
[tool.pyrefly]
# Python version to check against
python_version = "3.11"

# Paths to check
include = ["src/", "tests/"]

# Paths to skip
exclude = [".venv/", "build/"]
```

### IDE Integration

Pyrefly ships a language server that integrates with editors via LSP:

- **VS Code**: Install the Pyrefly extension from the marketplace
- **Neovim / Vim**: Configure with nvim-lspconfig
- **Other editors**: Any editor supporting LSP works

### Pyrefly vs Other Type Checkers

| Feature | Pyrefly | ty | mypy |
|---------|---------|-----|------|
| Author | Meta | Astral | Guido van Rossum et al. |
| Language | Rust | Rust | Python |
| Speed | Very fast | Very fast | Baseline |
| IDE Support | Built-in LSP | Built-in LSP | External tools |
| Maturity | New (2025) | New (2024+) | Mature (2012+) |
| Inference | Strong | Strong | Good |

### When to Use Pyrefly

**Good fit:**
- Large codebases where mypy is slow
- Teams already familiar with Meta's Python ecosystem
- Projects that want a second opinion alongside mypy or ty

**Try the Playground:**
Visit [pyrefly.org](https://pyrefly.org/) to experiment with Pyrefly in your browser before installing it locally.

**Note:** As Pyrefly is relatively new, check the [official documentation](https://pyrefly.org/) for the latest installation steps, configuration options, and features.

## Gradual Typing Strategy

You don't need to add types to everything at once. Here's a pragmatic approach:

### Level 1: Public APIs

Start with public-facing functions:

```python
# Type this
def create_user(name: str, email: str) -> User:
    ...

# But internal helpers can wait
def _validate_internal(data):
    ...
```

### Level 2: New Code

Add type hints to all new code you write.

### Level 3: Bug-Prone Areas

Add types to code that frequently has bugs or is hard to understand.

### Level 4: Complete Coverage

Gradually fill in the rest as time permits.

## Using Type Checkers with uv

```bash
uv add --dev mypy ty

uv run mypy src/
uv run ty check
uvx ty check
```

## IDE Integration

### VSCode with Pylance

Pylance (based on Pyright) provides excellent type checking:

**settings.json:**
```json
{
    "python.analysis.typeCheckingMode": "basic",  // or "strict"
    "python.analysis.diagnosticMode": "workspace",
    "editor.inlayHints.enabled": "on"
}
```

### PyCharm

PyCharm has built-in type checking:
- Enable: Settings → Editor → Inspections → Python → Type checker

## Best Practices

### 1. Start with Function Signatures

```python
# Good starting point
def process_data(data: dict[str, Any]) -> list[str]:
    ...
```

### 2. Use `Any` Sparingly

```python
# Avoid
def process(data: Any) -> Any:
    ...

# Better - be specific
def process(data: dict[str, int]) -> list[str]:
    ...
```

### 3. Leverage Type Inference

```python
# No need to annotate obvious cases
name = "Alice"  # Clearly str
count = 0       # Clearly int

# Do annotate when not obvious
items = []      # list[what?]
items: list[int] = []  # Better
```

### 4. Use Modern Syntax (Python 3.10+)

```python
# Old
from typing import Optional, Union
def process(value: Optional[Union[int, str]]) -> Union[dict, list]:
    ...

# New (Python 3.10+)
def process(value: int | str | None) -> dict | list:
    ...
```

### 5. Document Complex Types

```python
# Define type aliases for clarity
UserId = int
UserData = dict[str, Union[str, int, bool]]

def get_user_data(user_id: UserId) -> UserData:
    ...
```

### 6. Ignore When Necessary

Sometimes you need to bypass type checking:

```python
# Use type: ignore sparingly
result = some_untyped_library_function()  # type: ignore[misc]

# Better: create a typed stub
```

## Testing with Types

Type checkers complement tests but don't replace them:

```python
# Type checker catches this
def add(a: int, b: int) -> int:
    return str(a + b)  # Error: returning str instead of int

# Type checker can't catch this
def add(a: int, b: int) -> int:
    return a - b  # Wrong logic, but correct types!
```

Both type checking AND testing are essential.

## Stub Files

For libraries without type hints, create stub files (`.pyi`):

**mylib.pyi:**
```python
def process(data: str) -> int: ...

class Handler:
    def handle(self, value: int) -> None: ...
```

## Quick Reference

```bash
# ty (great default for new projects)
uvx ty check
uv run ty check
uv run ty check src/

# mypy
uv run mypy .
uv run mypy --strict .
uv run mypy --show-error-codes .

# Configuration sketches (pyproject.toml)
# [tool.ty.environment]
# python-version = "3.12"
#
# [tool.mypy]
# python_version = "3.12"
# strict = true

# Common type hints
def func(a: int, b: str) -> bool: ...
var: list[int] = []
opt: str | None = None
```

## What's Next?

With type checking in place, you're writing more robust code. Next: **logging** — so production systems report what they do without relying on `print()`.

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: Logging →](./09-logging.md)
