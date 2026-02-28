# Chapter 14: CLI Libraries

[← Previous: AWS Lambda](./13-aws-lambda.md) | [Back to README](./README.md) | [Next: Putting It All Together →](./15-putting-it-together.md)

## Why Build CLI Tools in Python?

Python has always been an excellent language for automation and tooling. While web APIs and GUIs get a lot of attention, command-line tools are among the most practical and immediately useful programs a developer can write. A well-built CLI tool can save hours of repetitive work, and Python's ecosystem makes building them surprisingly straightforward.

**Common use cases for Python CLI tools:**

- **Data processing scripts** — transform CSV files, generate reports, migrate data between systems
- **DevOps tooling** — automate deployments, manage cloud resources, run health checks
- **Project automation** — scaffold new projects, run linters and formatters, manage configuration
- **Developer tools** — database inspection utilities, log analyzers, environment setup scripts

### The Standard Library Starting Point

Python's standard library includes `argparse` for building CLIs. It works, and it's always available without installing anything:

```python
import argparse

parser = argparse.ArgumentParser(description="Greet someone")
parser.add_argument("name", help="Name of the person to greet")
parser.add_argument("--count", type=int, default=1, help="How many times to greet")
args = parser.parse_args()

for _ in range(args.count):
    print(f"Hello, {args.name}!")
```

`argparse` gets the job done, but it is verbose. For a tool with multiple subcommands, you end up writing a significant amount of boilerplate just to wire up parsers, validate types, and generate help text.

### Modern Alternatives

Two libraries have become the standard for Python CLI development:

- **Typer** — modern, type-hint-driven, minimal boilerplate
- **Click** — the battle-tested foundation that Typer builds on

This chapter focuses primarily on Typer, since it represents the current best practice and integrates naturally with the type-annotated Python you've already been writing. We'll also cover **Rich**, a companion library for producing beautiful terminal output — tables, progress bars, syntax highlighting, and more.

---

## Typer — Modern CLI with Type Hints

[Typer](https://typer.tiangolo.com) ([GitHub](https://github.com/tiangolo/typer)) was created by Sebastián Ramírez, the same developer behind FastAPI. The design philosophy is identical: use Python's type annotations to eliminate boilerplate and provide an excellent developer experience.

Where FastAPI reads type annotations to generate API schemas and validation, Typer reads them to generate CLI argument parsers, type validation, and help text. The result is a CLI framework where you write ordinary Python functions and get a fully-featured command-line interface for free.

Typer is built on top of Click, so it inherits Click's reliability and feature set while adding a much cleaner API.

### Installation

```bash
uv add typer
```

For rich terminal output in help text (recommended):

```bash
uv add "typer[all]"
```

The `[all]` extra installs Rich as well, which Typer uses to render more attractive help pages.

---

### Your First Typer App

Here is the simplest possible Typer application:

```python
# hello.py
import typer


def main(name: str) -> None:
    """Greet someone by name."""
    print(f"Hello, {name}!")


if __name__ == "__main__":
    typer.run(main)
```

Run it:

```bash
python hello.py Alice
# Hello, Alice!

python hello.py --help
# Usage: hello.py [OPTIONS] NAME
#
#  Greet someone by name.
#
# Arguments:
#   NAME  [required]
#
# Options:
#   --help  Show this message and exit.
```

A few things to notice:

- The `name: str` parameter becomes a required positional argument automatically
- The `--help` flag is generated with no extra work
- The docstring becomes the command description

`typer.run(main)` is a convenience wrapper for single-command applications. For tools with multiple commands, you use an `app` object instead:

```python
# tasks.py
import typer

app = typer.Typer()


@app.command()
def hello(name: str) -> None:
    """Greet someone."""
    print(f"Hello, {name}!")


@app.command()
def goodbye(name: str) -> None:
    """Say goodbye."""
    print(f"Goodbye, {name}!")


if __name__ == "__main__":
    app()
```

```bash
python tasks.py --help
# Usage: tasks.py [OPTIONS] COMMAND [ARGS]...
#
# Options:
#   --help  Show this message and exit.
#
# Commands:
#   goodbye  Say goodbye.
#   hello    Greet someone.

python tasks.py hello Alice
# Hello, Alice!
```

---

### Arguments vs Options

Typer distinguishes between two kinds of CLI inputs, and the distinction comes directly from your type annotations:

**Arguments** are positional and required by default. They come from parameters with no default value:

```python
@app.command()
def process(filename: str, output: str) -> None:
    """Process a file and write results."""
    print(f"Processing {filename} -> {output}")
```

```bash
python cli.py process input.csv output.csv
```

**Options** use `--name VALUE` syntax and are usually optional with defaults. They come from parameters with a default value:

```python
@app.command()
def process(
    filename: str,                        # Argument — required, positional
    output: str = "results.csv",          # Option — optional, has default
    verbose: bool = False,                # Option — flag, --verbose
) -> None:
    """Process a file and write results."""
    if verbose:
        print(f"Processing {filename}...")
    print(f"Writing to {output}")
```

```bash
python cli.py process input.csv
python cli.py process input.csv --output custom.csv
python cli.py process input.csv --verbose
```

Type checking happens automatically. If you pass a string where an integer is expected, Typer reports the error immediately with a helpful message rather than letting the error propagate into your code.

---

### A Complete Multi-Command CLI

Let's build a practical task manager CLI to see how these pieces fit together. The app stores tasks in a JSON file and supports adding, listing, completing, and deleting tasks.

```python
# tasks.py
import json
import typer
from pathlib import Path
from typing import Optional

app = typer.Typer(help="A simple task manager.")

TASKS_FILE = Path.home() / ".tasks.json"


def load_tasks() -> list[dict]:
    if not TASKS_FILE.exists():
        return []
    return json.loads(TASKS_FILE.read_text())


def save_tasks(tasks: list[dict]) -> None:
    TASKS_FILE.write_text(json.dumps(tasks, indent=2))


@app.command()
def add(task: str) -> None:
    """Add a new task."""
    tasks = load_tasks()
    new_task = {
        "id": len(tasks) + 1,
        "title": task,
        "done": False,
    }
    tasks.append(new_task)
    save_tasks(tasks)
    print(f"Added task #{new_task['id']}: {task}")


@app.command(name="list")
def list_tasks(show_done: bool = False) -> None:
    """List all tasks. Use --show-done to include completed tasks."""
    tasks = load_tasks()
    if not tasks:
        print("No tasks found.")
        return

    for task in tasks:
        if task["done"] and not show_done:
            continue
        status = "[x]" if task["done"] else "[ ]"
        print(f"{status} {task['id']}. {task['title']}")


@app.command()
def done(task_id: int) -> None:
    """Mark a task as complete."""
    tasks = load_tasks()
    for task in tasks:
        if task["id"] == task_id:
            task["done"] = True
            save_tasks(tasks)
            print(f"Marked task #{task_id} as done.")
            return
    print(f"Task #{task_id} not found.")
    raise typer.Exit(code=1)


@app.command()
def delete(task_id: int) -> None:
    """Delete a task permanently."""
    tasks = load_tasks()
    updated = [t for t in tasks if t["id"] != task_id]
    if len(updated) == len(tasks):
        print(f"Task #{task_id} not found.")
        raise typer.Exit(code=1)
    save_tasks(updated)
    print(f"Deleted task #{task_id}.")


if __name__ == "__main__":
    app()
```

Using the app:

```bash
python tasks.py add "Buy groceries"
# Added task #1: Buy groceries

python tasks.py add "Write documentation"
# Added task #2: Write documentation

python tasks.py list
# [ ] 1. Buy groceries
# [ ] 2. Write documentation

python tasks.py done 1
# Marked task #1 as done.

python tasks.py list
# [ ] 2. Write documentation

python tasks.py list --show-done
# [x] 1. Buy groceries
# [ ] 2. Write documentation

python tasks.py delete 2
# Deleted task #2.
```

Notice that `typer.Exit(code=1)` is the correct way to exit with a non-zero status code — it signals failure to the calling shell without raising an unhandled exception.

---

### Options and Their Types

Typer supports a wide range of option types through Python's type system.

**Boolean flags** with explicit on/off:

```python
@app.command()
def run(verbose: bool = typer.Option(False, "--verbose/--no-verbose")) -> None:
    """Run with optional verbosity."""
    print(f"Verbose: {verbose}")
```

```bash
python cli.py run --verbose
python cli.py run --no-verbose
```

**Enum choices** restrict input to a fixed set of values:

```python
from enum import Enum


class Environment(str, Enum):
    dev = "dev"
    staging = "staging"
    prod = "prod"


@app.command()
def deploy(env: Environment = Environment.dev) -> None:
    """Deploy to an environment."""
    print(f"Deploying to {env.value}...")
```

```bash
python cli.py deploy --env staging
python cli.py deploy --env invalid
# Error: Invalid value for '--env': 'invalid' is not one of 'dev', 'staging', 'prod'.
```

**Optional values** that default to `None`:

```python
from typing import Optional


@app.command()
def greet(name: str, title: Optional[str] = None) -> None:
    """Greet someone, optionally with a title."""
    if title:
        print(f"Hello, {title} {name}!")
    else:
        print(f"Hello, {name}!")
```

**Multiple values** using `List`:

```python
from typing import List


@app.command()
def notify(users: List[str] = typer.Option(..., "--user")) -> None:
    """Send a notification to one or more users."""
    for user in users:
        print(f"Notifying {user}...")
```

```bash
python cli.py notify --user alice --user bob --user carol
```

**File path validation** ensures a file exists before your code runs:

```python
@app.command()
def analyze(
    filepath: Path = typer.Argument(..., exists=True, file_okay=True, dir_okay=False)
) -> None:
    """Analyze a file."""
    print(f"Analyzing {filepath}...")
```

If the file does not exist, Typer reports a clear error immediately — no need to write your own `os.path.exists` check.

---

### Subcommands and Command Groups

For larger tools, you can compose multiple `Typer` instances into a hierarchy:

```python
# cli.py
import typer

app = typer.Typer(help="Project management tool.")
users_app = typer.Typer(help="Manage users.")
projects_app = typer.Typer(help="Manage projects.")

app.add_typer(users_app, name="users")
app.add_typer(projects_app, name="projects")


@users_app.command(name="list")
def list_users() -> None:
    """List all users."""
    print("Listing users...")


@users_app.command()
def add(name: str, email: str) -> None:
    """Add a new user."""
    print(f"Adding user: {name} <{email}>")


@projects_app.command(name="list")
def list_projects() -> None:
    """List all projects."""
    print("Listing projects...")


if __name__ == "__main__":
    app()
```

```bash
python cli.py --help
python cli.py users --help
python cli.py users add "Alice" "alice@example.com"
python cli.py projects list
```

**Callbacks** let you handle options that apply across all commands, like a `--verbose` flag at the top level:

```python
from typing import Optional

app = typer.Typer()


@app.callback()
def main(verbose: bool = False) -> None:
    """My CLI tool. Use --verbose for detailed output."""
    if verbose:
        print("Verbose mode enabled.")
```

---

### Auto-Generated Help

Typer's killer feature is that documentation writes itself. Every piece of your function signature contributes to the help output:

- **Docstrings** become command descriptions
- **Parameter names** become argument and option names
- **Type hints** are shown as the expected type
- **Default values** are shown in the help text
- **`typer.Option(help="...")`** adds inline descriptions

```python
@app.command()
def export(
    source: str = typer.Argument(help="Database connection string"),
    output: Path = typer.Option("export.csv", help="Output file path"),
    limit: int = typer.Option(1000, help="Maximum rows to export"),
    format: str = typer.Option("csv", help="Output format: csv or json"),
) -> None:
    """Export data from a database to a file.

    Connects to the specified database, queries up to LIMIT rows,
    and writes them to the output file in the specified format.
    """
    print(f"Exporting from {source} to {output}...")
```

```bash
python cli.py export --help

# Usage: cli.py export [OPTIONS] SOURCE
#
#  Export data from a database to a file.
#
#  Connects to the specified database, queries up to LIMIT rows, and writes
#  them to the output file in the specified format.
#
# Arguments:
#   SOURCE  Database connection string [required]
#
# Options:
#   --output PATH    Output file path [default: export.csv]
#   --limit INTEGER  Maximum rows to export [default: 1000]
#   --format TEXT    Output format: csv or json [default: csv]
#   --help           Show this message and exit.
```

You get this for free. No manual help text, no keeping documentation in sync — it comes directly from your code.

---

## Rich — Beautiful Terminal Output

[Rich](https://rich.readthedocs.io) ([GitHub](https://github.com/Textualize/rich)) is a library for rich text and beautiful formatting in the terminal. It was created by Will McGuire at Textualize and has become one of the most widely-used Python libraries for CLI and terminal output.

Rich provides:

- Colored and styled text with a simple markup syntax
- Tables with borders, alignment, and custom styles
- Progress bars for long-running operations
- Panels, columns, and layout primitives
- Syntax-highlighted code blocks
- Markdown rendering in the terminal
- Integration with Python's `logging` module

### Installation

```bash
uv add rich
```

---

### Basic Rich Output

The simplest way to use Rich is as a drop-in replacement for `print`:

```python
from rich import print

print("[bold]Hello, World![/bold]")
print("[red]Error:[/red] Something went wrong.")
print("[green]Success:[/green] Operation completed.")
print("[bold blue]Info:[/bold blue] Processing 42 items...")
```

The markup tags work like HTML: `[bold]`, `[red]`, `[green]`, `[blue]`, `[italic]`, `[underline]`, and combinations like `[bold red]`. They are only rendered when output goes to a terminal — when piped to a file or another process, the plain text is used instead.

For full control, use the `Console` object directly:

```python
from rich.console import Console

console = Console()

console.print("Plain output")
console.print("[bold]Bold output[/bold]")
console.print("Multiple", "args", "work", "too")
console.print({"key": "value"})  # Pretty-prints dicts and objects

# stderr output
error_console = Console(stderr=True)
error_console.print("[red]Error: something failed[/red]")
```

The `Console` object gives you more control: you can direct output to `stderr`, control the terminal width, capture output for testing, and more.

---

### Tables

Rich's `Table` class renders structured data as formatted terminal tables:

```python
from rich.console import Console
from rich.table import Table

console = Console()

table = Table(title="Team Members")

table.add_column("Name", style="cyan", no_wrap=True)
table.add_column("Role", style="magenta")
table.add_column("Email", style="dim")

table.add_row("Alice Chen", "Engineering Lead", "alice@example.com")
table.add_row("Bob Smith", "Backend Engineer", "bob@example.com")
table.add_row("Carol Jones", "Frontend Engineer", "carol@example.com")

console.print(table)
```

This renders a fully-bordered table with a title, column headers, and styled cells.

You can control borders, row styles, and column alignment:

```python
from rich.table import Table, box

table = Table(
    title="Deployment Status",
    box=box.ROUNDED,
    show_header=True,
    header_style="bold white on dark_blue",
)

table.add_column("Environment", justify="left", style="cyan")
table.add_column("Version", justify="center")
table.add_column("Status", justify="center")
table.add_column("Last Deploy", justify="right", style="dim")

table.add_row("Production", "v2.4.1", "[green]Healthy[/green]", "2 hours ago")
table.add_row("Staging", "v2.5.0-rc1", "[yellow]Degraded[/yellow]", "30 min ago")
table.add_row("Development", "v2.5.0-dev", "[red]Down[/red]", "5 min ago")

console.print(table)
```

---

### Progress Bars

Progress bars are invaluable for any script that processes a large number of items or makes the user wait. Rich's progress API makes them easy to add.

**Iterating with progress** using `track()`:

```python
import time
from rich.progress import track

items = list(range(100))

for item in track(items, description="Processing..."):
    time.sleep(0.02)  # Simulate work

print("Done!")
```

`track()` is the simplest form: wrap any iterable and Rich handles the rest.

**Manual progress control** using the `Progress` context manager:

```python
import time
from rich.progress import Progress

with Progress() as progress:
    task1 = progress.add_task("[cyan]Downloading...", total=100)
    task2 = progress.add_task("[magenta]Processing...", total=100)
    task3 = progress.add_task("[green]Uploading...", total=100)

    while not progress.finished:
        progress.update(task1, advance=0.9)
        progress.update(task2, advance=0.6)
        progress.update(task3, advance=0.3)
        time.sleep(0.02)
```

This displays multiple progress bars simultaneously, each advancing at a different rate.

---

### Panels and Layout

Panels draw a border around content with an optional title — useful for highlighting important output:

```python
from rich.console import Console
from rich.panel import Panel

console = Console()

console.print(Panel("Operation completed successfully!", title="Success", style="green"))
console.print(Panel("[red]Database connection failed[/red]", title="Error", border_style="red"))
```

`Columns` arranges multiple items side by side:

```python
from rich.columns import Columns
from rich.panel import Panel

panels = [Panel(f"Item {i}", expand=True) for i in range(4)]
console.print(Columns(panels))
```

`Padding` and `Align` control spacing and position:

```python
from rich.align import Align
from rich.padding import Padding

console.print(Padding("Indented text", (1, 4)))  # 1 line top/bottom, 4 chars left/right
console.print(Align.center("[bold]Centered heading[/bold]"))
```

---

### Syntax Highlighting

Rich can render syntax-highlighted code blocks — useful for displaying config files, generated code, or error context:

```python
from rich.console import Console
from rich.syntax import Syntax

console = Console()

code = '''
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
'''

syntax = Syntax(code, "python", theme="monokai", line_numbers=True)
console.print(syntax)
```

You can also load from a file:

```python
syntax = Syntax.from_path("myconfig.toml", theme="github-dark")
console.print(syntax)
```

---

### Logging with Rich

Rich integrates with Python's standard `logging` module through `RichHandler`, replacing the default plain-text log output with colored, structured log lines:

```python
import logging
from rich.logging import RichHandler

logging.basicConfig(
    level=logging.DEBUG,
    format="%(message)s",
    datefmt="[%X]",
    handlers=[RichHandler(rich_tracebacks=True)],
)

log = logging.getLogger("myapp")

log.debug("Loading configuration...")
log.info("Server started on port 8080")
log.warning("Cache miss rate is high")
log.error("Failed to connect to database")
```

`rich_tracebacks=True` makes exception tracebacks beautiful and much easier to read, with syntax-highlighted source code and local variable values.

---

## Typer + Rich Together

Typer and Rich are designed to work together. Typer handles argument parsing and command routing; Rich handles output formatting. The combination produces CLI tools that are both easy to write and polished to use.

**When to use `typer.echo()` vs `console.print()`:**

- Use `typer.echo()` for simple, unformatted output where you want Typer to handle newlines
- Use `console.print()` when you want colors, tables, panels, or any Rich formatting
- Prefer `console.print()` in most cases — it's more capable and still handles plain text fine

**Error output** should go to stderr:

```python
import sys
from rich.console import Console

console = Console()
error_console = Console(stderr=True)


@app.command()
def process(filename: str) -> None:
    """Process a file."""
    if not Path(filename).exists():
        error_console.print(f"[red]Error:[/red] File not found: {filename}")
        raise typer.Exit(code=1)
    console.print(f"[green]Processing:[/green] {filename}")
```

---

### Complete Example: A Project Scaffolding CLI

Here is a complete CLI tool that combines Typer and Rich. It creates a new project directory with starter files, shows a progress bar while working, and prints a summary panel when done.

```python
# mkproject.py
import time
import typer
from enum import Enum
from pathlib import Path
from rich.console import Console
from rich.panel import Panel
from rich.progress import Progress, SpinnerColumn, TextColumn, BarColumn
from rich.table import Table

app = typer.Typer(help="Scaffold a new Python project.")
console = Console()


class ProjectType(str, Enum):
    python = "python"
    fastapi = "fastapi"


TEMPLATES: dict[str, dict[str, str]] = {
    "python": {
        "README.md": "# {name}\n\nA Python project.\n",
        "pyproject.toml": '[project]\nname = "{name}"\nversion = "0.1.0"\n',
        "src/__init__.py": "",
        "src/main.py": 'def main() -> None:\n    print("Hello from {name}!")\n',
        "tests/__init__.py": "",
        "tests/test_main.py": "from src.main import main\n\ndef test_main() -> None:\n    main()\n",
    },
    "fastapi": {
        "README.md": "# {name}\n\nA FastAPI project.\n",
        "pyproject.toml": '[project]\nname = "{name}"\nversion = "0.1.0"\ndependencies = ["fastapi", "uvicorn"]\n',
        "src/__init__.py": "",
        "src/main.py": 'from fastapi import FastAPI\n\napp = FastAPI()\n\n@app.get("/")\ndef root() -> dict:\n    return {{"message": "Hello from {name}"}}\n',
        "tests/__init__.py": "",
        "tests/test_main.py": "from fastapi.testclient import TestClient\nfrom src.main import app\n\nclient = TestClient(app)\n\ndef test_root() -> None:\n    response = client.get(\"/\")\n    assert response.status_code == 200\n",
    },
}

NEXT_STEPS: dict[str, list[tuple[str, str]]] = {
    "python": [
        ("cd {name}", "Enter your project directory"),
        ("uv venv && source .venv/bin/activate", "Create and activate virtual env"),
        ("uv sync", "Install dependencies"),
        ("python src/main.py", "Run the project"),
    ],
    "fastapi": [
        ("cd {name}", "Enter your project directory"),
        ("uv venv && source .venv/bin/activate", "Create and activate virtual env"),
        ("uv sync", "Install dependencies"),
        ("uvicorn src.main:app --reload", "Start the development server"),
    ],
}


@app.command()
def create(
    name: str = typer.Argument(help="Project name"),
    project_type: ProjectType = typer.Option(
        ProjectType.python, "--type", "-t", help="Project type"
    ),
    output_dir: Path = typer.Option(
        Path("."), "--output", "-o", help="Where to create the project"
    ),
) -> None:
    """Scaffold a new Python project with starter files."""
    project_dir = output_dir / name

    if project_dir.exists():
        console.print(f"[red]Error:[/red] Directory already exists: {project_dir}")
        raise typer.Exit(code=1)

    console.print(
        Panel(
            f"Creating [bold cyan]{name}[/bold cyan] ({project_type.value} project)",
            title="mkproject",
        )
    )

    template = TEMPLATES[project_type.value]

    with Progress(
        SpinnerColumn(),
        TextColumn("[progress.description]{task.description}"),
        BarColumn(),
        TextColumn("[progress.percentage]{task.percentage:>3.0f}%"),
        console=console,
    ) as progress:
        task = progress.add_task("Creating project files...", total=len(template))

        for filepath, content in template.items():
            full_path = project_dir / filepath
            full_path.parent.mkdir(parents=True, exist_ok=True)
            full_path.write_text(content.format(name=name))
            progress.update(task, advance=1, description=f"Writing {filepath}...")
            time.sleep(0.1)  # Makes the progress visible for demo purposes

    # Summary table
    file_table = Table(title="Created Files", show_header=True, header_style="bold")
    file_table.add_column("File", style="cyan")
    file_table.add_column("Size", justify="right", style="dim")

    for filepath in template:
        full_path = project_dir / filepath
        size = full_path.stat().st_size
        file_table.add_row(filepath, f"{size} bytes")

    console.print(file_table)

    # Next steps table
    steps_table = Table(title="Next Steps", show_header=True, header_style="bold")
    steps_table.add_column("#", style="dim", justify="right")
    steps_table.add_column("Command", style="cyan")
    steps_table.add_column("Description")

    for i, (cmd, desc) in enumerate(NEXT_STEPS[project_type.value], 1):
        steps_table.add_row(str(i), cmd.format(name=name), desc)

    console.print(steps_table)
    console.print(Panel(f"[green]Project [bold]{name}[/bold] created successfully![/green]"))


if __name__ == "__main__":
    app()
```

Running the tool:

```bash
python mkproject.py create my-api --type fastapi
```

This displays a spinner-based progress bar as files are created, followed by a table of the created files, a numbered table of next steps, and a success panel. The output is polished enough for a real tool you'd distribute to teammates.

---

## Click — The Foundation

[Click](https://click.palletsprojects.com) is the library that Typer is built on. It was created by Armin Ronacher (the creator of Flask) and has been the standard for Python CLIs for many years.

Click uses a decorator-based API:

```python
import click


@click.command()
@click.argument("name")
@click.option("--count", default=1, help="Number of greetings")
def hello(name: str, count: int) -> None:
    """Simple program that greets NAME."""
    for _ in range(count):
        click.echo(f"Hello, {name}!")


if __name__ == "__main__":
    hello()
```

**When to use Click directly instead of Typer:**

- You need fine-grained control over argument parsing behavior that Typer doesn't expose
- You're working in a codebase that already uses Click extensively
- You need a Click plugin or extension that requires the Click API directly
- You are building a library that other Click applications can extend

For most new projects, Typer is the better starting point. The cleaner API and automatic type validation save time and reduce bugs. Since Typer is built on Click, you can always access the underlying Click objects through `typer.Context` if you need lower-level control.

---

## Packaging Your CLI as a Tool

A CLI that only runs via `python cli.py` is convenient, but a properly packaged tool can be installed and run as a regular command: `my-tool` rather than `python /path/to/my_tool.py`.

### Entry Points in pyproject.toml

The `[project.scripts]` table in `pyproject.toml` maps command names to Python entry points:

```toml
[project]
name = "my-cli-tool"
version = "0.1.0"
description = "A handy CLI tool"
requires-python = ">=3.11"
dependencies = [
    "typer[all]>=0.9.0",
    "rich>=13.0.0",
]

[project.scripts]
my-tool = "my_cli_tool.cli:app"
tasks = "my_cli_tool.tasks:app"
```

The format is `command-name = "package.module:callable"`. For Typer apps, the callable is your `app` object.

### Installing in Development Mode

```bash
uv pip install -e .
```

The `-e` flag installs the package in editable mode: the command is registered in your environment but reads directly from your source files, so changes take effect immediately without reinstalling.

After this, you can run:

```bash
my-tool --help
tasks list
```

### Publishing to PyPI

When your tool is ready to share, you can publish it to PyPI so others can install it with `pip install my-cli-tool`:

```bash
# Build the distribution packages
uv build

# Upload to PyPI (requires a PyPI account and API token)
uv publish
```

Once published, anyone can install and use your tool:

```bash
pip install my-cli-tool
my-tool --help
```

### Running Without Installing with uvx

`uvx` runs a Python CLI tool in a temporary environment without permanently installing it. This is useful for tools you want to use occasionally:

```bash
uvx my-cli-tool --help
```

`uvx` is equivalent to creating a temporary virtual environment, installing the package, running the command, and cleaning up. It is ideal for one-off uses of tools like formatters, linters, and scaffolding utilities.

---

## Type Checking CLI Code

Typer's reliance on type annotations makes your CLI code naturally compatible with mypy and ty. The type hints you write for Typer are the same type hints a type checker uses, so you get both CLI validation and static analysis from the same annotations.

A few patterns to keep in mind:

**`Optional[str]` requires a None check before use:**

```python
from typing import Optional
import typer

app = typer.Typer()


@app.command()
def greet(name: str, title: Optional[str] = None) -> None:
    """Greet someone."""
    if title is not None:
        display_name = f"{title} {name}"
    else:
        display_name = name
    print(f"Hello, {display_name}!")
```

A type checker will catch any attempt to use `title` as a `str` without the None check.

**Enum members are typed correctly:**

```python
from enum import Enum


class Color(str, Enum):
    red = "red"
    green = "green"
    blue = "blue"


@app.command()
def paint(color: Color) -> None:
    """Paint in a color."""
    # color.value is typed as str — no cast needed
    print(f"Painting in {color.value}")
```

**`typer.Exit` is not an exception that carries a value** — use it only to signal success or failure, not to pass data out of a command.

Running your type checker over CLI code catches issues early:

```bash
ty check cli.py
# or
mypy cli.py
```

---

## What's Next?

You now have two powerful tools in your Python CLI development toolkit. Typer eliminates the boilerplate of argument parsing by reading your type annotations, and Rich transforms plain terminal output into something genuinely pleasant to use.

These tools are practical from day one. The next time you find yourself writing a repetitive shell script, writing a data processing pipeline, or building internal developer tooling, reach for Typer and Rich. The investment in a proper CLI pays off quickly: `--help` that actually helps, type-validated inputs, and output that's easy to read at a glance.

In the next chapter, we bring everything together: virtual environments, dependency management, linting, formatting, type checking, testing, and now CLI libraries — all working in concert in a complete, real-world Python project.

[← Previous: AWS Lambda](./13-aws-lambda.md) | [Back to README](./README.md) | [Next: Putting It All Together →](./15-putting-it-together.md)
