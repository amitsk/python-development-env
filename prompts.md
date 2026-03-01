
* Generate a tutorial on setting up a Python development environment using 
the details in this file. The audience for the tutorial are students or folks learning programming with some experience in writing code. Create this as a Github project with a README that has a table of contents and individual chapters . Create navigation links in each document.
* Tutorial on setting up a Python Development environment
** Topics 
*** Introduction
    **** Background. Not a Python tutorial. Provide a link to popular python tutorials including the one from Python.org
    **** Explain basic software engineering best practices like Version control, Linting, Code formatting, Unit Testing, CI/CD
    **** Set the contex about uv. Mention that uv will be used. Links to uv and uv installation for Mac+Windows+Linux
    **** Explain how VSCode will be used. Point to VSCode github link https://github.com/microsoft/vscode.
    **** Add a note that the tools mentioned are some of the popular modern choices and that there are many other excellent tools available (Poetry, PDM, Flake8, Pylint, Black, Pyright, Flask, Django, Litestar, etc.) - the ones covered provide a solid foundation but users should pick what works best for their project.
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
    **** Detailed explainer on ruff. Point to ruff github link https://github.com/charliermarsh/ruff and running ruff on a sample project using uv. Link to the uv docs for ruff.
*** Code formatting
    **** Explain what is code formatting. Benefits of code formatting. 
    **** Very short explainer about black. Point to black github link https://github.com/psf/black
    **** Detailed explainer on code formatting using ruff. Point to ruff github link https://github.com/charliermarsh/ruff and running ruff on a sample project using uv. Link to the uv docs for ruff.
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
    **** Picologging - https://github.com/microsoft/picologging - drop-in C replacement for stdlib logging
        ***** Installation: uv add picologging
        ***** Same API as stdlib logging: import picologging as logging
        ***** When to use: existing codebases, high-throughput services
    **** Comparison table: stdlib vs loguru vs picologging
    **** Recommendations

*** CI/CD
    **** Explain what is CI/CD. Benefits of CI/CD.
    **** Very short explainer about github actions. Point to github actions github link https://github.com/features/actions

*** LLMs
    **** Explain what is LLMs. Benefits of LLMs and using these in python projects
    **** Very short explainer about Cursor. Point to Cursor github link https://github.com/cursor-ai/cursor
    **** Short explainer about Antigravity. Point to Antigravity github link https://github.com/antigravity-ai/antigravity
    **** Short explainer about Claude Code. Point to Claude Code github link https://github.com/anthropic/claude-code

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
    **** Installation: uv add fastapi uvicorn[standard]
    **** Path parameters, query parameters, request body with Pydantic models
    **** Complete CRUD REST API example (heroes resource)
    **** Automatic OpenAPI docs at /docs and /redoc
    **** Dependency injection with Depends()
    **** Integration with SQLModel for database-backed APIs
    **** Testing with pytest + httpx TestClient
    **** Running in production: uvicorn workers, Gunicorn
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
    **** DynamoDB with boto3 and boto3-stubs type hints
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
    **** Explain how to create a new project using uv
    **** Short explainer about cookiecutter. Point to cookiecutter github link https://github.com/amitsk/cookiecutter-python-library
    **** Walk through the cookiecutter workflow, use the cookiecutter-python-library to create a new project and run tests, linting, formatting etc.
    **** Explain how to create a github repository and push the code to github.
    **** Explain how to run the tests, linting, formatting etc. using github actions.