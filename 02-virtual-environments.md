# Chapter 2: Python Virtual Environments

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Managing Dependencies →](./03-dependencies.md)

## What are Virtual Environments?

A virtual environment is an isolated Python install for one project. Packages you add there stay there. Your system Python stays out of it.

Picture two projects. Project A needs version 1 of a library; project B needs version 2. One global `site-packages` cannot hold both. Virtual environments give each project its own interpreter, its own `pip`, and its own package directory. That is the whole trick.

## Why bother

Without an environment per project you eventually install a package “just for this script” and break a system tool you forgot depended on the previous version. With one:

- **Isolation** — each project has its own dependencies. Version conflicts become a project problem, not a machine problem.
- **Reproducibility** — `pyproject.toml` plus a lockfile (or, in older projects, `requirements.txt`) is how you hand someone else the same stack.
- **A clean system Python** — OS tools that happen to be written in Python keep their own packages.
- **Easy reset** — delete the environment directory and make a new one. You are not uninstalling packages one by one from `/usr`.
- **Multiple Pythons** — 3.12 on this repo, 3.13 on that one, without a prayer circle around `$PATH`.

Day to day you will not think about this much. `uv init`, `uv add`, `uv run`. The environment is a directory named `.venv`. The rest of this chapter is so you know what that directory *is*.

## Creating Virtual Environments with Python 3

Python ships the `venv` module. You should see it once, even if uv will create environments for you afterward.

### Creating a Virtual Environment

```bash
# Create a virtual environment named 'venv'
python3 -m venv venv

# Or pick a name
python3 -m venv myenv

# Prefer an explicit interpreter when you have more than one
python3.13 -m venv venv
```

That directory contains a Python interpreter, a copy of pip, and an isolated `site-packages` for installs.

### Activating the Virtual Environment

**On macOS/Linux:**

```bash
source venv/bin/activate
```

**On Windows:**

```bash
venv\Scripts\activate
```

Your prompt grows a prefix so you know where `python` and `pip` point:

```bash
(venv) user@machine:~/project$
```

### Using the Virtual Environment

Once activated:

```bash
# Installs land in this environment only
pip install requests pandas

# This python is the environment's python
python script.py

pip list
```

### Deactivating the Virtual Environment

```bash
deactivate
```

`venv` is the concept. uv is the workflow.

## Creating Virtual Environments with uv

uv is the primary tool in this guide. It creates environments quickly and manages the project around them: Python version, dependencies, lockfile, and commands.

### Preferred: start a project with `uv init`

For new work, initialize a project. That gives you `pyproject.toml`, a virtualenv (on first install), and a layout you can actually share.

```bash
# New directory plus scaffold
uv init --python 3.13 myapp
cd myapp

# Already in a directory? Initialize in place
uv init --python 3.13
```

If 3.13 is not on the machine, install it first:

```bash
uv python install 3.13
uv python list
```

Then add packages and sync. You rarely need to activate anything.

```bash
uv add requests          # runtime dependency
uv add --dev pytest ruff # development tools (the `dev` group)
uv sync                  # install from the lockfile into .venv
uv run python main.py    # run in the project environment
```

`uv run` is the daily command. Activation is optional.

### Creating only a virtual environment

Sometimes you want an environment and nothing else:

```bash
# Default name: .venv
uv venv

# Custom name
uv venv myenv

# Specific Python (uv can fetch it if needed)
uv venv --python 3.13
```

### Activating the uv virtual environment

If you want a classic `(venv)` prompt, activation is the same as stdlib `venv`. uv just defaults to a hidden `.venv`.

**On macOS/Linux:**

```bash
source .venv/bin/activate
```

**On Windows:**

```bash
.venv\Scripts\activate
```

Most of the time, skip this and use `uv run`.

### Installing packages with uv

**Project-first (the path this guide uses once `pyproject.toml` exists):**

```bash
uv add requests
uv add --dev ruff pytest
uv remove requests
uv sync
```

**pip-compatible interface** (scripts, one-offs, or an existing `requirements.txt`):

```bash
uv pip install requests
uv pip install -r requirements.txt
uv pip install -e .
```

That interface is useful. It is not how you should manage a project you intend to keep.

### Why uv is faster

uv is written in Rust and spends its energy on parallel downloads, a serious cache, and a resolver that does not take a coffee break. In practice it is often many times faster than pip for the same install. Speed is a nice bonus. The project workflow is the reason to use it.

## Running Python Programs

### With activation

```bash
source .venv/bin/activate  # macOS/Linux
# or: .venv\Scripts\activate  # Windows

python my_script.py
python -m my_module
```

### Using `uv run` (no activation required)

```bash
# Script in the project environment
uv run python my_script.py

# Tools declared in the project
uv run pytest
uv run ruff check .

# One-off tool, not added as a dependency
uvx ruff check .
```

`uv run` is the right default locally and in CI. Nobody has to remember `source .venv/bin/activate`, and CI images will not “forget” to activate.

## Best Practices

Always use a virtual environment; never install project dependencies onto the system interpreter. One environment per project. Put the environment directory in `.gitignore` — it is rebuildable, and it is huge. Record what belongs in it with `pyproject.toml` (this guide) or `requirements.txt` (older projects). Stick to a boring name: `.venv` with uv, `venv` if you are using stdlib `venv`.

### Example .gitignore Entry

```gitignore
# Virtual environments
venv/
.venv/
env/
ENV/
```

`uv init` writes a `.gitignore` that already covers `.venv`. If you created the environment by hand, add the lines yourself. Chapter 4 goes deeper on Git; this is the one ignore rule you want before the first commit.

## Quick Reference

### Standard venv

```bash
# Create
python3 -m venv venv
python3.13 -m venv venv

# Activate (macOS/Linux)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Install packages
pip install package_name

# Deactivate
deactivate
```

### uv

```bash
# Python, if you need it
uv python install 3.13

# New project — this is the daily path
uv init --python 3.13 myapp && cd myapp
uv add package_name
uv add --dev ruff pytest
uv sync
uv run python main.py

# Or only a venv
uv venv --python 3.13
source .venv/bin/activate   # optional on macOS/Linux
.venv\Scripts\activate      # optional on Windows

# Run without activation
uv run python script.py
uv run pytest
uvx ruff check .

# Deactivate (only if you activated)
deactivate
```

## Troubleshooting

### Command Not Found

If `uv` or `python3.13` is “not found” after a fresh install, your `PATH` is stale or incomplete. Close and reopen the terminal. Recheck the installer output for the directory it added. As a last resort, call the binary by full path.

### Permission Errors

Do not use `sudo` to create a virtual environment. That is how you decorate a local directory with root-owned files and then spend the afternoon undoing it. You need write permission in the project directory and a working Python. That is all.

### Wrong Python Version

Name the interpreter:

```bash
python3.13 -m venv venv

# Or with uv — install, then create
uv python install 3.13
uv venv --python 3.13
```

`uv python list` shows what uv can see.

## What's Next?

You can create an environment, and you know `uv run` is how you enter it without ceremony. Next: the packages that go inside it, declared in `pyproject.toml` and locked so the next machine gets the same tree.

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Managing Dependencies →](./03-dependencies.md)
