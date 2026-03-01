# Chapter 9: Logging

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: CI/CD →](./10-ci-cd.md)

---

## Why Logging Matters

Every application needs a way to report what it is doing. The simplest approach is `print()`:

```python
print("Starting job...")
print(f"Processing {len(records)} records")
print("Done.")
```

This works fine for quick scripts, but falls apart in real applications:

- You can't easily turn debug output on or off
- There's no timestamp, severity level, or context attached to messages
- Output goes only to stdout — you can't route errors to a file or external service
- You can't filter messages by module or severity without editing the code

Proper logging gives you all of that with minimal effort.

---

## The Standard Library: `logging`

Python's built-in `logging` module is the baseline. It introduces the concepts all logging libraries build on.

### Log Levels

```python
import logging

# Five standard levels, in increasing severity:
logging.debug("Detailed diagnostic info")     # DEBUG
logging.info("Normal operation")              # INFO
logging.warning("Something unexpected")       # WARNING
logging.error("Something failed")             # ERROR
logging.critical("Application cannot continue")  # CRITICAL
```

By default, only WARNING and above are shown. You configure the threshold when setting up logging.

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
2024-05-01 12:00:00 DEBUG    myapp.main: Debug message
2024-05-01 12:00:00 INFO     myapp.main: Application started
2024-05-01 12:00:00 WARNING  myapp.main: Config file not found, using defaults
2024-05-01 12:00:00 ERROR    myapp.main: Failed to connect to database
```

### Named Loggers

Use `logging.getLogger(__name__)` in every module. This creates a hierarchy:

```python
# In myapp/database.py
logger = logging.getLogger(__name__)  # logger name: "myapp.database"

# In myapp/api/routes.py
logger = logging.getLogger(__name__)  # logger name: "myapp.api.routes"
```

You can then configure log levels per module:

```python
logging.getLogger("myapp.database").setLevel(logging.DEBUG)
logging.getLogger("myapp.api").setLevel(logging.WARNING)
```

### Handlers

Handlers control where log messages go:

```python
import logging

logger = logging.getLogger("myapp")
logger.setLevel(logging.DEBUG)

# Console handler
console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(logging.Formatter("%(levelname)s: %(message)s"))

# File handler
file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter(
    "%(asctime)s %(levelname)s %(name)s: %(message)s"
))

logger.addHandler(console)
logger.addHandler(file_handler)
```

### Structured Logging with `logging`

Add context to messages using the `extra` parameter:

```python
logger.info("Request completed", extra={
    "method": "GET",
    "path": "/api/users",
    "status": 200,
    "duration_ms": 42,
})
```

For JSON output (useful when sending logs to a service like Datadog or CloudWatch), use a custom formatter or a library like `python-json-logger`:

```bash
uv add python-json-logger
```

```python
from pythonjsonlogger import jsonlogger

handler = logging.StreamHandler()
handler.setFormatter(jsonlogger.JsonFormatter())
logger.addHandler(handler)
```

---

## Loguru

[Loguru](https://github.com/Delgan/loguru) is a third-party library that replaces the standard `logging` module with a simpler, more ergonomic API. The entire configuration happens in one place, there's no handler/formatter boilerplate, and it adds features like colored output, automatic tracebacks, and easy structured logging.

```bash
uv add loguru
```

### Basic Usage

```python
from loguru import logger

logger.debug("Debug message")
logger.info("Application started")
logger.warning("Config file not found")
logger.error("Failed to connect")
logger.critical("Unrecoverable error")
```

That's it — no setup required. Loguru configures sensible defaults (colored output, timestamps, log level, caller information) out of the box.

### Configuring Sinks

In Loguru, a "sink" is the destination for log messages (equivalent to a Handler):

```python
from loguru import logger
import sys

# Remove the default sink
logger.remove()

# Add a console sink (stderr) showing INFO and above
logger.add(sys.stderr, level="INFO", colorize=True,
           format="<green>{time:YYYY-MM-DD HH:mm:ss}</green> | "
                  "<level>{level: <8}</level> | "
                  "<cyan>{name}</cyan>:<cyan>{line}</cyan> - "
                  "<level>{message}</level>")

# Add a file sink with rotation
logger.add(
    "logs/app.log",
    level="DEBUG",
    rotation="10 MB",    # rotate when file reaches 10 MB
    retention="7 days",  # delete logs older than 7 days
    compression="gz",    # compress rotated files
)

# Add a JSON sink for production/shipping
logger.add("logs/app.json", level="INFO", serialize=True)
```

### Structured Context

```python
from loguru import logger

# Bind context that will appear in every subsequent message
request_logger = logger.bind(request_id="abc-123", user_id=42)

request_logger.info("Processing request")
request_logger.debug("Query executed", query="SELECT ...")
```

### Automatic Exception Tracebacks

```python
from loguru import logger

@logger.catch  # automatically logs exceptions with full traceback
def process(data):
    return data["key"]  # KeyError if key missing

process({})  # logs the exception, does not crash silently
```

Or use `logger.exception()` in a try/except block:

```python
try:
    result = risky_operation()
except Exception:
    logger.exception("Operation failed")  # logs message + full traceback
```

### Using Loguru with Standard Library Logging

Third-party libraries use the standard `logging` module. You can route their output through Loguru:

```python
import logging
from loguru import logger

class InterceptHandler(logging.Handler):
    def emit(self, record):
        try:
            level = logger.level(record.levelname).name
        except ValueError:
            level = record.levelno
        logger.opt(depth=6, exception=record.exc_info).log(
            level, record.getMessage()
        )

logging.basicConfig(handlers=[InterceptHandler()], level=0, force=True)
```

After this, all `logging.getLogger(...)` output from libraries (SQLAlchemy, httpx, etc.) flows through Loguru.

---

## Picologging

[Picologging](https://github.com/microsoft/picologging) is a drop-in replacement for the standard `logging` module developed by Microsoft. It has the same API as `logging` — you change only the import — but is implemented in C and is significantly faster.

```bash
uv add picologging
```

### Usage

Replace `import logging` with `import picologging as logging`:

```python
import picologging as logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

logger.info("Same API, faster implementation")
logger.warning("Config issue detected")
logger.error("Connection failed")
```

That's it. The entire standard `logging` API works — handlers, formatters, named loggers, log levels — with no code changes.

### Structured Logging

```python
import picologging as logging
from picologging import StreamHandler
import json

class JsonFormatter(logging.Formatter):
    def format(self, record):
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

Picologging is the right choice when:
- You have an existing codebase using the standard `logging` module and don't want to refactor
- Logging is on the hot path — high-throughput services where logging overhead matters
- You want a drop-in performance improvement with zero risk

---

## Comparison

| Feature | `logging` (stdlib) | Loguru | Picologging |
|---|---|---|---|
| **Setup** | Verbose | Minimal | Same as `logging` |
| **API** | Standard | New API | Standard (drop-in) |
| **Colored output** | Manual | Built-in | Manual |
| **File rotation** | Manual | Built-in | Manual |
| **JSON output** | Via formatter | `serialize=True` | Via formatter |
| **Tracebacks** | Basic | Rich + decorators | Basic |
| **Performance** | Baseline | ~stdlib | 4-10× faster |
| **Dependencies** | None | 1 package | 1 package (C ext) |
| **Best for** | Simple scripts, max compatibility | New projects, DX | High-throughput, drop-in upgrade |

### Recommendations

- **New project, small team:** Use **Loguru**. The ergonomics are excellent and the setup is trivial.
- **Existing codebase using `logging`:** Drop in **Picologging** for a free performance boost.
- **No external dependencies allowed:** Use the **standard `logging` module**. It covers everything you need.
- **FastAPI / async service:** Use **Loguru** with a JSON sink — structured logs integrate well with log aggregation services.

---

## Logging in FastAPI

```python
import sys
from loguru import logger
from fastapi import FastAPI, Request

# Configure once at startup
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

---

## What's Next?

With logging set up, the next step is automating all your quality checks — testing, linting, type checking, and more — through CI/CD.

[← Previous: Static Type Checking](./08-static-type-checking.md) | [Back to README](./README.md) | [Next: CI/CD →](./10-ci-cd.md)
