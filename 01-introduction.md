# Chapter 1: Introduction

[← Back to README](./README.md) | [Next: Virtual Environments →](./02-virtual-environments.md)

## Background

This guide is about a professional Python *development environment*. Before we install anything, a boundary:

**This is not a Python programming tutorial.** We assume you already know the language well enough to write a function, import a module, and not panic at a stack trace. If you need to learn or refresh Python itself, use these:

- [Official Python Tutorial](https://docs.python.org/3/tutorial/) — the authoritative guide from Python.org
- [Real Python](https://realpython.com/) — tutorials and articles with more patience than this chapter
- [Python for Beginners](https://www.python.org/about/gettingstarted/) — getting-started resources from python.org
- [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) — practical Python for people who have jobs to finish

**This is a guide about professional development practices.** The tools and workflows that keep code maintainable, make collaboration possible, and stop “it works on my machine” from becoming a personality trait.

As of August 2026, Python **3.14** is the current stable release, **3.13** is the previous one, and **3.12** is still a reasonable floor. Examples in this guide prefer 3.13 and declare `requires-python = ">=3.12"`. Testing a matrix of versions belongs later; here, just do not start a new project on 3.11 because an old screenshot said it was current.

## Software Engineering Best Practices

Tools change. The practices do not. Here is the short version of what “professional” actually means, with pointers to the chapters that implement it.

### Version Control

Version control records what changed, when, and why. You get a history you can search, a way to work on a feature without wrecking `main`, and a path back when a change was a bad idea.

We cover Git in [Chapter 4](./04-version-control.md).

### Linting

A linter reads your code and complains before your users do: unused imports, undefined names, the `except:` that swallows everything. On a team, it also enforces a baseline so code review is not a spelling bee.

Linting tools are in [Chapter 5](./05-linting.md).

### Code Formatting

Formatting is the agreement to stop arguing about whitespace. Consistent code is easier to read, diffs are smaller, and nobody has to spend a pull request debating hanging indents.

Formatting is [Chapter 6](./06-code-formatting.md).

### Unit Testing

Tests are code that checks your code. They give you the confidence to refactor, they document expected behavior better than a comment you will not update, and they turn “something broke” into “this assertion failed.”

Testing is [Chapter 7](./07-unit-testing.md).

### CI/CD (Continuous Integration/Continuous Deployment)

CI/CD is the machine that runs the checks you would skip on a Friday. Tests, linters, and deploys happen on every change, the same way, without a checklist taped to a monitor.

We set that up in [Chapter 10](./10-ci-cd.md).

## Tools We'll Use

### uv: Modern Python Package and Project Manager

Throughout this tutorial we use [uv](https://github.com/astral-sh/uv), a fast Python package and project manager written in Rust. You will live in these commands:

- `uv init` — start a project (`pyproject.toml`, layout, a sensible `.gitignore`)
- `uv add` / `uv add --dev` — declare runtime and development dependencies
- `uv sync` — install from the lockfile into `.venv` (project + the `dev` group by default)
- `uv lock --upgrade` — refresh the lockfile toward the latest allowed versions
- `uv run` — run a script or tool in the project environment without activating anything
- `uvx` — run a tool once without making it a project dependency
- `uv tree` — see what you actually installed
- `uv python install` — install a Python version if your machine does not have the one you want

Lockfiles (`uv.lock`) make installs reproducible. When you meet a `requirements.txt` from 2019, uv still speaks that dialect.

#### Installing uv

Use the standalone installer. That is the supported path.

**macOS and Linux:**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Close and reopen the terminal (or source your shell config) so `uv` is on your `PATH`. Check with `uv --version`.

**Using pip (last resort):** if the standalone installer is not an option and you already have Python:

```bash
pip install uv
```

Prefer the standalone installer. `pip install uv` is a fallback, not the recommended setup. More options live in the [uv installation docs](https://docs.astral.sh/uv/getting-started/installation/).

### VS Code: Your Code Editor

We use Visual Studio Code as the editor. It is free, it runs on Windows, macOS, and Linux, and the Python ecosystem around it is mature. You can use another editor; the commands in this guide still apply.

**Get VS Code:**

- [VS Code GitHub Repository](https://github.com/microsoft/vscode)
- [Download VS Code](https://code.visualstudio.com/)

**Recommended Python extensions:**

- **Python** (Microsoft) — language support
- **Python Debugger** (Microsoft) — debugging
- **Ruff** (Astral) — linting and formatting, quickly
- **Pylance** — still the common language server / IntelliSense layer; install it if the Python extension does not already pull it in for you

Install them from the Extensions view (`Ctrl+Shift+X`, or `Cmd+Shift+X` on macOS). Search for the names above; do not hunt for mysterious settings keys.

> **A note on tool choices:** The tools in this guide are popular modern defaults, not the only valid stack. Excellent alternatives exist in every category — package managers (Poetry, PDM, pip-tools), linters (Flake8, Pylint, Bandit), formatters (Black, autopep8), type checkers (Pyright, mypy, Pyrefly), test runners (unittest), web frameworks (Django, Flask, Litestar), and AI tools (GitHub Copilot, Cursor, Claude Code, Codex, Grok Build). Learn the *concepts*, then pick what fits your project and team.

## What's Next?

You know what this guide is for, and you can install uv. Next we build the actual foundation: virtual environments — first with the standard library so the idea is real, then with uv, which is how you will work day to day.

[← Back to README](./README.md) | [Next: Virtual Environments →](./02-virtual-environments.md)
