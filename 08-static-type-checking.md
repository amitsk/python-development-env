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

## Deep Dive: mypy

mypy is the most widely used type checker. Let's explore it in detail.

### Installing mypy

```bash
# Using uv
uv pip install mypy

# Verify
mypy --version
```

### Basic Usage

```bash
# Check all Python files
mypy .

# Check specific files or directories
mypy src/
mypy script.py

# Strict mode (recommended for new projects)
mypy --strict .

# Show error codes
mypy --show-error-codes .
```

### Configuration

Create a `myproject.toml` configuration:

```toml
[tool.mypy]
# Basic options
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true

# Strictness options
disallow_untyped_defs = true
disallow_any_unimported = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_unreachable = true

# Import discovery
mypy_path = "src"
packages = ["mypackage"]

# Output
show_error_codes = true
pretty = true

# Per-module options
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

### Common mypy Errors and Fixes

#### Error: Function is missing a type annotation

```python
# Error
def process(data):
    return data.upper()

# Fix
def process(data: str) -> str:
    return data.upper()
```

#### Error: Incompatible return value type

```python
# Error
def get_age() -> int:
    return "25"  # Error: Expected int, got str

# Fix
def get_age() -> int:
    return 25
```

#### Error: Argument has incompatible type

```python
# Error
def greet(name: str) -> str:
    return f"Hello, {name}"

greet(123)  # Error: Argument 1 has incompatible type "int"; expected "str"

# Fix
greet(str(123))  # Convert to string
# Or
greet("Alice")  # Use correct type
```

#### Error: Need type annotation for variable

```python
# Error
items = []  # Error: Need type annotation for "items"

# Fix
items: list[int] = []
# Or
items: list[str] = []
```

## Practical Example with mypy

**user_manager.py:**
```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class User:
    username: str
    email: str
    age: int
    is_active: bool = True

class UserManager:
    def __init__(self) -> None:
        self.users: dict[str, User] = {}

    def add_user(self, user: User) -> None:
        """Add a user to the manager."""
        if user.username in self.users:
            raise ValueError(f"User {user.username} already exists")
        self.users[user.username] = user

    def get_user(self, username: str) -> Optional[User]:
        """Get a user by username."""
        return self.users.get(username)

    def remove_user(self, username: str) -> bool:
        """Remove a user. Returns True if user was removed."""
        if username in self.users:
            del self.users[username]
            return True
        return False

    def get_active_users(self) -> list[User]:
        """Get all active users."""
        return [user for user in self.users.values() if user.is_active]

    def count_users(self) -> int:
        """Count total users."""
        return len(self.users)

def create_sample_users() -> list[User]:
    """Create sample users for testing."""
    return [
        User("alice", "alice@example.com", 30),
        User("bob", "bob@example.com", 25),
        User("charlie", "charlie@example.com", 35, is_active=False),
    ]
```

**Run mypy:**
```bash
mypy user_manager.py
```

**Output:**
```
Success: no issues found in 1 source file
```

Now let's introduce a type error:

```python
def process_user(user: User) -> str:
    return user.age  # Error: should return str, not int

manager = UserManager()
manager.add_user("not a user")  # Error: expected User, got str
```

**mypy output:**
```
user_manager.py:42: error: Incompatible return value type (got "int", expected "str")
user_manager.py:45: error: Argument 1 to "add_user" has incompatible type "str"; expected "User"
```

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
uv pip install --dev mypy pyright

# Run type checking
uv run mypy src/
uv run pyright

# Add to scripts in pyproject.toml
[project.scripts]
typecheck = "mypy src/"
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
# mypy
mypy .                          # Check all files
mypy --strict .                 # Strict mode
mypy --install-types            # Install missing type stubs
mypy --show-error-codes .       # Show error codes

# Configuration (pyproject.toml)
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
