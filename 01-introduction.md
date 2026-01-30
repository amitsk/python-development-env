# Chapter 1: Introduction

[← Back to README](./README.md) | [Next: Virtual Environments →](./02-virtual-environments.md)

## Background

Welcome to this comprehensive guide on setting up a professional Python development environment. Before we dive in, it's important to clarify what this guide is and isn't:

**This is NOT a Python programming tutorial.** We assume you already have some familiarity with Python basics. If you need to learn or refresh your Python skills, here are some excellent resources:

- [Official Python Tutorial](https://docs.python.org/3/tutorial/) - The authoritative guide from Python.org
- [Real Python](https://realpython.com/) - Comprehensive tutorials and articles
- [Python for Beginners](https://www.python.org/about/gettingstarted/) - Getting started resources
- [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/) - Practical Python for beginners

**This IS a guide about professional development practices.** We'll cover the tools and workflows that professional developers use to write better, more maintainable code and collaborate effectively with others.

## Software Engineering Best Practices

Before we get into specific tools, let's understand the key practices that make software development professional and sustainable:

### Version Control

Version control systems track changes to your code over time. They allow you to:
- See what changed, when, and why
- Collaborate with others without conflicts
- Revert to previous versions if something breaks
- Work on multiple features simultaneously

We'll cover Git in detail in [Chapter 3](./03-version-control.md).

### Linting

Linting is the process of automatically checking your code for potential errors, bugs, and style issues. Benefits include:
- Catching bugs before they reach production
- Enforcing coding standards across a team
- Learning best practices through automated feedback
- Reducing code review time

We'll explore linting tools in [Chapter 4](./04-linting.md).

### Code Formatting

Code formatting ensures your code follows consistent style guidelines. This is important because:
- Consistent code is easier to read and understand
- It eliminates debates about style in code reviews
- It makes collaboration smoother
- Many tools can format code automatically

We'll discuss formatting in [Chapter 5](./05-code-formatting.md).

### Unit Testing

Unit testing involves writing code to test your code. Benefits include:
- Confidence that your code works as expected
- Ability to refactor without fear of breaking things
- Documentation of how code should behave
- Faster debugging when issues arise

Testing is covered in [Chapter 6](./06-unit-testing.md).

### CI/CD (Continuous Integration/Continuous Deployment)

CI/CD automates the process of testing and deploying code:
- Automatically run tests on every code change
- Catch integration issues early
- Deploy code reliably and consistently
- Reduce manual work and human error

We'll set up CI/CD in [Chapter 8](./08-ci-cd.md).

## Tools We'll Use

### uv: Modern Python Package Management

Throughout this tutorial, we'll use [uv](https://github.com/astral-sh/uv), a fast, modern Python package and project manager written in Rust. uv provides:

- Extremely fast package installation
- Built-in virtual environment management
- Tool execution without installation
- Drop-in replacement for pip and pip-tools

#### Installing uv

**macOS and Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Windows:**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Using pip:**
```bash
pip install uv
```

For more installation options, visit the [uv documentation](https://docs.astral.sh/uv/getting-started/installation/).

### VSCode: Your Code Editor

We'll use Visual Studio Code (VSCode) as our primary code editor. VSCode is:
- Free and open-source
- Highly extensible with thousands of extensions
- Well-supported with regular updates
- Available for Windows, macOS, and Linux

**Get VSCode:**
- [VSCode GitHub Repository](https://github.com/microsoft/vscode)
- [Download VSCode](https://code.visualstudio.com/)

**Recommended Python Extensions:**
- Python (Microsoft) - Python language support
- Pylance - Fast, feature-rich language support
- Python Debugger - Debugging support
- Ruff - Fast linting and formatting

You can install VSCode extensions from the Extensions view (Ctrl+Shift+X or Cmd+Shift+X on macOS).

## What's Next?

Now that you understand the context and have an overview of the tools we'll use, let's start with the foundation of Python development: virtual environments.

[← Back to README](./README.md) | [Next: Virtual Environments →](./02-virtual-environments.md)
