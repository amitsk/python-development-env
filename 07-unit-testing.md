# Chapter 7: Unit Testing

[← Previous: Code Formatting](./06-code-formatting.md) | [Back to README](./README.md) | [Next: Static Type Checking →](./08-static-type-checking.md)

## What is Unit Testing?

A unit test is a small program that checks a small piece of your program, usually a function or method, in isolation. You write it once; it re-runs whenever the code changes.

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

That is the whole idea. The rest of this chapter is machinery so you can do it hundreds of times without going mad.

## Why Write Them?

Tests find bugs while the file is still on your machine. They also give you permission to refactor: if the suite is green, the public behavior you cared about still holds.

They double as documentation that cannot quietly rot. A README can lie. A test fails CI when it lies.

Hard-to-test code is often hard-to-use code. If you cannot call a function without standing up a database, the function is doing too much.

When something breaks, a failed test names the function. That beats `print` and hope. After you fix a bug, leave a test for it. The same bug returning later is embarrassing; a regression test is cheap.

## The Testing Pyramid

```
      /\        End-to-end
     /  \       Whole workflows. Slow. Few.
    /----\
   / Integ\
  /  ration \   Components together. Medium.
 /__________\
  Unit tests    One function, no network. Fast. Many.
```

This chapter is the base of the pyramid. Integration and E2E tests matter; they are slower and you want fewer of them.

## pytest

[pytest](https://github.com/pytest-dev/pytest) is the Python test runner you actually want. Tests are functions named `test_*`, assertions are plain `assert`, and failure output tells you the values.

The 9.x line is current (9.1.1 at the time of writing). Pin the major:

```bash
uv add --dev "pytest>=9.0.0" pytest-cov
uv run pytest --version
```

**Also exists:** `unittest` (stdlib, more ceremony), `nose2` (quiet), `doctest` (examples in docstrings). Fine for small cases. pytest is the default here.

## First Tests

### 1. Something to test (`calculator.py`)

```python
def add(a, b):
    """Add two numbers."""
    return a + b


def subtract(a, b):
    """Subtract b from a."""
    return a - b


def multiply(a, b):
    """Multiply two numbers."""
    return a * b


def divide(a, b):
    """Divide a by b."""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

### 2. A test module (`test_calculator.py`)

pytest collects files matching `test_*.py` or `*_test.py`.

```python
import pytest

from calculator import add, divide, multiply, subtract


def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0


def test_subtract():
    assert subtract(5, 3) == 2
    assert subtract(0, 5) == -5


def test_multiply():
    assert multiply(3, 4) == 12
    assert multiply(0, 100) == 0


def test_divide():
    assert divide(10, 2) == 5
    assert divide(9, 3) == 3


def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)
```

### 3. Run them

```bash
uv run pytest
uv run pytest test_calculator.py
uv run pytest test_calculator.py::test_add
uv run pytest -v
uv run pytest -s          # show print()
```

```
============================= test session starts ==============================
collected 5 items

test_calculator.py .....                                                 [100%]

============================== 5 passed in 0.02s ===============================
```

Five dots. That is the sound of a good morning.

## Layout and config

```
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── calculator.py
│       └── user.py
├── tests/
│   ├── __init__.py
│   ├── test_calculator.py
│   └── test_user.py
├── pyproject.toml
└── README.md
```

Keep tests out of the package you ship. `src/` plus `tests/` is the usual split.

```toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.12"

[tool.pytest.ini_options]
minversion = "9.0"
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "-v",
    "--strict-markers",
    "--tb=short",
]
```

`minversion` is the pytest you expect, not the Python version. 9.x is current; `"8.0"` is acceptable if you still support pytest 8 in CI.

## Writing Tests That Help

Arrange, act, assert. Setup, call, check. One idea per test. Names that say what happened, not `test_1`.

```python
def test_user_full_name():
    # Arrange
    user = User(first_name="John", last_name="Doe")

    # Act
    full_name = user.get_full_name()

    # Assert
    assert full_name == "John Doe"
```

```python
# One behavior per test
def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-2, -3) == -5
```

A 40-line `test_calculator` that checks add, subtract, and multiply tells you "calculator is broken" and nothing else.

## pytest Features You Will Use

### Fixtures

Reusable setup, injected by argument name:

```python
import pytest


@pytest.fixture
def sample_user():
    return User(
        username="testuser",
        email="test@example.com",
        age=25,
    )


def test_user_greeting(sample_user):
    assert sample_user.greet() == "Hello, testuser!"


def test_user_email(sample_user):
    assert sample_user.email == "test@example.com"
```

Setup/teardown with `yield`:

```python
@pytest.fixture
def database():
    db = Database()
    db.connect()
    yield db
    db.disconnect()


def test_database_query(database):
    result = database.query("SELECT * FROM users")
    assert len(result) > 0
```

### Parametrize

Several inputs, one test body:

```python
@pytest.mark.parametrize(
    "a,b,expected",
    [
        (2, 3, 5),
        (0, 0, 0),
        (-1, 1, 0),
        (100, 200, 300),
    ],
)
def test_add_various_inputs(a, b, expected):
    assert add(a, b) == expected
```

### Exceptions

```python
def test_divide_by_zero_raises_error():
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)
    assert "Cannot divide by zero" in str(exc_info.value)


def test_invalid_age_raises_error():
    with pytest.raises(ValueError, match="Age must be positive"):
        User(name="John", age=-5)
```

### Markers

```python
import sys

import pytest


@pytest.mark.slow
def test_large_dataset_processing():
    pass


@pytest.mark.integration
def test_database_connection():
    pass


@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass


@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix_specific_feature():
    pass
```

```bash
uv run pytest -m slow
uv run pytest -m "not slow"
uv run pytest -m integration
```

Register custom markers under `tool.pytest.ini_options` so `--strict-markers` does not complain.

### Async

Need `pytest-asyncio`. Mark async tests (or set `asyncio_mode = "auto"`):

```bash
uv add --dev pytest-asyncio
```

```python
import pytest


@pytest.mark.asyncio
async def test_async_function():
    result = await async_fetch_data()
    assert result == expected_value
```

## Worked Example: a User class

**`user.py`:**

```python
from dataclasses import dataclass


@dataclass
class User:
    username: str
    email: str
    age: int
    is_active: bool = True

    def __post_init__(self):
        if self.age < 0:
            raise ValueError("Age must be non-negative")
        if "@" not in self.email:
            raise ValueError("Invalid email format")

    def greet(self) -> str:
        return f"Hello, {self.username}!"

    def can_vote(self, voting_age: int = 18) -> bool:
        return self.age >= voting_age

    def deactivate(self) -> None:
        self.is_active = False
```

**`test_user.py`:**

```python
import pytest

from user import User


@pytest.fixture
def valid_user():
    return User(
        username="johndoe",
        email="john@example.com",
        age=25,
    )


@pytest.fixture
def minor_user():
    return User(
        username="jane",
        email="jane@example.com",
        age=16,
    )


def test_user_creation(valid_user):
    assert valid_user.username == "johndoe"
    assert valid_user.email == "john@example.com"
    assert valid_user.age == 25
    assert valid_user.is_active is True


def test_user_creation_with_invalid_age():
    with pytest.raises(ValueError, match="Age must be non-negative"):
        User(username="test", email="test@example.com", age=-1)


def test_user_creation_with_invalid_email():
    with pytest.raises(ValueError, match="Invalid email format"):
        User(username="test", email="invalid-email", age=25)


def test_greet(valid_user):
    assert valid_user.greet() == "Hello, johndoe!"


@pytest.mark.parametrize(
    "age,voting_age,can_vote",
    [
        (18, 18, True),
        (25, 18, True),
        (17, 18, False),
        (21, 21, True),
        (20, 21, False),
    ],
)
def test_can_vote(age, voting_age, can_vote):
    user = User(username="test", email="test@example.com", age=age)
    assert user.can_vote(voting_age) == can_vote


def test_deactivate(valid_user):
    assert valid_user.is_active is True
    valid_user.deactivate()
    assert valid_user.is_active is False


def test_user_with_zero_age():
    user = User(username="baby", email="baby@example.com", age=0)
    assert user.age == 0
    assert user.can_vote() is False
```

```bash
uv run pytest test_user.py -v
```

`minor_user` is unused here; delete it or write a test that needs a 16-year-old. Fixtures you never request are just clutter.

## Coverage

Coverage answers "did any test execute this line?" It does not answer "did any test check this line." 100% coverage with no assertions is a very expensive green bar.

```bash
uv add --dev "pytest>=9.0.0" pytest-cov

uv run pytest --cov=mypackage
uv run pytest --cov=mypackage --cov-report=term-missing
uv run pytest --cov=mypackage --cov-report=html
uv run pytest --cov=mypackage --cov-fail-under=80
```

```
---------- coverage: platform linux, python 3.13 -----------
Name                  Stmts   Miss  Cover   Missing
---------------------------------------------------
mypackage/__init__.py     2      0   100%
mypackage/user.py        25      2    92%   45-46
---------------------------------------------------
TOTAL                    27      2    93%
```

```toml
[tool.coverage.run]
source = ["src"]
omit = [
    "*/tests/*",
    "*/test_*.py",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
]
```

## Mocking

Unit tests should not hit the network. Patch the boundary:

```python
from unittest.mock import patch

from mymodule import fetch_user_data


def test_fetch_user_data():
    with patch("mymodule.api_client.get") as mock_get:
        mock_get.return_value = {"name": "John", "age": 30}

        user = fetch_user_data(user_id=123)

        assert user["name"] == "John"
        mock_get.assert_called_once_with("/users/123")
```

## Habits That Stick

TDD (failing test, minimal code, refactor) is optional as a religion and useful as a habit when the behavior is clear before the implementation is.

Keep unit tests fast: no real databases, no sleeps, no "this takes 20 seconds" in the default suite. Mark the slow ones and skip them locally with `-m "not slow"`.

Tests should not depend on each other. Shared mutable fixtures are how you get "fails only when run after test_deactivate".

Test the edges: zero, negatives, empty lists, the error path.

```python
def test_divide_edge_cases():
    assert divide(0, 5) == 0
    assert divide(10**100, 10**50) == 10**50
    assert divide(-10, 2) == -5
```

Assert something specific. `assert result` is a shrug. `assert result == expected` is a test.

Do not assert on `_private` attributes. Call the public API; if you cannot, the API is incomplete.

## Quick Reference

```bash
uv add --dev "pytest>=9.0.0" pytest-cov

uv run pytest
uv run pytest tests/test_file.py
uv run pytest tests/test_file.py::test_name
uv run pytest -k "test_add"
uv run pytest -v
uv run pytest -s
uv run pytest -x              # first failure
uv run pytest --lf            # last failed
uv run pytest --ff            # failed first
uv run pytest --cov=src
uv run pytest --cov=src --cov-report=html
uv run pytest -m slow
uv run pytest -m "not slow"
```

## What's Next?

Tests tell you the code behaves. Static types tell you the code even *type-checks* before it runs. Next: type hints, ty, mypy, and friends.

[← Previous: Code Formatting](./06-code-formatting.md) | [Back to README](./README.md) | [Next: Static Type Checking →](./08-static-type-checking.md)
