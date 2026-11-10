# Chapter 8: Static Type Checking

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: Logging →](./09-logging.md)

## What is Static Type Checking?

Python checks types at runtime. That is flexible. It is also how `greet(123)` becomes an error only after someone actually calls it.

A static type checker reads annotations and traces types through the program *without running it*.

```python
def greet(name):
    return "Hello, " + name

greet("Alice")  # fine
greet(123)      # TypeError, but only when this line runs
```

```python
def greet(name: str) -> str:
    return "Hello, " + name

greet("Alice")  # fine
greet(123)      # the type checker flags this now
```

The interpreter still will not care. The checker will. That gap is the whole feature.

## Why Annotate?

Wrong types show up in the editor and in CI instead of in a stack trace from last Tuesday. Hints also document the function in a way the tooling can verify, so they stay honest.

Editors use them for completion and rename. Change a signature and the checker lists the call sites you forgot. You do not have to annotate a whole legacy tree on day one: start at the edges and work inward.

Tests still matter. Types will not catch `return a - b` in a function named `add`. They catch a different class of mistake. Use both.

## Type Hints, the Normal Syntax

Python 3.9+ built-ins are generic. Python 3.10+ unions use `|`. This tutorial targets 3.14, so this is the syntax. You do not need `from typing import List, Dict, Optional` for ordinary collections.

### Scalars and functions

```python
name: str = "Alice"
age: int = 30
height: float = 5.8
is_active: bool = True


def add(a: int, b: int) -> int:
    return a + b


def greet(name: str) -> str:
    return f"Hello, {name}"


def log_message(message: str) -> None:
    print(message)
```

### Collections and optionals

```python
numbers: list[int] = [1, 2, 3, 4]
names: list[str] = ["Alice", "Bob"]

scores: dict[str, int] = {"age": 30, "score": 100}
config: dict[str, str] = {"host": "localhost", "port": "5432"}

tags: set[str] = {"python", "typing"}

point: tuple[int, int] = (10, 20)
person: tuple[str, int, bool] = ("Alice", 30, True)

# None is a union, not a special import
name: str | None = None
```

### Unions, Any, Callable

```python
from collections.abc import Callable
from typing import Any, TypedDict


def process(value: int | str) -> str:
    return str(value)


def flexible(data: Any) -> Any:
    return data


def apply_function(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)


class UserRecord(TypedDict):
    name: str
    age: int
```

`Any` is the escape hatch. Use it at untyped boundaries, not as the type of everything you could not be bothered to name.

`Callable` and `TypedDict` still live in `typing` / `collections.abc`. That is expected.

### Classes

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

### Type aliases

Python 3.12+ has a dedicated statement:

```python
type UserId = int
type UserData = dict[str, str | int | bool]


def get_user_data(user_id: UserId) -> UserData:
    ...
```

Older style, still valid:

```python
from typing import TypeAlias

UserId: TypeAlias = int
```

Prefer `type UserId = int` in new 3.12+ code.

## The Checkers

| Tool | Who | Language | Role here |
|------|-----|----------|-----------|
| **[ty](https://github.com/astral-sh/ty)** | Astral | Rust | Tutorial default. Fast. **Beta** (0.0.100). |
| **[mypy](https://github.com/python/mypy)** | python/mypy | Python | Mature CLI checker. Safe default if you want stable diagnostics. |
| **[Pyright](https://github.com/microsoft/pyright)** / Pylance | Microsoft | TypeScript | Editor type checking in VS Code. |
| **[Pyrefly](https://pyrefly.org/)** | Meta | Rust | Fast alternative, strong on large repos. |

For this stack (uv, Ruff, VS Code), use **ty or mypy on the CLI**. This chapter uses ty. Pin the ty version in CI: it is beta, and diagnostics can change between releases.

Pyright still powers Pylance. You can run ty in CI and keep Pylance in the editor, or switch the editor to ty's language server. Do not run three checkers against the same file and expect identical output.

## ty (tutorial default)

[ty](https://docs.astral.sh/ty/) is Astral's type checker: same shop as uv and Ruff, written in Rust, built-in language server. It is **beta** as of 0.0.100. Fast, good on partially typed code, not a frozen specification yet.

Say that out loud to your team before you make it a merge gate. Then pin it.

Docs: [docs.astral.sh/ty](https://docs.astral.sh/ty/). Playground (no install): [play.ty.dev](https://play.ty.dev).

### Install and run

```bash
uv add --dev ty
uv run ty check
uv run ty check src/

# No project install
uvx ty check
uvx ty check src/
```

Pin in `pyproject.toml` (or your lockfile) so CI and laptops match. When you upgrade ty, read the changelog and expect some new (or quieter) diagnostics.

### Configuration

```toml
[project]
requires-python = ">=3.13"

[tool.ty.environment]
python-version = "3.14"

[tool.ty.src]
include = ["src", "tests"]

# Optional: tune one rule at a time
# [tool.ty.rules]
# possibly-unresolved-reference = "warn"
```

Full keys: [ty configuration](https://docs.astral.sh/ty/reference/configuration/).

### Example

**`calculator.py`:**

```python
def add(a: int, b: int) -> int:
    return a + b


def greet(name: str) -> str:
    return f"Hello, {name}"


result = add("1", "2")
```

```bash
uvx ty check calculator.py
```

```
error[invalid-argument-type]: Argument to function `add` is incorrect
 --> calculator.py:8:14
  |
8 | result = add("1", "2")
  |              ^^^ Expected `int`, found `Literal["1"]`
info: Function defined here
 --> calculator.py:1:5
  |
1 | def add(a: int, b: int) -> int:
  |     ^^^ ------ Parameter declared here

error[invalid-argument-type]: Argument to function `add` is incorrect
 --> calculator.py:8:19
  |
8 | result = add("1", "2")
  |                   ^^^ Expected `int`, found `Literal["2"]`
info: Function defined here
 --> calculator.py:1:5
  |
1 | def add(a: int, b: int) -> int:
  |     ^^^         ------ Parameter declared here

Found 2 diagnostics
```

Two arguments, two errors. Concatenating `"1"` and `"2"` would have been a perfectly typed way to get the wrong answer; the checker is not a substitute for `test_add`.

### Editor

ty ships a language server. In VS Code, install the official **ty** extension from Astral (`astral-sh.ty`). It can sit alongside Pylance or replace it; if a ty feature is still missing, keep Pylance for that part and let CI be the source of truth.

Neovim and other LSP clients can run `ty server`. See [the editor docs](https://docs.astral.sh/ty/).

### When to use ty

Good fit: new projects on uv/Ruff, large trees where mypy is slow, editors that want a fast language server.

Caveats: beta. Pin the version. Do not assume a diagnostic that exists today will have the same code next month. Advanced mypy plugins are not a ty feature.

If you need a checker whose behavior has been boring for a decade, that is mypy.

## mypy

[mypy](https://github.com/python/mypy) is the original Python type checker. Mature, well documented, still the thing many CI pipelines run.

```bash
uv add --dev mypy
uv run mypy src/
uv run mypy --strict src/
uv run mypy --show-error-codes src/
```

```toml
[tool.mypy]
python_version = "3.14"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
show_error_codes = true
pretty = true

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

Typical first errors: missing annotations (`def process(data):`) and `return "25"` from a function declared `-> int`. Add the hint, or fix the return. `--strict` is reasonable on new packages and loud on old ones; turn flags on one at a time if you are migrating.

mypy is slower than ty on big repos. It is also less likely to change its mind next week. Either is a valid CLI choice; pick one as the merge gate.

## Pyright and Pylance

[Pyright](https://github.com/microsoft/pyright) is Microsoft's checker. [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance) is the VS Code extension that uses it.

```bash
uvx pyright
```

Pylance settings, if you keep it as the in-editor checker:

```json
{
    "python.analysis.typeCheckingMode": "basic",
    "python.analysis.diagnosticMode": "workspace"
}
```

`"strict"` is available when you want it. PyCharm has its own inspections under Settings → Editor → Inspections → Python → Type checker.

## Pyrefly

[Pyrefly](https://pyrefly.org/) is Meta's Rust type checker (the successor in spirit to Pyre). Fast incremental checking, LSP, playground on the site.

```bash
uvx pyrefly check
uv add --dev pyrefly
uv run pyrefly check src/
```

With **no config**, Pyrefly uses a conservative `basic` preset: high-confidence issues only. That preset will *not* report `add("1", "2")`. Run `uvx pyrefly init` (or add a `[tool.pyrefly]` table) before you treat it as a real checker.

```toml
[tool.pyrefly]
python-version = "3.14"
```

After init, the same `calculator.py` looks like this:

```
ERROR Argument `Literal['1']` is not assignable to parameter `a` with type `int` in function `add` [bad-argument-type]
 --> calculator.py:8:14
  |
8 | result = add("1", "2")
  |              ^^^
```

Useful as a second opinion, or if you already live in Meta's Python world. For this tutorial's CLI, prefer ty or mypy so the rest of the uv/Ruff setup stays one-vendor-simple.

## Gradual Typing

You do not owe the type checker a complete novel.

1. **Public functions first.** `create_user(name: str, email: str) -> User` pays rent immediately. `_validate_internal` can wait.
2. **All new code.** Stop adding untyped surfaces.
3. **The scary modules.** The ones that keep producing bugs get annotations next.
4. **The rest**, when you have a slow afternoon.

Empty containers need a hint (`items: list[int] = []`). `name = "Alice"` does not. The checker can see that one.

## Stubs

Third-party code without hints can get a `.pyi` file next to it, or in a stubs package:

```python
# mylib.pyi
def process(data: str) -> int: ...

class Handler:
    def handle(self, value: int) -> None: ...
```

Many popular libraries already ship types or have stubs on Typeshed. Check before you write your own.

Suppressions exist for a reason. Use them with a code, and sparingly:

```python
result = some_untyped_library_function()  # type: ignore[misc]
```

A stub is better than a blanket ignore. A typed wrapper around the untyped library is better still.

## Types Are Not Tests

```python
def add(a: int, b: int) -> int:
    return str(a + b)  # checker: returning str, declared int


def add(a: int, b: int) -> int:
    return a - b  # checker: types are fine. tests: not fine.
```

The first function is a type error. The second is a logic error with perfect types. Run `uv run pytest` *and* `uv run ty check`.

## Quick Reference

```bash
# ty (default here; beta — pin it)
uv add --dev ty
uv run ty check
uv run ty check src/
uvx ty check

# mypy (stable alternative)
uv add --dev mypy
uv run mypy src/
uv run mypy --strict src/

# Pyrefly (init first, or it stays on the basic preset)
uvx pyrefly init
uvx pyrefly check

# [tool.ty.environment]
# python-version = "3.14"
# [tool.ty.src]
# include = ["src", "tests"]

# [tool.mypy]
# python_version = "3.14"
# strict = true

# Everyday hints
def func(a: int, b: str) -> bool: ...
numbers: list[int] = []
maybe: str | None = None
type UserId = int
```

## What's Next?

Types make the shape of the program explicit. Logging makes its *behavior* visible once it is running. Next chapter: stop relying on `print()`.

[← Previous: Unit Testing](./07-unit-testing.md) | [Back to README](./README.md) | [Next: Logging →](./09-logging.md)
