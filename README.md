# Python Development Environment Setup Guide

You already know how to write Python. This guide is about the rest of the job: isolated environments, pinned dependencies, Git that other people can follow, and the tools that keep a project from rotting the first time it leaves your laptop.

## About This Guide

This is for students and working developers who can write code and want the professional workflow around it. We cover the practices and tools that make Python projects reproducible, reviewable, and less fragile.

**This is not a Python programming tutorial.** If you need language basics first, start here:

- [Official Python Tutorial](https://docs.python.org/3/tutorial/)
- [Real Python](https://realpython.com/)
- [Python for Beginners](https://www.python.org/about/gettingstarted/)
- [Automate the Boring Stuff with Python](https://automatetheboringstuff.com/)

## What You'll Learn

Modern Python development, in the order you actually need it:

- Setting up virtual environments with modern tools
- Managing project dependencies
- Version control with Git
- Code quality through linting and formatting
- Testing your code effectively
- Logging with loguru and picologging
- Continuous Integration and Deployment (CI/CD)
- Using AI-powered development tools
- Working with databases using SQLModel and SQLAlchemy
- Building REST APIs with FastAPI
- Deploying serverless functions with AWS Lambda and SAM
- Building CLI tools with Typer and Rich
- Putting it all together in a real project

## Table of Contents

1. [Introduction](./01-introduction.md)
   - Background and Context
   - Software Engineering Best Practices
   - Tools Overview (uv, VS Code)

2. [Python Virtual Environments](./02-virtual-environments.md)
   - Understanding Virtual Environments
   - Creating Virtual Environments
   - Working with uv

3. [Managing Dependencies](./03-dependencies.md)
   - What are Dependencies?
   - Using pip for Package Management
   - Managing Dependencies with uv
   - Working with pyproject.toml

4. [Version Control](./04-version-control.md)
   - What is Version Control?
   - Getting Started with Git
   - Best Practices

5. [Linting](./05-linting.md)
   - What is Linting?
   - Overview of Python Linters
   - Deep Dive into Ruff

6. [Code Formatting](./06-code-formatting.md)
   - Importance of Code Formatting
   - Formatting Tools Overview
   - Formatting with Ruff

7. [Unit Testing](./07-unit-testing.md)
   - Understanding Unit Testing
   - Introduction to pytest
   - Writing and Running Tests

8. [Static Type Checking](./08-static-type-checking.md)
   - What is Static Type Checking?
   - Type Checking Tools
   - Using mypy, ty, and Pyrefly

9. [Logging](./09-logging.md)
   - print() vs proper logging
   - Python logging stdlib, loguru, and picologging
   - Structured logging and log rotation

10. [CI/CD](./10-ci-cd.md)
    - Continuous Integration and Deployment
    - GitHub Actions
    - Automating Your Workflow

11. [LLMs and AI-Powered Development](./11-llms.md)
    - AI in Python Development
    - GitHub Copilot, Cursor, Claude Code, Codex, and Grok Build
    - Best Practices with AI Tools

12. [Database Libraries](./12-database-libraries.md)
    - DBAPI, SQLAlchemy, and SQLModel compared
    - Type-safe database access with SQLModel
    - Migrations with Alembic

13. [REST APIs with FastAPI](./13-fastapi.md)
    - Building type-safe APIs with FastAPI
    - Pydantic models and automatic validation
    - Automatic OpenAPI documentation

14. [AWS Lambda with Python](./14-aws-lambda.md)
    - Serverless Python with AWS Lambda
    - AWS SAM for local development and deployment
    - AWS Lambda Powertools for production

15. [CLI Libraries](./15-cli-libraries.md)
    - Building CLI tools with Typer
    - Beautiful terminal output with Rich
    - Combining Typer and Rich

16. [Putting It All Together](./16-putting-it-together.md)
    - Creating a Complete Project
    - Using Cookiecutter Templates
    - Setting Up CI/CD Pipeline
    - Publishing to GitHub

## Prerequisites

- Basic understanding of Python programming
- A computer running Windows, macOS, or Linux
- Willingness to use the command line. Clicking around is optional; typing `uv run` is not.

## Core vs optional tracks

**Core (recommended first):** Chapters 1–10 and 16 — environments, dependencies, Git, Ruff, pytest, types, logging, CI/CD, then a full project setup.

**Optional modules:** Chapters 11–15 — AI coding tools, databases, FastAPI, AWS Lambda, and CLI libraries. Read them when you need those skills, not because they look impressive in a table of contents.

## Getting Started

Start with [Chapter 1: Introduction](./01-introduction.md). As of this writing, Python 3.15 is the current stable release, 3.14 is the previous one, and 3.13 is the practical floor. Examples in the core chapters use 3.14 — it has had a year of wheels, which is more useful than being newest — and declare `requires-python = ">=3.13"`. Do not freeze a new project on 3.11 because a blog post from 2024 told you it was current.

**On Python 3.15:** it shipped on October 9, 2026, and the 3.15 series is already at 3.15.2. It brings lazy imports, `frozendict` and `sentinel` built-ins, UTF-8 as the default text encoding, unpacking in comprehensions, and a stable ABI for free-threaded builds. Add 3.15 to your CI matrix now; switch your default interpreter once your dependencies and tests pass on it.

**Last reviewed:** November 2026 (Python 3.15 current stable; 3.14 in examples; `requires-python >= 3.13`; uv project workflow).

## Contributing

Found an issue or have a suggestion? Open an issue or submit a pull request. Typos in tutorials have a long half-life; we would rather hear about them.

## License

This guide is released under the [MIT License](./LICENSE). You are free to use, share, and adapt it — just include attribution.
