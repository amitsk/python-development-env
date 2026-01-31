# Chapter 8: Static Type Checking

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: CI/CD →](./09-ci-cd.md)

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

**Blazing Fast:**
- 10-100x faster than mypy and Pyright
- Optimized for IDE responsiveness with fine-grained incremental analysis

**Developer-Friendly:**
- Comprehensive diagnostics with rich contextual information
- Configurable rule levels and per-file overrides
- Supports partially typed codebases for easier adoption

**Modern Features:**
- Advanced type narrowing and intersection types
- Built-in language server for IDE integration
- Auto-import, code navigation, and completions

### Installing and Using ty

```bash
# Run directly with uvx (no installation needed)
uvx ty check

# Or install with uv
uv pip install ty

# Check your code
ty check

# Watch mode for continuous checking
ty check --watch

# Check specific files or directories
ty check src/
ty check script.py
```

### Configuration

Create `pyproject.toml` configuration:

```toml
[tool.ty]
# Python version
python-version = "3.11"

# Type checking strictness
strict = true

# Exclude patterns
exclude = [
    ".git",
    ".venv",
    "__pycache__",
]

# Include patterns
include = ["src/**/*.py", "tests/**/*.py"]

# Per-file configuration
[[tool.ty.overrides]]
files = ["tests/**"]
disallow-untyped-defs = false
```

### ty vs mypy

| Feature | ty | mypy |
|---------|----|----- |
| Speed | 10-100x faster | Baseline |
| Language | Rust | Python |
| IDE Support | Built-in language server | External tools |
| Maturity | New (2024+) | Mature (2012+) |
| Community | Growing | Large, established |
| Configuration | Modern, simple | Comprehensive |

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

[Pyrefly](https://pyrefly.org/) is a newer tool that aims to make type checking easier and more intuitive.

### Installing Pyrefly

Check the official website for installation instructions:

```bash
# Installation varies by platform
# Visit https://pyrefly.org/ for latest instructions
```

### Pyrefly Features

- Simpler error messages
- Better inference
- Interactive mode
- Focus on developer experience

**Note:** As Pyrefly is newer, check the official documentation for the latest features and usage.

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
# Install as dev dependencies
uv pip install --dev mypy ty

# Run type checking
uv run mypy src/
uv run ty check

# Or use uvx to run ty without installation
uvx ty check

# Add to scripts in pyproject.toml
[project.scripts]
typecheck = "ty check src/"
typecheck-mypy = "mypy src/"
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
# ty (recommended for new projects)
uvx ty check                    # Check with uvx (no install)
ty check                        # Check all files
ty check --watch                # Watch mode
ty check src/                   # Check specific directory

# mypy
mypy .                          # Check all files
mypy --strict .                 # Strict mode
mypy --install-types            # Install missing type stubs
mypy --show-error-codes .       # Show error codes

# Configuration (pyproject.toml)
[tool.ty]
python-version = "3.11"
strict = true

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true

# Common type hints
def func(a: int, b: str) -> bool: ...
var: list[int] = []
opt: str | None = None
```

## What's Next?

With type checking in place, you're writing more robust code. Next, we'll automate all these quality checks with CI/CD, ensuring every code change meets your standards.

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: CI/CD →](./09-ci-cd.md)
