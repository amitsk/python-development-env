
* Generate a tutorial on setting up a Python development environment using 
the details in this file. The audience for the tutorial are students or folks learning programming with some experience in writing code. Create this as a Github project with a README that has a table of contents and individual chapters . Create navigation links in each document.
* Tutorial on setting up a Python Development environment
** Topics 
*** Introduction
    **** Background. Not a Python tutorial. Provide a link to popular python tutorials including the one from Python.org
    **** Explain basic software engineering best practices like Version control, Linting, Code formatting, Unit Testing, CI/CD
    **** Set the contex about uv. Mention that uv will be used. Links to uv and uv installation for Mac+Windows+Linux
    **** Explain how VSCode will be used. Point to VSCode github link https://github.com/microsoft/vscode.
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
*** CI/CD
    **** Explain what is CI/CD. Benefits of CI/CD. 
    **** Very short explainer about github actions. Point to github actions github link https://github.com/features/actions

*** LLMs
    **** Explain what is LLMs. Benefits of LLMs and using these in python projects
    **** Very short explainer about Cursor. Point to Cursor github link https://github.com/cursor-ai/cursor
    **** Short explainer about Antigravity. Point to Antigravity github link https://github.com/antigravity-ai/antigravity
    **** Short explainer about Claude Code. Point to Claude Code github link https://github.com/anthropic/claude-code

*** Stitch it all together.
    **** Stitch it all together.
    **** Explain how to create a new project using uv
    **** Short explainer about cookiecutter. Point to cookiecutter github link https://github.com/amitsk/cookiecutter-python-library
    **** Walk through the cookiecutter workflow, use the cookiecutter-python-library to create a new project and run tests, linting, formatting etc.
    **** Explain how to create a github repository and push the code to github.
    **** Explain how to run the tests, linting, formatting etc. using github actions.