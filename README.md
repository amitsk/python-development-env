# Python Development Environment Setup Guide

A comprehensive guide for students and developers learning to set up a professional Python development environment with modern tools and best practices.

## About This Guide

This tutorial is designed for students and folks learning programming who have some experience writing code. It covers essential software engineering practices and tools that will help you become a more effective Python developer.

**Note:** This is not a Python programming tutorial. If you're looking to learn Python basics, check out these resources:
- [Official Python Tutorial](https://docs.python.org/3/tutorial/)
- [Real Python](https://realpython.com/)
- [Python for Beginners](https://www.python.org/about/gettingstarted/)

## What You'll Learn

This guide covers the essential tools and practices for modern Python development:

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
   - Tools Overview (uv, VSCode)

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
    - Cursor, Antigravity, and Claude Code
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
- Willingness to use the command line/terminal

## Getting Started

Start with [Chapter 1: Introduction](./01-introduction.md) to begin your journey toward professional Python development.

## Contributing

Found an issue or have a suggestion? Feel free to open an issue or submit a pull request.

## License

This guide is released under the [MIT License](./LICENSE). You are free to use, share, and adapt it — just include attribution.
