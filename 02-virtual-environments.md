# Chapter 2: Python Virtual Environments

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Managing Dependencies →](./03-dependencies.md)

## What are Virtual Environments?

A virtual environment is an isolated Python environment that allows you to install packages for a specific project without affecting other projects or your system Python installation.

Think of it like this: imagine you're working on two different Python projects. Project A needs version 1.0 of a library, but Project B needs version 2.0 of the same library. Without virtual environments, you'd be stuck - you can only have one version installed globally. Virtual environments solve this problem by creating separate, isolated spaces for each project.

## Benefits of Virtual Environments

### 1. Dependency Isolation
Each project can have its own dependencies, regardless of what dependencies other projects have. This prevents version conflicts and "it works on my machine" problems.

### 2. Reproducibility
You can easily share your exact environment with others by sharing a requirements file. This ensures everyone working on the project has the same package versions.

### 3. System Protection
Installing packages in a virtual environment keeps your system Python clean and prevents accidentally breaking system tools that depend on specific Python packages.

### 4. Easy Cleanup
If something goes wrong or you want to start fresh, you can simply delete the virtual environment and create a new one. No need to uninstall packages one by one.

### 5. Multiple Python Versions
You can work with different Python versions for different projects without conflicts.

## Creating Virtual Environments with Python 3

Python 3 includes the `venv` module for creating virtual environments. Here's how to use it:

### Creating a Virtual Environment

```bash
# Create a virtual environment named 'venv'
python3 -m venv venv

# Or use a different name
python3 -m venv myenv
```

This creates a directory (e.g., `venv/`) containing:
- A copy of the Python interpreter
- The pip package manager
- An isolated site-packages directory for installing packages

### Activating the Virtual Environment

**On macOS/Linux:**
```bash
source venv/bin/activate
```

**On Windows:**
```bash
venv\Scripts\activate
```

When activated, you'll see the environment name in your terminal prompt:
```bash
(venv) user@machine:~/project$
```

### Using the Virtual Environment

Once activated:
```bash
# Install packages (they'll only install in this environment)
pip install requests pandas

# Run Python (uses the environment's Python)
python script.py

# Check installed packages
pip list
```

### Deactivating the Virtual Environment

When you're done working:
```bash
deactivate
```

## Creating Virtual Environments with uv

While Python's built-in `venv` works well, **uv** is the primary tool in this guide: it creates environments quickly and manages projects end-to-end.

### Preferred: start a project with uv

For new work, initialize a project (creates `pyproject.toml`, a virtualenv, and a modern layout):

```bash
# Create a new project directory and scaffold
uv init myapp
cd myapp

# Or initialize inside the current directory
uv init
```

Then add packages and sync:

```bash
uv add requests          # runtime dependency
uv add --dev pytest ruff # development tools
uv sync                  # install from lockfile into .venv
uv run python main.py    # run without manual activation
```

### Creating only a virtual environment

```bash
# Create a virtual environment (default name: .venv)
uv venv

# Custom name
uv venv myenv

# Specific Python version (uv can download it if needed)
uv venv --python 3.12
```

### Activating the uv virtual environment

Activation works like standard venv when you want a classic shell prompt:

**On macOS/Linux:**
```bash
source .venv/bin/activate
```

**On Windows:**
```bash
.venv\Scripts\activate
```

Note: uv creates a `.venv` directory by default (dot prefix). In day-to-day work you can often skip activation and use `uv run` instead.

### Installing packages with uv

**Project-first (recommended once you have `pyproject.toml`):**

```bash
uv add requests
uv add --dev ruff pytest
uv remove requests
uv sync
```

**pip-compatible interface** (handy for scripts or existing `requirements.txt`):

```bash
uv pip install requests
uv pip install -r requirements.txt
uv pip install -e .
```

### Why uv is faster

uv is written in Rust and optimizes for parallel downloads, caching, and fast resolution. In practice it is often many times faster than pip for common operations.

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
# Run a script in the project environment
uv run python my_script.py

# Run tools declared in the project
uv run pytest
uv run ruff check .

# One-off tool without adding it permanently
uvx ruff check .
```

`uv run` is ideal for local scripts and CI: no need to remember `source .venv/bin/activate`.

## Best Practices

1. **Always use virtual environments** - Never install project dependencies globally
2. **One environment per project** - Keep projects isolated
3. **Add to .gitignore** - Don't commit virtual environment directories to version control
4. **Document dependencies** - Maintain a requirements.txt or pyproject.toml file
5. **Use consistent names** - Stick to common names like `venv` or `.venv`

### Example .gitignore Entry

```gitignore
# Virtual environments
venv/
.venv/
env/
ENV/
```

## Quick Reference

### Standard venv

```bash
# Create
python3 -m venv venv

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
# New project
uv init myapp && cd myapp
uv add package_name
uv add --dev ruff pytest
uv sync

# Or only a venv
uv venv
source .venv/bin/activate   # optional on macOS/Linux
.venv\Scripts\activate      # optional on Windows

# Run without activation
uv run python script.py
uv run pytest

# Deactivate (if you activated)
deactivate
```

## Troubleshooting

### Command Not Found

If you get "command not found" errors after installation, your PATH may not be set up correctly. Try:
- Closing and reopening your terminal
- Checking the installation instructions for your OS
- Using the full path to the Python executable

### Permission Errors

If you get permission errors when creating a virtual environment:
- Don't use `sudo` - this defeats the purpose of virtual environments
- Make sure you have write permissions in the directory
- Check that Python is installed correctly

### Wrong Python Version

If the virtual environment uses the wrong Python version:
```bash
# Specify the Python version explicitly
python3.12 -m venv venv

# Or with uv
uv venv --python 3.12
```

## What's Next?

Now that you understand how to create and manage virtual environments, let's explore how to manage dependencies - the external packages your project needs to function.

[← Previous: Introduction](./01-introduction.md) | [Back to README](./README.md) | [Next: Managing Dependencies →](./03-dependencies.md)
