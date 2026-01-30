# Chapter 6: Unit Testing

[← Previous: Code Formatting](./05-code-formatting.md) | [Back to README](./README.md) | [Next: Static Type Checking →](./07-static-type-checking.md)

## What is Unit Testing?

Unit testing is the practice of writing code to test your code. A unit test verifies that a small piece of your program (usually a function or method) works correctly in isolation.

Think of unit tests as automated quality checks. Instead of manually testing your code every time you make a change, tests do it for you automatically.

**Example:**
```python
# Code to test
def add(a, b):
    return a + b

# Unit test
def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

## Why Write Unit Tests?

### 1. Catch Bugs Early
Tests catch errors before they reach production. It's much cheaper to fix bugs during development than after deployment.

### 2. Confidence to Refactor
With comprehensive tests, you can refactor code without fear. If tests pass, the code still works.

### 3. Living Documentation
Tests show how code is meant to be used. They're examples that never get out of date because they must work.

### 4. Better Design
Writing tests often reveals design issues. If code is hard to test, it's probably hard to use.

### 5. Faster Debugging
When a test fails, you know exactly where the problem is. No need to debug the entire application.

### 6. Regression Prevention
Once you fix a bug, write a test for it. The bug can never sneak back in unnoticed.

## Testing Pyramid

Tests come in different sizes:

```
      /\        End-to-End (E2E) Tests
     /  \       - Test complete workflows
    /    \      - Slow, expensive
   /------\     - Fewer tests
  / Integ  \    Integration Tests
 /  ration  \   - Test components together
/__________  \  - Medium speed
              \ - Medium number
 Unit Tests    \
 - Test individual functions/methods
 - Fast, cheap
 - Many tests
```

This chapter focuses on unit tests, the foundation of the pyramid.

## Introduction to pytest

[pytest](https://github.com/pytest-dev/pytest) is the most popular Python testing framework. It's powerful yet simple to use.

### Why pytest?

1. **Simple**: Just use `assert`, no special methods
2. **Powerful**: Rich feature set for advanced scenarios
3. **Extensible**: Huge plugin ecosystem
4. **Detailed Output**: Clear failure messages
5. **Widely Adopted**: Industry standard

### Alternative Frameworks

- **unittest**: Built into Python, more verbose
- **nose2**: Based on unittest, less active development
- **doctest**: Tests in docstrings, good for simple examples

pytest is recommended for most projects.

## Installing pytest

```bash
# Using uv
uv pip install pytest

# Or as a development dependency
uv pip install --dev pytest

# Verify installation
pytest --version
```

## Writing Your First Test

### 1. Create a Module to Test (calculator.py)

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

### 2. Create a Test File (test_calculator.py)

pytest discovers test files that match:
- `test_*.py`
- `*_test.py`

```python
from calculator import add, subtract, multiply, divide
import pytest

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

### 3. Run the Tests

```bash
# Run all tests
pytest

# Run specific file
pytest test_calculator.py

# Run specific test
pytest test_calculator.py::test_add

# Verbose output
pytest -v

# Show print statements
pytest -s
```

**Output:**
```
================================ test session starts =================================
collected 5 items

test_calculator.py .....                                                      [100%]

================================= 5 passed in 0.02s ==================================
```

## Test Organization

### Directory Structure

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

### Configuration (pyproject.toml)

```toml
[tool.pytest.ini_options]
# Test discovery patterns
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]

# Output options
addopts = [
    "-v",                    # Verbose
    "--strict-markers",      # Error on unknown markers
    "--tb=short",           # Shorter traceback format
]

# Minimum Python version
minversion = "7.0"
```

## Writing Effective Tests

### The AAA Pattern

Structure tests using Arrange-Act-Assert:

```python
def test_user_full_name():
    # Arrange - Set up test data
    user = User(first_name="John", last_name="Doe")

    # Act - Perform the action
    full_name = user.get_full_name()

    # Assert - Check the result
    assert full_name == "John Doe"
```

### Test One Thing at a Time

```python
# Bad - tests multiple things
def test_calculator():
    assert add(2, 3) == 5
    assert subtract(5, 2) == 3
    assert multiply(2, 3) == 6

# Good - separate tests
def test_add():
    assert add(2, 3) == 5

def test_subtract():
    assert subtract(5, 2) == 3

def test_multiply():
    assert multiply(2, 3) == 6
```

### Use Descriptive Names

```python
# Bad
def test_1():
    assert add(2, 3) == 5

# Good
def test_add_positive_numbers():
    assert add(2, 3) == 5

def test_add_negative_numbers():
    assert add(-2, -3) == -5

def test_add_mixed_signs():
    assert add(-2, 3) == 1
```

## pytest Features

### Fixtures

Fixtures provide reusable test data and setup:

```python
import pytest

@pytest.fixture
def sample_user():
    """Create a sample user for testing."""
    return User(
        username="testuser",
        email="test@example.com",
        age=25
    )

def test_user_greeting(sample_user):
    assert sample_user.greet() == "Hello, testuser!"

def test_user_email(sample_user):
    assert sample_user.email == "test@example.com"
```

### Parametrized Tests

Test multiple inputs with one test:

```python
import pytest

@pytest.mark.parametrize("a,b,expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_add_various_inputs(a, b, expected):
    assert add(a, b) == expected
```

### Testing Exceptions

```python
import pytest

def test_divide_by_zero_raises_error():
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)
    assert "Cannot divide by zero" in str(exc_info.value)

def test_invalid_age_raises_error():
    with pytest.raises(ValueError, match="Age must be positive"):
        User(name="John", age=-5)
```

### Markers

Organize and selectively run tests:

```python
import pytest

@pytest.mark.slow
def test_large_dataset_processing():
    # Slow test
    pass

@pytest.mark.integration
def test_database_connection():
    # Integration test
    pass

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass

@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix_specific_feature():
    pass
```

Run specific markers:
```bash
pytest -m slow              # Run only slow tests
pytest -m "not slow"        # Skip slow tests
pytest -m integration       # Run integration tests
```

## Practical Example: Testing a User Class

**user.py:**
```python
from dataclasses import dataclass
from typing import Optional

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

**test_user.py:**
```python
import pytest
from user import User

# Fixtures
@pytest.fixture
def valid_user():
    return User(
        username="johndoe",
        email="john@example.com",
        age=25
    )

@pytest.fixture
def minor_user():
    return User(
        username="jane",
        email="jane@example.com",
        age=16
    )

# Test initialization
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

# Test methods
def test_greet(valid_user):
    assert valid_user.greet() == "Hello, johndoe!"

@pytest.mark.parametrize("age,voting_age,can_vote", [
    (18, 18, True),
    (25, 18, True),
    (17, 18, False),
    (21, 21, True),
    (20, 21, False),
])
def test_can_vote(age, voting_age, can_vote):
    user = User(username="test", email="test@example.com", age=age)
    assert user.can_vote(voting_age) == can_vote

def test_deactivate(valid_user):
    assert valid_user.is_active is True
    valid_user.deactivate()
    assert valid_user.is_active is False

# Test edge cases
def test_user_with_zero_age():
    user = User(username="baby", email="baby@example.com", age=0)
    assert user.age == 0
    assert user.can_vote() is False
```

Run the tests:
```bash
pytest test_user.py -v
```

## Code Coverage

Code coverage measures how much of your code is executed by tests.

### Installing coverage

```bash
uv pip install pytest-cov
```

### Running with Coverage

```bash
# Show coverage report
pytest --cov=mypackage

# Show line-by-line coverage
pytest --cov=mypackage --cov-report=term-missing

# Generate HTML report
pytest --cov=mypackage --cov-report=html

# Set minimum coverage threshold
pytest --cov=mypackage --cov-fail-under=80
```

**Example output:**
```
---------- coverage: platform linux, python 3.11 -----------
Name                  Stmts   Miss  Cover   Missing
---------------------------------------------------
mypackage/__init__.py     2      0   100%
mypackage/user.py        25      2    92%   45-46
---------------------------------------------------
TOTAL                    27      2    93%
```

### Coverage Configuration (pyproject.toml)

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

## Running Tests with uv

```bash
# Install pytest as dev dependency
uv pip install --dev pytest pytest-cov

# Run tests
uv run pytest

# With coverage
uv run pytest --cov=src

# Watch mode (with pytest-watch)
uv pip install pytest-watch
uv run ptw
```

## Best Practices

### 1. Write Tests First (TDD)

Test-Driven Development:
1. Write a failing test
2. Write minimal code to pass
3. Refactor
4. Repeat

### 2. Keep Tests Fast

- Use mocks for external dependencies
- Avoid actual database calls in unit tests
- Run slow tests separately

### 3. Make Tests Independent

Each test should:
- Set up its own data
- Clean up after itself
- Not depend on other tests' state

### 4. Test Edge Cases

```python
def test_divide_edge_cases():
    # Zero numerator
    assert divide(0, 5) == 0

    # Very large numbers
    assert divide(10**100, 10**50) == 10**50

    # Negative numbers
    assert divide(-10, 2) == -5
```

### 5. Use Meaningful Assertions

```python
# Bad
assert result

# Good
assert result == expected_value
assert len(items) == 3
assert user.is_active is True
```

### 6. Don't Test Implementation Details

Test behavior, not implementation:

```python
# Bad - tests internal implementation
def test_user_storage():
    user = User("john", "john@example.com")
    assert user._internal_cache is not None

# Good - tests public behavior
def test_user_email_retrieval():
    user = User("john", "john@example.com")
    assert user.get_email() == "john@example.com"
```

## Common Testing Patterns

### Testing with Mock Data

```python
from unittest.mock import Mock, patch

def test_fetch_user_data():
    # Mock the API call
    with patch('mymodule.api_client.get') as mock_get:
        mock_get.return_value = {"name": "John", "age": 30}

        user = fetch_user_data(user_id=123)

        assert user["name"] == "John"
        mock_get.assert_called_once_with("/users/123")
```

### Testing Async Code

```python
import pytest

@pytest.mark.asyncio
async def test_async_function():
    result = await async_fetch_data()
    assert result == expected_value
```

### Setup and Teardown

```python
@pytest.fixture
def database():
    # Setup
    db = Database()
    db.connect()

    yield db  # Test runs here

    # Teardown
    db.disconnect()

def test_database_query(database):
    result = database.query("SELECT * FROM users")
    assert len(result) > 0
```

## Quick Reference

```bash
# Run tests
pytest                          # All tests
pytest test_file.py            # Specific file
pytest test_file.py::test_name # Specific test
pytest -k "test_add"           # Tests matching pattern

# Options
pytest -v                      # Verbose
pytest -s                      # Show print output
pytest -x                      # Stop on first failure
pytest --lf                    # Run last failed tests
pytest --ff                    # Run failed tests first

# Coverage
pytest --cov=src              # Show coverage
pytest --cov=src --cov-report=html  # HTML report

# Markers
pytest -m slow                # Run tests with @pytest.mark.slow
pytest -m "not slow"          # Skip slow tests
```

## What's Next?

Testing ensures your code works correctly. Next, we'll explore static type checking, which helps catch type-related bugs before running your code.

[← Previous: Code Formatting](./05-code-formatting.md) | [Back to README](./README.md) | [Next: Static Type Checking →](./07-static-type-checking.md)
