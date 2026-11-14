
* Generate a tutorial on setting up a Python development environment using 
the details in this file. The audience for the tutorial are CS students, new developers, or folks learning programming with some experience in writing code. Keep the tone lively, casual, and fun rather than textbooky. Include Mermaid diagrams, custom visual illustrations in `assets/`, and relatable examples across web, AI, and robotics/physical computing. Create this as a Github project with a README that has a table of contents and individual chapters. Create navigation links in each document.
* Tutorial on setting up a Python Development environment
** Topics 
*** Introduction
    **** Background. Not a Python tutorial. Provide a link to popular python tutorials including the one from Python.org
    **** Showcase where Python powers the real world: Robotics (ROS 2 / rclpy, MuJoCo / Isaac Lab simulation, OpenCV, MicroPython / CircuitPython), AI/LLMs (training + how LLMs use Python as their own scripting language), Web APIs, and CLI tools.
    **** Explain basic software engineering best practices like Version control, Linting, Code formatting, Unit Testing, CI/CD with Mermaid diagrams.
    **** Set the context about uv. Mention that uv will be used. Links to uv and uv installation for Mac+Windows+Linux
    **** Explain how VSCode will be used. Point to VSCode github link https://github.com/microsoft/vscode.
    **** Add a note that the tools mentioned are some of the popular modern choices and that there are many other excellent tools available (Poetry, PDM, Conda/Pixi, Flake8, Pylint, Black, Pyright, Flask, Django, Litestar, etc.) - the ones covered provide a solid foundation but users should pick what works best for their project.
*** Python Virtual environments
    **** Explain what are Virtual envs. Benefits of virtual environments. 
    **** Explain how to create venv in Python3 
    **** Explain how to create a virtual environment using uv. Explain how to use the venv, running python programs
*** Version control
    **** Explain what is version control. Benefits of version control. 
    **** Very short explainer about git. Point to git github link https://github.com
*** Linting
    **** Explain what is linting. Benefits of linting. 
    ****  Very Short explainer about tools like flake8, ruff, pyright, mypy etc. Point to flake8 github link https://github.com/PyCQA/flake8
    **** Detailed explainer on ruff. Point to ruff github link https://github.com/astral-sh/ruff and running ruff on a sample project using uv. Link to the uv docs for ruff.
*** Code formatting
    **** Explain what is code formatting. Benefits of code formatting. 
    **** Very short explainer about black. Point to black github link https://github.com/psf/black
    **** Detailed explainer on code formatting using ruff. Point to ruff github link https://github.com/astral-sh/ruff and running ruff on a sample project using uv. Link to the uv docs for ruff.
*** Unit Testing
    **** Explain what is unit testing. Benefits of unit testing. 
    **** Very short explainer about pytest. Point to pytest github link https://github.com/pytest-dev/pytest
    **** Detailed explainer on unit testing using pytest. Point to pytest github link https://github.com/pytest-dev/pytest and running pytest on a sample project using uv. Link to the uv docs for pytest.
*** Static Type Checking
    **** Explain what is static type checking. Benefits of static type checking. 
    **** Very short explainer about mypy. Point to mypy github link https://github.com/python/mypy
    **** Detailed explainer on static type checking using ty.
    **** Detailed explainer on pyrefly https://pyrefly.org/
*** Logging
    **** Why print() is not enough for production
    **** Python stdlib logging: basicConfig, named loggers, handlers, formatters, log levels
    **** Loguru - https://github.com/Delgan/loguru - ergonomic logging with zero boilerplate
        ***** Installation: uv add loguru
        ***** Zero-config default sink
        ***** Configuring sinks: logger.remove(), logger.add()
        ***** File rotation, retention, compression
        ***** Structured context with logger.bind()
        ***** Exception tracebacks with @logger.catch and logger.exception()
        ***** Intercepting stdlib logging with InterceptHandler
        ***** JSON via serialize=True
    **** Picologging - https://github.com/microsoft/picologging - C reimplementation of the stdlib logging API (still beta)
        ***** Installation: uv add picologging
        ***** Same API as stdlib logging: import picologging as logging
        ***** Still beta. The latest release (0.9.3) ships wheels only through CPython 3.12 — do not sell it as a risk-free drop-in on 3.13 or newer
        ***** When to use: already using stdlib logging, on a supported Python, and logging is on a hot path
    **** Comparison table: stdlib vs loguru vs picologging
    **** Recommendations: Loguru for new projects; stdlib if you want zero deps; picologging only in the narrow case above
    **** python-json-logger is fine to mention for stdlib JSON logs; do not invent APIs. Prefer documented imports.

*** CI/CD
    **** Explain what is CI/CD. Benefits of CI/CD.
    **** Very short explainer about github actions. Point to github actions github link https://github.com/features/actions
    **** Workflow examples with uv: actions/checkout@v7 (v4 is outdated; Node 20 runners going away), astral-sh/setup-uv@v10.3.3 (setup-uv dropped moving major tags; pin a full version), enable-cache: true
    **** Python matrix: ['3.13', '3.14', '3.15']. Default PYTHON_VERSION: '3.14'
    **** Steps: uv python install, uv sync, ruff check, ruff format --check, ty check, pytest --cov
    **** Coverage: codecov/codecov-action@v7
    **** Publish: Trusted Publishing with pypa/gh-action-pypi-publish@release/v1
    **** Pinning third-party actions to commit SHAs is the extra-careful option; examples may use the version tags above so the tutorial stays readable
    **** act for local testing, secrets, required status checks, badges
    **** Chapter 10 previous-chapter nav must be Logging (09-logging.md), not Static Type Checking

*** LLMs
    **** Explain what is LLMs. Benefits of LLMs and using these in python projects
    **** Explain how LLMs use Python as their own scripting language for complex tasks (Code Interpreter sandboxes, deterministic math/data analysis, programmatic tool use, and "Code as Policies" in robotics)
    **** Explain Spec-Driven Development (SDD) and the Elephant-Goldfish mental model (https://egm.dev/ — encyclopedic pre-trained knowledge like an elephant, zero persistent session/project memory like a goldfish—solved by keeping written specs and CLAUDE.md/AGENTS.md in the repo)
    **** Concise one-sentence overview with links for AI development tools: Claude Code, GitHub Copilot & Copilot CLI, Cursor, OpenAI Codex CLI, Google Antigravity (agy), OpenCode, and Grok Build
    **** Detailed walkthrough and examples for Claude Code (native install: curl -fsSL https://claude.ai/install.sh | bash, CLAUDE.md, writing a feature spec in specs/, implementing in a fresh session, and verification loop) tied into SDD and the Elephant-Goldfish model
    **** Tools change fast; send readers to official docs for pricing/install
    **** Review generated code, run ruff + ty + pytest, never paste secrets
    **** This is product coverage, not authorship. Do not add "Generated with …" or similar.

*** Database Libraries
    **** Landscape: DBAPI (PEP 249, sqlite3), SQLAlchemy Core, SQLAlchemy ORM, SQLModel
    **** DBAPI - PEP 249 standard, sqlite3 example, parameterized queries, context managers
    **** SQLAlchemy - https://www.sqlalchemy.org - Core (SQL Expression Language) and ORM (DeclarativeBase, Session, relationships)
    **** SQLModel - https://sqlmodel.tiangolo.com - built on SQLAlchemy + Pydantic by FastAPI creator. One class = DB table + validation schema. Full CRUD example.
    **** Alembic - migrations tool for SQLAlchemy: alembic init, autogenerate, upgrade head
    **** Comparison table: DBAPI vs SQLAlchemy Core vs SQLAlchemy ORM vs SQLModel
    **** When to choose each

*** REST APIs with FastAPI
    **** FastAPI - https://fastapi.tiangolo.com - modern, fast, type-safe Python web framework built on Starlette + Pydantic
    **** Installation: uv add "fastapi[standard]"
    **** Path parameters, query parameters, request body with Pydantic models
    **** Complete CRUD REST API example (heroes resource)
    **** Automatic OpenAPI docs at /docs and /redoc
    **** Dependency injection with Depends()
    **** Integration with SQLModel for database-backed APIs
    **** Testing with pytest + FastAPI TestClient (httpx still used under the hood; FastAPI docs still say install httpx)
    **** Running in production: uvicorn workers, Gunicorn
    **** HTTP client libraries: Requests (https://requests.readthedocs.io/) and HTTPX with HTTP/2 support (https://www.python-httpx.org/, uv add "httpx[http2]")
    **** Alternatives: Flask, Django REST Framework, Litestar

*** AWS Lambda with Python
    **** What is AWS Lambda - serverless, event-driven, automatic scaling, pay per invocation
    **** Python Lambda handler structure: def handler(event, context) -> dict
    **** AWS Lambda Powertools - https://docs.powertools.aws.dev/lambda/python/ - structured logging, tracing, metrics, typed event parsing
    **** API Gateway handler example using Powertools
    **** SQS handler with partial batch failure handling
    **** AWS SAM (Serverless Application Model) - https://aws.amazon.com/serverless/sam/
        ***** Installation: AWS CLI + SAM CLI
        ***** template.yaml for Python Lambda, API Gateway, DynamoDB
        ***** sam build, sam local invoke, sam local start-api, sam deploy
    **** Packaging Python dependencies
    **** Testing Lambda handlers with pytest and moto
    **** DynamoDB with boto3 and types-boto3 type hints (preferred name; boto3-stubs is the older extra name)
    **** CI/CD with GitHub Actions: OIDC authentication, sam build, sam deploy on main

*** CLI Libraries
    **** Why build CLI tools in Python
    **** Typer - https://typer.tiangolo.com - modern CLI using type hints, built on Click
        ***** Installation: uv add typer
        ***** Single command app with typer.run()
        ***** Multi-command app with @app.command()
        ***** Arguments vs Options from type annotations
        ***** Complete tasks CLI example
        ***** Subcommands with app.add_typer()
        ***** Auto-generated --help from type hints and docstrings
    **** Rich - https://rich.readthedocs.io - beautiful terminal output
        ***** Installation: uv add rich
        ***** Console, markup, tables, progress bars, panels, syntax highlighting
        ***** Logging integration with RichHandler
    **** Typer + Rich together - complete mkproject scaffolding CLI example
    **** Packaging CLI as a tool: [project.scripts] in pyproject.toml, uvx

*** Stitch it all together.
    **** Stitch it all together.
    **** Include a copy-paste AI Agent Bootstrap Prompt to scaffold and verify the entire project in one step (uv, Ruff, ty, pytest, Makefile, pre-commit, CLAUDE.md, specs/, and GitHub Actions CI)
    **** Explain how to create a new project using uv
    **** Short explainer about cookiecutter. Point to cookiecutter github link https://github.com/amitsk/cookiecutter-python-library
    **** Walk through the cookiecutter workflow, use the cookiecutter-python-library to create a new project and run tests, linting, formatting etc.
    **** Explain how to create a github repository and push the code to github.
    **** Explain how to run the tests, linting, formatting etc. using github actions.
