# Chapter 12: REST APIs with FastAPI

[← Previous: Database Libraries](./11-database-libraries.md) | [Back to README](./README.md) | [Next: AWS Lambda →](./13-aws-lambda.md)

## What is FastAPI?

[FastAPI](https://fastapi.tiangolo.com) ([GitHub](https://github.com/fastapi/fastapi)) is a modern Python web framework for building REST APIs. It was created by Sebastián Ramírez and first released in 2018, and has since become one of the most popular Python frameworks for building web services.

FastAPI is built on two foundational libraries:

- **[Starlette](https://www.starlette.io/)**: An ASGI (Asynchronous Server Gateway Interface) toolkit that provides the underlying web primitives — routing, requests, responses, middleware, and WebSocket support.
- **[Pydantic](https://docs.pydantic.dev/)**: A data validation library that uses Python type hints to define data schemas and automatically validates incoming data.

The combination is powerful: Starlette gives you async-native, high-performance HTTP handling, and Pydantic gives you type-safe, automatic validation with zero boilerplate.

### Why FastAPI?

**Type hints all the way down.** FastAPI uses standard Python type annotations for everything: path parameters, query parameters, request bodies, and response models. There is no separate schema definition language to learn.

**Automatic interactive documentation.** FastAPI generates OpenAPI (formerly Swagger) documentation from your code automatically. The `/docs` endpoint gives you a fully interactive UI to explore and test your API without writing a single line of documentation.

**Native async support.** FastAPI is built on ASGI and supports `async def` route handlers out of the box. You can mix synchronous and asynchronous endpoints freely.

**Performance.** FastAPI consistently ranks among the fastest Python frameworks in independent benchmarks, comparable to Node.js and Go in many scenarios. This is largely due to Starlette's ASGI foundation and Pydantic V2's Rust-powered validation core.

**Production-proven.** FastAPI is used in production by organizations including Microsoft, Uber, Netflix, and many others.

### Installation

```bash
uv add fastapi uvicorn[standard]
```

`uvicorn` is the ASGI server used to run FastAPI applications. The `[standard]` extra includes useful additions like `watchfiles` (for `--reload`) and `httptools` (for faster HTTP parsing).

---

## Your First FastAPI App

Create a file named `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root() -> dict[str, str]:
    return {"message": "Hello, world!"}
```

Run the development server:

```bash
uvicorn main:app --reload
```

The `main` refers to the module (`main.py`) and `app` is the FastAPI instance. The `--reload` flag watches your files for changes and restarts the server automatically — useful during development, but not for production.

You should see output like:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12345] using WatchFiles
INFO:     Started server process [12346]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

Now open your browser and visit:

- **http://127.0.0.1:8000** — the API response (`{"message": "Hello, world!"}`)
- **http://127.0.0.1:8000/docs** — interactive Swagger UI
- **http://127.0.0.1:8000/redoc** — clean ReDoc documentation

The docs pages are generated automatically from your code. You did not write any YAML, JSON, or documentation strings to get them — FastAPI inferred everything from the function signature and route decorator.

---

## Path Parameters and Query Parameters

### Path Parameters

Path parameters are part of the URL itself. Declare them in the route path with curly braces and in the function signature with the same name:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}")
def get_user(user_id: int) -> dict[str, int]:
    return {"user_id": user_id}
```

FastAPI automatically validates the type. If someone visits `/users/abc`, they get a `422 Unprocessable Entity` error with a clear explanation — before your function is ever called. No `try/except` or manual validation required.

You can use any Python type that Pydantic understands: `int`, `float`, `str`, `UUID`, `datetime`, and more.

### Query Parameters

Query parameters are the key-value pairs after `?` in a URL (e.g., `/users?skip=0&limit=10`). Declare them as function parameters with default values:

```python
@app.get("/users")
def list_users(skip: int = 0, limit: int = 10) -> dict[str, int]:
    return {"skip": skip, "limit": limit}
```

Visiting `/users?skip=20&limit=5` returns `{"skip": 20, "limit": 5}`. Visiting `/users` returns `{"skip": 0, "limit": 10}`.

### Optional Parameters

Use `str | None` (or `Optional[str]` from `typing`) with a default of `None` for optional query parameters:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items")
def search_items(
    q: str | None = None,
    skip: int = 0,
    limit: int = 10,
) -> dict:
    results: dict = {"skip": skip, "limit": limit}
    if q is not None:
        results["query"] = q
    return results
```

FastAPI knows `q` is optional because its default is `None`. It appears in the docs as an optional parameter.

---

## Request Body with Pydantic

For POST, PUT, and PATCH requests you typically send a JSON body. Define the expected shape using a Pydantic `BaseModel`:

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserCreate(BaseModel):
    name: str
    email: str
    age: int


class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int


# Fake in-memory store for the example
_next_id = 1
_users: dict[int, UserResponse] = {}


@app.post("/users", response_model=UserResponse, status_code=201)
def create_user(user: UserCreate) -> UserResponse:
    global _next_id
    new_user = UserResponse(id=_next_id, **user.model_dump())
    _users[_next_id] = new_user
    _next_id += 1
    return new_user
```

Key points:

- FastAPI sees that `user` is a `BaseModel` subclass and treats it as a request body, not a path or query parameter.
- The request body is automatically parsed from JSON, validated against `UserCreate`, and passed to your function as a typed Python object.
- If the client sends invalid JSON or missing required fields, FastAPI returns a detailed `422` error before your handler runs.
- `response_model=UserResponse` tells FastAPI what shape to serialize the response to. Any extra fields on the returned object are filtered out. This is good for security — you can return a full internal model and only expose what `UserResponse` declares.
- `status_code=201` sets the HTTP status code for successful responses.

---

## A Complete REST API Example

Let's build a full CRUD API for a "heroes" resource. This example uses an in-memory dictionary as the data store to keep the focus on FastAPI's features.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(title="Heroes API", version="1.0.0")


# --- Models ---

class HeroCreate(BaseModel):
    name: str
    superpower: str
    age: int | None = None


class HeroUpdate(BaseModel):
    name: str | None = None
    superpower: str | None = None
    age: int | None = None


class HeroRead(BaseModel):
    id: int
    name: str
    superpower: str
    age: int | None = None


# --- In-memory store ---

_heroes: dict[int, HeroRead] = {}
_next_id: int = 1


# --- Routes ---

@app.get("/heroes", response_model=list[HeroRead])
def list_heroes() -> list[HeroRead]:
    """Return all heroes."""
    return list(_heroes.values())


@app.get("/heroes/{hero_id}", response_model=HeroRead)
def get_hero(hero_id: int) -> HeroRead:
    """Return a single hero by ID."""
    hero = _heroes.get(hero_id)
    if hero is None:
        raise HTTPException(status_code=404, detail=f"Hero {hero_id} not found")
    return hero


@app.post("/heroes", response_model=HeroRead, status_code=201)
def create_hero(hero: HeroCreate) -> HeroRead:
    """Create a new hero."""
    global _next_id
    new_hero = HeroRead(id=_next_id, **hero.model_dump())
    _heroes[_next_id] = new_hero
    _next_id += 1
    return new_hero


@app.patch("/heroes/{hero_id}", response_model=HeroRead)
def update_hero(hero_id: int, updates: HeroUpdate) -> HeroRead:
    """Partially update an existing hero."""
    hero = _heroes.get(hero_id)
    if hero is None:
        raise HTTPException(status_code=404, detail=f"Hero {hero_id} not found")

    # Apply only the fields that were provided
    update_data = updates.model_dump(exclude_unset=True)
    updated_hero = hero.model_copy(update=update_data)
    _heroes[hero_id] = updated_hero
    return updated_hero


@app.delete("/heroes/{hero_id}", status_code=204)
def delete_hero(hero_id: int) -> None:
    """Delete a hero by ID."""
    if hero_id not in _heroes:
        raise HTTPException(status_code=404, detail=f"Hero {hero_id} not found")
    del _heroes[hero_id]
```

Notable patterns in this example:

- **`HTTPException`**: Raise this to return an error response with an appropriate status code and a `detail` message. FastAPI serializes it to JSON automatically.
- **`response_model=list[HeroRead]`**: Works with any generic type, including lists of models.
- **`status_code=201`** on POST: Signals that a resource was created.
- **`status_code=204`** on DELETE: Signals success with no response body. The return type is `None`.
- **`exclude_unset=True`**: On the PATCH endpoint, this is critical. It means `model_dump()` only returns the fields the client actually provided, so you do not accidentally overwrite existing data with `None` defaults.

Run the server and visit `http://127.0.0.1:8000/docs` to try each endpoint interactively.

---

## Automatic API Documentation

One of FastAPI's most practical features is automatic documentation generation. Every route you define is reflected immediately in two documentation UIs.

### Swagger UI (`/docs`)

The Swagger UI at `/docs` is fully interactive. You can:

- Browse all available endpoints, grouped by tags
- See the expected request body schema with field types and requirements
- Expand an endpoint and click "Try it out" to send a real request from the browser
- See the exact response, status code, and response headers

This is invaluable during development. You can prototype and test your API without writing any client code or using a separate tool like Postman.

### ReDoc (`/redoc`)

The ReDoc UI at `/redoc` generates a clean, read-only reference document. It is better suited for sharing with API consumers who want to understand the API structure without interacting with it.

### How FastAPI Generates the Docs

FastAPI generates an OpenAPI schema (a JSON document describing your entire API) at `/openapi.json`. Both `/docs` and `/redoc` are just UIs that render this schema.

The schema is generated entirely from:

- Route decorators (`@app.get(...)`, `@app.post(...)`)
- Function signatures (path params, query params, body types)
- Pydantic models (field names, types, required vs. optional, default values)
- Docstrings on route functions (shown as descriptions in the UI)

You do not maintain a separate API specification file. Your code is the specification. When you change a route, the docs update automatically.

You can add metadata to improve the generated docs:

```python
app = FastAPI(
    title="Heroes API",
    description="A simple API for managing heroes.",
    version="1.0.0",
)


@app.get(
    "/heroes/{hero_id}",
    response_model=HeroRead,
    summary="Get a hero",
    tags=["heroes"],
)
def get_hero(hero_id: int) -> HeroRead:
    """Return a single hero by their numeric ID.

    Raises a 404 if the hero does not exist.
    """
    ...
```

---

## Dependency Injection

FastAPI has a built-in dependency injection system using `Depends()`. Dependencies are functions that are called before your route handler, and their return values are injected as arguments.

This is useful for:

- Sharing database sessions across routes
- Extracting and validating authentication tokens
- Applying rate limiting or permission checks

### A Simple Dependency

```python
from fastapi import Depends, FastAPI

app = FastAPI()


def get_db_session():
    """Simulate a database session dependency."""
    db = {"connection": "active"}  # In reality: SQLAlchemy Session, etc.
    try:
        yield db
    finally:
        pass  # Close connection here


@app.get("/items/{item_id}")
def get_item(item_id: int, db: dict = Depends(get_db_session)) -> dict:
    # db is now the yielded value from get_db_session
    return {"item_id": item_id, "db_status": db["connection"]}
```

When `get_db_session` uses `yield`, FastAPI treats it like a context manager. Code before the `yield` runs before your handler; code after runs after (even if an exception occurs). This is the standard pattern for database session management.

### Authentication Dependency

```python
from fastapi import Depends, FastAPI, HTTPException, Header

app = FastAPI()


def require_auth(x_api_key: str = Header()) -> str:
    """Validate the API key passed in the X-Api-Key header."""
    if x_api_key != "secret-key":
        raise HTTPException(status_code=401, detail="Invalid API key")
    return x_api_key


@app.get("/protected")
def protected_route(api_key: str = Depends(require_auth)) -> dict:
    return {"message": "You are authorized", "key": api_key}
```

Dependencies can themselves have dependencies, forming a tree. FastAPI resolves the entire dependency graph before calling your handler.

You can also apply a dependency to all routes in a router or to the entire application:

```python
from fastapi import APIRouter

router = APIRouter(prefix="/admin", dependencies=[Depends(require_auth)])
```

---

## Integration with SQLModel

[SQLModel](https://sqlmodel.tiangolo.com/) is a library, also created by Sebastián Ramírez, that combines SQLAlchemy (for the database ORM) and Pydantic (for validation). The key idea is that you define a model once and it serves as both the database table definition and the API schema.

This integration is where FastAPI truly shines. Here is a complete example:

```python
from contextlib import asynccontextmanager
from typing import AsyncGenerator

from fastapi import Depends, FastAPI, HTTPException
from sqlmodel import Field, Session, SQLModel, create_engine, select

DATABASE_URL = "sqlite:///heroes.db"
engine = create_engine(DATABASE_URL, echo=True)


# --- Models ---

class HeroBase(SQLModel):
    name: str = Field(index=True)
    superpower: str
    age: int | None = None


class Hero(HeroBase, table=True):
    """The database table model."""
    id: int | None = Field(default=None, primary_key=True)


class HeroCreate(HeroBase):
    """Used for POST request bodies."""
    pass


class HeroRead(HeroBase):
    """Used for response serialization."""
    id: int


class HeroUpdate(SQLModel):
    """Used for PATCH request bodies — all fields optional."""
    name: str | None = None
    superpower: str | None = None
    age: int | None = None


# --- Lifespan: create tables on startup ---

@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    SQLModel.metadata.create_all(engine)
    yield


app = FastAPI(lifespan=lifespan)


# --- Dependency: database session ---

def get_session():
    with Session(engine) as session:
        yield session


# --- Routes ---

@app.get("/heroes", response_model=list[HeroRead])
def list_heroes(session: Session = Depends(get_session)) -> list[Hero]:
    return list(session.exec(select(Hero)).all())


@app.get("/heroes/{hero_id}", response_model=HeroRead)
def get_hero(hero_id: int, session: Session = Depends(get_session)) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero


@app.post("/heroes", response_model=HeroRead, status_code=201)
def create_hero(
    hero: HeroCreate,
    session: Session = Depends(get_session),
) -> Hero:
    db_hero = Hero.model_validate(hero)
    session.add(db_hero)
    session.commit()
    session.refresh(db_hero)
    return db_hero


@app.patch("/heroes/{hero_id}", response_model=HeroRead)
def update_hero(
    hero_id: int,
    updates: HeroUpdate,
    session: Session = Depends(get_session),
) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")

    update_data = updates.model_dump(exclude_unset=True)
    hero.sqlmodel_update(update_data)
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero


@app.delete("/heroes/{hero_id}", status_code=204)
def delete_hero(
    hero_id: int,
    session: Session = Depends(get_session),
) -> None:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
```

Key points in this example:

- **`table=True`** on the `Hero` class tells SQLModel to register it as a database table. `HeroCreate` and `HeroRead` are plain Pydantic models (no table).
- **`lifespan`**: The `@asynccontextmanager` lifespan function runs startup and shutdown logic. Code before `yield` runs on startup (creating the tables here), and code after runs on shutdown. This replaced the older `@app.on_event("startup")` pattern in modern FastAPI.
- **`get_session` dependency**: Each request gets its own database session, which is committed or rolled back and then closed when the request finishes.
- **`response_model=HeroRead`**: Even though we return a `Hero` (the ORM model), FastAPI serializes it using `HeroRead`, which only exposes the fields we want.

---

## Testing FastAPI with pytest and httpx

FastAPI applications are straightforward to test. Install the test dependencies:

```bash
uv add -D pytest httpx
```

`httpx` is an async-capable HTTP client. FastAPI's `TestClient` wraps it to provide a synchronous testing interface that does not require a running server.

### Basic Tests

```python
# tests/test_main.py
import pytest
from fastapi.testclient import TestClient

from main import app  # Import your FastAPI app

client = TestClient(app)


def test_list_heroes_empty():
    response = client.get("/heroes")
    assert response.status_code == 200
    assert response.json() == []


def test_create_hero():
    payload = {"name": "Spider-Man", "superpower": "Wall-crawling", "age": 17}
    response = client.post("/heroes", json=payload)
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Spider-Man"
    assert data["superpower"] == "Wall-crawling"
    assert data["age"] == 17
    assert "id" in data


def test_get_hero():
    # Create first
    create_response = client.post(
        "/heroes",
        json={"name": "Thor", "superpower": "Thunder"},
    )
    hero_id = create_response.json()["id"]

    # Then retrieve
    response = client.get(f"/heroes/{hero_id}")
    assert response.status_code == 200
    assert response.json()["name"] == "Thor"


def test_get_hero_not_found():
    response = client.get("/heroes/99999")
    assert response.status_code == 404
    assert response.json()["detail"] == "Hero 99999 not found"


def test_update_hero():
    create_response = client.post(
        "/heroes",
        json={"name": "Iron Man", "superpower": "Powered armor", "age": 45},
    )
    hero_id = create_response.json()["id"]

    patch_response = client.patch(f"/heroes/{hero_id}", json={"age": 46})
    assert patch_response.status_code == 200
    assert patch_response.json()["age"] == 46
    # Other fields unchanged
    assert patch_response.json()["name"] == "Iron Man"


def test_delete_hero():
    create_response = client.post(
        "/heroes",
        json={"name": "Black Widow", "superpower": "Espionage"},
    )
    hero_id = create_response.json()["id"]

    delete_response = client.delete(f"/heroes/{hero_id}")
    assert delete_response.status_code == 204

    # Confirm it is gone
    get_response = client.get(f"/heroes/{hero_id}")
    assert get_response.status_code == 404
```

Run the tests:

```bash
pytest -v
```

### Dependency Overrides

When testing with a real database, you typically want to use a separate test database or an in-memory SQLite instance rather than your production database. FastAPI's `app.dependency_overrides` lets you swap out dependencies for tests:

```python
# tests/test_with_db.py
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, create_engine
from sqlmodel.pool import StaticPool

from main import app, get_session  # Import your app and the dep to override

TEST_DATABASE_URL = "sqlite://"  # In-memory SQLite


@pytest.fixture(name="session")
def session_fixture():
    engine = create_engine(
        TEST_DATABASE_URL,
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session


@pytest.fixture(name="client")
def client_fixture(session: Session):
    def get_session_override():
        return session

    app.dependency_overrides[get_session] = get_session_override
    client = TestClient(app)
    yield client
    app.dependency_overrides.clear()


def test_create_hero_with_db(client: TestClient):
    response = client.post(
        "/heroes",
        json={"name": "Captain America", "superpower": "Super soldier serum"},
    )
    assert response.status_code == 201
    assert response.json()["name"] == "Captain America"
```

Each test run gets a fresh in-memory database. The `session_fixture` creates the tables, and `client_fixture` overrides the `get_session` dependency so the routes use the test session.

---

## Type Safety Benefits

FastAPI's use of Python type hints creates a chain of type safety from the database layer through to the API documentation:

- **Pydantic validates inputs** at the boundary of your application. If a client sends a string where an integer is expected, Pydantic rejects it with a detailed error before your code runs. You never receive malformed data.
- **SQLModel connects DB and API schemas.** A single model definition describes both the table structure and the wire format. There is no risk of the two falling out of sync.
- **Response models filter outputs.** The `response_model` parameter ensures you never accidentally leak internal fields (like password hashes) by controlling exactly what gets serialized.
- **Type checkers (mypy, ty) can verify the whole chain.** Because everything is annotated, your type checker can catch errors like returning the wrong model type from a route before you run the code.
- **No manual serialization.** You return Python objects and FastAPI handles the JSON serialization. You never call `json.dumps()` or manually convert objects to dictionaries.

The practical effect is that many entire categories of bugs — missing fields, wrong types, unexpected nulls — are caught either by the type checker at development time or by Pydantic at runtime before they reach your business logic.

---

## Running in Production

The `--reload` flag is only for development. In production, you want multiple worker processes to handle concurrent requests.

### Multiple Workers with Uvicorn

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

The number of workers is typically set to `(2 * CPU cores) + 1`. For a 2-core machine, that is 5 workers.

### Gunicorn with Uvicorn Workers

For more robust process management, combine Gunicorn (a battle-tested WSGI/ASGI process manager) with Uvicorn worker classes:

```bash
uv add gunicorn
gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000
```

Gunicorn handles process lifecycle (graceful restarts, worker health monitoring), while each worker runs a Uvicorn event loop.

### Configuration with pydantic-settings

Hard-coding configuration values is a common source of errors and security issues. Use `pydantic-settings` to load configuration from environment variables:

```bash
uv add pydantic-settings
```

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    database_url: str = "sqlite:///heroes.db"
    api_key: str = "change-me-in-production"
    debug: bool = False
    workers: int = 4


settings = Settings()
```

Pydantic-settings reads values from environment variables first, then from a `.env` file, then falls back to the default. The variable names match the field names (case-insensitive by default). Type coercion works automatically — setting `DEBUG=true` in your environment gives you a Python `bool`.

In your application, import `settings` and use `settings.database_url` instead of hard-coded strings.

---

## Alternatives to FastAPI

FastAPI is an excellent default choice for new Python REST APIs, but it is worth knowing what else exists.

**[Flask](https://flask.palletsprojects.com/)** is the oldest and most widely deployed Python web framework in this space. It is lightweight, synchronous, and has a vast ecosystem of extensions. Flask is a good fit if you need a simple synchronous API, are joining an existing Flask project, or want to minimize dependencies. It does not have built-in async support or automatic documentation.

**[Django REST Framework](https://www.django-rest-framework.org/)** is built on top of Django and provides a full-featured toolkit for building APIs including serializers, viewsets, routers, authentication, permissions, and a browsable API. If you are already using Django for its ORM, admin interface, or authentication system, DRF is the natural choice. It is more opinionated and has a steeper learning curve than FastAPI.

**[Litestar](https://litestar.dev/)** (formerly Starlite) is a high-performance, modern ASGI framework with a strong emphasis on type safety and a rich plugin system. It is a compelling alternative to FastAPI with some differences in design philosophy, including a class-based controller system and more built-in features. Worth evaluating if FastAPI's design choices do not match your preferences.

---

## What's Next?

You now have a complete picture of building type-safe, well-documented REST APIs with FastAPI. The next step is deployment.

In the next chapter, we look at **AWS Lambda** — a serverless compute platform where you can run your Python code (including FastAPI applications, via adapters like Mangum) without provisioning or managing servers. You pay only for the time your code actually runs, and the platform scales automatically from zero to millions of requests.

[← Previous: Database Libraries](./11-database-libraries.md) | [Back to README](./README.md) | [Next: AWS Lambda →](./13-aws-lambda.md)
