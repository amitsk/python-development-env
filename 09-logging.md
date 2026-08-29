# Chapter 9: Logging

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: CI/CD →](./10-ci-cd.md)

## Why Logging Matters

Every application needs a way to report what it is doing. The simplest approach is `print()`:

```python
print("Starting job...")
print(f"Processing {len(records)} records")
print("Done.")
```

This works for a ten-line script. It falls apart the moment you care about *when* something happened, *how bad* it was, or *where* the message should go.

`print()` cannot:

- Turn debug chatter on in development and off in production without editing code
- Attach a timestamp, severity, or request ID
- Send errors to a file (or CloudWatch, or Datadog) while keeping info on stdout
- Filter by module without a pile of `if` statements you will regret

Logging is that same idea with a volume knob, a clock, and a forwarding address.

This chapter covers three tools that still dominate Python logging:

| Tool | Role |
|------|------|
| [stdlib `logging`](https://docs.python.org/3/library/logging.html) | Zero dependencies, the API everyone else speaks |
| [Loguru](https://github.com/Delgan/loguru) | Ergonomic logging with almost no boilerplate |
| [Picologging](https://github.com/microsoft/picologging) | C-speed replacement for the stdlib API (still beta) |

## The Standard Library: `logging`

Python's built-in `logging` module is the baseline. Every other library either wraps it, intercepts it, or pretends to be it. Learn these concepts once and they transfer.

### Log Levels

```python
import logging

# Five standard levels, in increasing severity:
logging.debug("Detailed diagnostic info")      # DEBUG
logging.info("Normal operation")               # INFO
logging.warning("Something unexpected")        # WARNING
logging.error("Something failed")              # ERROR
logging.critical("Application cannot continue")  # CRITICAL
```

By default, only WARNING and above are shown. That is why a fresh `logging.info("hello")` appears to do nothing — it is not broken, it is being polite. You set the threshold when you configure logging.

### Basic Setup

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

logger = logging.getLogger(__name__)

logger.debug("Debug message")
logger.info("Application started")
logger.warning("Config file not found, using defaults")
logger.error("Failed to connect to database")
```

Output:

```
2026-08-28 12:00:00 DEBUG    myapp.main: Debug message
2026-08-28 12:00:00 INFO     myapp.main: Application started
2026-08-28 12:00:00 WARNING  myapp.main: Config file not found, using defaults
2026-08-28 12:00:00 ERROR    myapp.main: Failed to connect to database
```

`basicConfig()` is enough for scripts. Larger apps usually create handlers and formatters by hand so console and file output can disagree about verbosity.

### Named Loggers

Use `logging.getLogger(__name__)` in every module. The name follows the package hierarchy:

```python
# In myapp/database.py
logger = logging.getLogger(__name__)  # "myapp.database"

# In myapp/api/routes.py
logger = logging.getLogger(__name__)  # "myapp.api.routes"
```

You can then turn the volume up in one corner of the app without drowning in the rest:

```python
logging.getLogger("myapp.database").setLevel(logging.DEBUG)
logging.getLogger("myapp.api").setLevel(logging.WARNING)
```

Never use the root logger (`logging.info(...)`) from library-style code. Named loggers are how you keep SQLAlchemy's chatter from looking like your own.

### Handlers and Formatters

Handlers decide *where* messages go. Formatters decide *how they look*.

```python
import logging

logger = logging.getLogger("myapp")
logger.setLevel(logging.DEBUG)

console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(
    logging.Formatter("%(asctime)s %(levelname)s %(name)s: %(message)s")
)

logger.addHandler(console)
logger.addHandler(file_handler)
```

A logger's own level is a first filter. Each handler can filter again. Debug lines can land in a file while the console stays readable.

### Structured Logging with `logging`

Machines prefer fields to sentences. The `extra` dict is the stdlib's way to attach context:

```python
logger.info(
    "Request completed",
    extra={
        "method": "GET",
        "path": "/api/users",
        "status": 200,
        "duration_ms": 42,
    },
)
```

Those keys only appear if your formatter knows about them. For JSON — the format log aggregators actually want — use a JSON formatter. [python-json-logger](https://github.com/nhairs/python-json-logger) is a common choice:

```bash
uv add python-json-logger
```

```python
import logging
from pythonjsonlogger.json import JsonFormatter

handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger = logging.getLogger("myapp")
logger.addHandler(handler)
logger.setLevel(logging.INFO)
```

Check the library's current docs if the import path looks unfamiliar. The package has been through a fork; older tutorials still show `pythonjsonlogger.jsonlogger`.

## Loguru

[Loguru](https://github.com/Delgan/loguru) replaces the handler/formatter ritual with one object you import and start using. Colored output, timestamps, and caller info come for free. For new projects, this is the library we recommend.

```bash
uv add loguru
```

### Zero-Config Default Sink

```python
from loguru import logger

logger.debug("Debug message")
logger.info("Application started")
logger.warning("Config file not found")
logger.error("Failed to connect")
logger.critical("Unrecoverable error")
```

No `basicConfig()`, no `getLogger(__name__)`. There is one `logger`, pre-configured to write to stderr. That is the whole pitch, and it delivers.

### Configuring Sinks

A *sink* is Loguru's word for a destination (what stdlib logging calls a handler):

```python
import sys
from loguru import logger

logger.remove()  # drop the default stderr sink

logger.add(
    sys.stderr,
    level="INFO",
    colorize=True,
    format=(
        "<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
        "<level>{level: <8}</level> | "
        "<cyan>{name}</cyan>:<cyan>{line}</cyan> - "
        "<level>{message}</level>"
    ),
)

logger.add(
    "logs/app.log",
    level="DEBUG",
    rotation="10 MB",    # new file when this one fills up
    retention="7 days",  # delete older rotated files
    compression="gz",    # compress the ones you keep
)

# JSON for production / log shipping
logger.add("logs/app.json", level="INFO", serialize=True)
```

`serialize=True` is Loguru's JSON switch. No extra formatter package required.

### Structured Context with `logger.bind()`

```python
from loguru import logger

request_logger = logger.bind(request_id="abc-123", user_id=42)

request_logger.info("Processing request")
request_logger.debug("Query executed", query="SELECT ...")
```

Bound fields travel with every subsequent message from that logger. This is how you correlate a request across functions without threading an ID through every signature.

### Exception Tracebacks

```python
from loguru import logger

@logger.catch  # logs the exception with a full traceback
def process(data):
    return data["key"]  # KeyError if the key is missing

process({})
```

`@logger.catch` logs the crash; it does not magically make the error disappear. For `try`/`except` you already own, use `logger.exception()`:

```python
try:
    result = risky_operation()
except Exception:
    logger.exception("Operation failed")  # message + traceback
```

### Intercepting stdlib Logging

Third-party libraries (SQLAlchemy, httpx, uvicorn, …) still use `logging`. Route that output through Loguru with an `InterceptHandler`:

```python
import inspect
import logging

from loguru import logger


class InterceptHandler(logging.Handler):
    def emit(self, record: logging.LogRecord) -> None:
        try:
            level: str | int = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno

        frame, depth = inspect.currentframe(), 0
        while frame and (depth == 0 or frame.f_code.co_filename == logging.__file__):
            frame = frame.f_back
            depth += 1

        logger.opt(depth=depth, exception=record.exc_info).log(
            level, record.getMessage()
        )


logging.basicConfig(handlers=[InterceptHandler()], level=0, force=True)
```

After this, `logging.getLogger(...)` calls flow through your Loguru sinks. One format, one rotation policy, one place to look.

## Picologging

[Picologging](https://github.com/microsoft/picologging) is a Microsoft-built, C-accelerated reimplementation of the stdlib `logging` API. Change the import, keep the rest of the code. Benchmarks on the project README put it roughly 4–17× faster than stdlib logging, depending on the call.

It is still **beta**. The README says so plainly. Features are incomplete (see the project's [limitations](https://microsoft.github.io/picologging/limitations.html)), and there is an [open issue](https://github.com/microsoft/picologging/issues/229) about Python 3.13 and 3.14 support. Treat it as a performance experiment on a version it actually ships wheels for — not as a risk-free drop-in on 3.13/3.14.

```bash
uv add picologging
```

### Same API as stdlib `logging`

```python
import picologging as logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

logger.info("Same API, faster implementation")
logger.warning("Config issue detected")
logger.error("Connection failed")
```

Handlers, formatters, named loggers, and log levels work the way they do in the stdlib. That is the appeal: you do not rewrite the logging layer, you swap the engine.

### Structured Logging

Picologging does not ship a JSON formatter. Roll a small one, or keep using a stdlib-compatible formatter if it works on your version:

```python
import json

import picologging as logging
from picologging import StreamHandler


class JsonFormatter(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        log_entry = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
        }
        if record.exc_info:
            log_entry["exception"] = self.formatException(record.exc_info)
        return json.dumps(log_entry)


handler = StreamHandler()
handler.setFormatter(JsonFormatter())

logger = logging.getLogger("myapp")
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("Request processed")
```

### When to Use Picologging

Use it only when all of these are true:

- The codebase already uses stdlib `logging` and you do not want to migrate to Loguru
- You are on a Python version picologging actually supports (check wheels and that open 3.13/3.14 issue before you commit)
- Logging is on a hot path — a high-throughput service where the stdlib's overhead shows up in profiles

If any of those is a no, pick Loguru or stay on the stdlib. Speed that you cannot install is not speed.

## Comparison

| Feature | `logging` (stdlib) | Loguru | Picologging |
|---|---|---|---|
| **Setup** | Verbose | Minimal | Same as `logging` |
| **API** | Standard | New API | Standard (drop-in) |
| **Colored output** | Manual | Built-in | Manual |
| **File rotation** | Manual (`RotatingFileHandler`) | Built-in | Manual |
| **JSON output** | Via formatter (`python-json-logger`, etc.) | `serialize=True` | Via formatter |
| **Tracebacks** | Basic | Rich + `@logger.catch` | Basic |
| **Performance** | Baseline | Roughly stdlib | Several times faster, when it runs |
| **Dependencies** | None | 1 package | 1 package (C extension) |
| **Maturity** | Standard library | Stable | Beta; 3.13/3.14 support is unresolved |
| **Best for** | Zero deps, max compatibility | New projects | Hot-path stdlib code on a supported Python |

### Recommendations

- **New project:** Use **Loguru**. You will write less logging code and get rotation, color, and JSON without a side quest.
- **Zero extra dependencies:** Use the **stdlib `logging` module**. It is not fashionable. It will still be there in ten years.
- **Existing stdlib `logging`, hot path, supported Python:** Evaluate **Picologging**. Confirm it installs and behaves on *your* interpreter. Do not assume 3.13/3.14 are fine.
- **FastAPI / anything behind a log aggregator:** Emit **structured JSON** (Loguru `serialize=True`, or stdlib plus a JSON formatter) so fields can be indexed instead of grepped.

## Logging in FastAPI

Configure logging once at startup, then bind request fields in middleware:

```python
import sys

from fastapi import FastAPI, Request
from loguru import logger

logger.remove()
logger.add(sys.stdout, level="INFO", serialize=True)  # JSON for production

app = FastAPI()


@app.middleware("http")
async def log_requests(request: Request, call_next):
    logger.info("Request started", method=request.method, path=request.url.path)
    response = await call_next(request)
    logger.info("Request completed", status=response.status_code)
    return response
```

`serialize=True` turns keyword arguments into JSON fields. Your future self, debugging a 500 from a log search UI, will be grateful.

## What's Next?

Logs tell you what the app did. CI/CD is how you make sure the next change did not quietly stop it from doing that. Next: GitHub Actions with uv, Ruff, ty, and pytest.

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: CI/CD →](./10-ci-cd.md)
