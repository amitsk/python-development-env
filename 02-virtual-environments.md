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

While Python's built-in `venv` works well, `uv` provides a faster and more modern approach to managing virtual environments and packages.

### Creating a Virtual Environment with uv

```bash
# Create a virtual environment
uv venv

# Or specify a custom name
uv venv myenv

# Or specify a Python version
uv venv --python 3.12
```

### Activating the uv Virtual Environment

Activation works the same as with standard venv:

**On macOS/Linux:**
```bash
source .venv/bin/activate
```

**On Windows:**
```bash
.venv\Scripts\activate
```

Note: uv creates a `.venv` directory by default (with a dot prefix).

### Installing Packages with uv

uv offers several ways to install packages, and it's much faster than pip:

```bash
# Install a package (like pip install)
uv pip install requests

# Install multiple packages
uv pip install requests pandas numpy

# Install from requirements.txt
uv pip install -r requirements.txt

# Install in development mode
uv pip install -e .
```

### Why uv is Faster

uv is written in Rust and uses several optimizations:
- Parallel downloads and installs
- Efficient caching
- Smart dependency resolution
- Pre-built wheel optimization

In practice, uv can be 10-100x faster than pip for many operations.

## Running Python Programs

Once your virtual environment is set up and activated, running Python programs is straightforward:

### Basic Execution

```bash
# Make sure your environment is activated
source .venv/bin/activate  # macOS/Linux
# or
.venv\Scripts\activate  # Windows

# Run your Python script
python my_script.py

# Or run a module
python -m my_module
```

### Using uv run (Without Activation)

One of uv's most convenient features is the ability to run commands in the virtual environment without explicitly activating it:

```bash
# Run a Python script using the project's environment
uv run python my_script.py

# Run a module
uv run python -m pytest

# Run installed command-line tools
uv run black .
uv run ruff check .
```

This is particularly useful in scripts and CI/CD pipelines where you don't want to worry about activation.

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
# Create
uv venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install packages
uv pip install package_name

# Run without activation
uv run python script.py

# Deactivate
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
