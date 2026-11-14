# Chapter 12: Database Libraries

[← Previous: LLMs](./11-llms.md) | [Back to README](./README.md) | [Next: FastAPI →](./13-fastapi.md)

## Database Access in Python (Graduating from `data.json`)

Every hackathon project starts the same way: *"Let's just save the data in a `data.json` file for now!"* That works until two requests write at the same time and corrupt the file, or your robot logs 50,000 telemetry rows and grinds to a halt re-parsing a 100 MB JSON blob.

Python's database ecosystem is a **four-layer stack**, ranging from raw SQL strings to unified ORM + Pydantic validation models:

```mermaid
flowchart BT
    L1["1️⃣ Raw SQL (DBAPI / sqlite3 / psycopg)\nYou write SQL strings, get back raw tuples: (1, 'Alice', 30)"]
    L2["2️⃣ SQL Expression Builder (SQLAlchemy Core)\nYou write composable Python expressions that compile to SQL"]
    L3["3️⃣ Object-Relational Mapper (SQLAlchemy ORM)\nYou work with Python classes & objects; ORM writes the SQL"]
    L4["4️⃣ Type-Safe ORM + Validation (SQLModel)\nSQLAlchemy ORM + Pydantic combined: ONE class = DB Table + API Schema!"]

    L1 --> L2 --> L3 --> L4
```

| Level | Representative tool | What you write |
|---|---|---|
| Raw SQL (DBAPI) | `sqlite3`, `psycopg`, `pymysql` | Plain SQL strings + Python tuples |
| SQL abstraction / query builder | SQLAlchemy Core | Composable Python expressions that compile to SQL |
| ORM | SQLAlchemy ORM | Mapped Python classes (`DeclarativeBase`, `Mapped[T]`) |
| Type-safe ORM + validation | SQLModel | One class acting as both DB table and Pydantic schema |

*(Other options you may see in the wild include [Tortoise ORM](https://tortoise.github.io/) for Django-style async and [Peewee](https://peewee-orm.com/) for lightweight scripts. For PostgreSQL, use modern **[psycopg](https://www.psycopg.org/)** v3—`uv add "psycopg[binary]"`—rather than legacy `psycopg2`.)*

---

## 1. DBAPI (`sqlite3`) — The Standard Foundation

[PEP 249 (DBAPI 2.0)](https://peps.python.org/pep-0249/) defines the standard interface every Python database driver follows (`connect()`, `cursor()`, `execute()`, `fetchone()`/`fetchall()`, `commit()`, `rollback()`). Python includes `sqlite3` in the standard library—zero dependencies required.

### `sqlite3` with Context Managers and Parameterized Queries

Using `with sqlite3.connect(...) as conn:` automatically commits on clean exit or rolls back if an exception is raised:

```python
import sqlite3

with sqlite3.connect("users.db") as conn:
    conn.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id    INTEGER PRIMARY KEY AUTOINCREMENT,
            name  TEXT    NOT NULL,
            email TEXT    NOT NULL UNIQUE,
            age   INTEGER
        )
    """)

    # ✅ SAFE: Parameterized query with `?` placeholders
    conn.execute(
        "INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
        ("Alice", "alice@example.com", 30),
    )

    cursor = conn.execute("SELECT id, name, email, age FROM users WHERE age >= ?", (25,))
    for row in cursor.fetchall():
        print(row)  # (1, 'Alice', 'alice@example.com', 30)
```

> [!CAUTION]
> **Never build SQL with f-strings (`f"SELECT * FROM users WHERE name = '{name}'"`).**
> That opens a **SQL Injection** hole where an input like `' OR '1'='1` dumps your entire table. Always pass values as a tuple to `execute(sql, params)` (`?` in `sqlite3`, `%s` in `psycopg`).

---

## 2. SQLAlchemy (Core & ORM)

[SQLAlchemy](https://www.sqlalchemy.org) ([GitHub](https://github.com/sqlalchemy/sqlalchemy)) is the backbone of Python's database ecosystem:

```bash
uv add sqlalchemy
```

### SQLAlchemy Core (Composable SQL Expressions)

Core lets you build type-checked, parameterized SQL queries programmatically without full ORM object tracking—ideal for analytics, complex joins, or fixed schemas:

```python
from sqlalchemy import Column, Integer, MetaData, String, Table, create_engine, insert, select

engine = create_engine("sqlite:///core_example.db")
metadata = MetaData()

users_table = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("name", String(100), nullable=False),
    Column("email", String(200), nullable=False, unique=True),
    Column("age", Integer),
)
metadata.create_all(engine)

with engine.begin() as conn:  # Auto-commits at block end
    conn.execute(
        insert(users_table),
        [{"name": "Alice", "email": "alice@example.com", "age": 30}],
    )
    rows = conn.execute(select(users_table).where(users_table.c.age >= 25)).all()
    print(rows)
```

### SQLAlchemy 2.0 ORM (`DeclarativeBase`, `Session`, Relationships)

The ORM maps Python classes to tables using `Mapped[T]` annotations so `ty` and `mypy` can type-check your database attributes:

```python
from sqlalchemy import ForeignKey, String, create_engine, select
from sqlalchemy.orm import DeclarativeBase, Mapped, Session, mapped_column, relationship

engine = create_engine("sqlite:///orm_example.db")


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))
    email: Mapped[str] = mapped_column(String(200), unique=True)
    posts: Mapped[list["Post"]] = relationship(back_populates="author")


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    author: Mapped[User] = relationship(back_populates="posts")


Base.metadata.create_all(engine)

with Session(engine) as session:
    alice = User(name="Alice", email="alice@example.com")
    post = Post(title="Hello ORM", author=alice)
    session.add_all([alice, post])
    session.commit()

    # Query with relationships
    user = session.scalar(select(User).where(User.email == "alice@example.com"))
    if user:
        print(user.name, [p.title for p in user.posts])
```

---

## 3. SQLModel — One Class for Database + Pydantic Validation

[SQLModel](https://sqlmodel.tiangolo.com) ([GitHub](https://github.com/tiangolo/sqlmodel)), created by the author of FastAPI, combines **SQLAlchemy ORM** and **Pydantic** so you don't have to duplicate classes for database tables and API schemas.

```mermaid
flowchart TB
    SM["🦸 class Hero(SQLModel, table=True)\nid: int | None\nname: str\nsecret_name: str\nage: int | None"]
    SA["🗄️ SQLAlchemy ORM\nCreates SQL tables, runs SELECT/INSERT/UPDATE,\nmanages transactions & foreign keys"]
    PY["🛡️ Pydantic v2\nValidates JSON types, serializes API responses,\npowers FastAPI /docs"]
    SM --> SA
    SM --> PY
```

```bash
uv add sqlmodel
```

### Complete SQLModel CRUD Example

```python
from sqlmodel import Field, Session, SQLModel, create_engine, select

engine = create_engine("sqlite:///heroes.db")


class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)


def init_db() -> None:
    SQLModel.metadata.create_all(engine)


def create_hero(hero: Hero) -> Hero:
    with Session(engine) as session:
        session.add(hero)
        session.commit()
        session.refresh(hero)
        return hero


def list_heroes(min_age: int | None = None) -> list[Hero]:
    with Session(engine) as session:
        stmt = select(Hero)
        if min_age is not None:
            stmt = stmt.where(Hero.age >= min_age)  # type: ignore[arg-type]
        return list(session.exec(stmt).all())


def update_hero_age(hero_id: int, new_age: int) -> Hero | None:
    with Session(engine) as session:
        hero = session.get(Hero, hero_id)
        if not hero:
            return None
        hero.age = new_age
        session.add(hero)
        session.commit()
        session.refresh(hero)
        return hero


def delete_hero(hero_id: int) -> bool:
    with Session(engine) as session:
        hero = session.get(Hero, hero_id)
        if not hero:
            return False
        session.delete(hero)
        session.commit()
        return True
```

*(With `table=True`, `Hero` maps to a database table; without `table=True`, a `SQLModel` subclass is a pure Pydantic schema—perfect for FastAPI request/response models in [Chapter 13](./13-fastapi.md)!)*

---

## 4. Database Migrations with Alembic

`create_all()` creates missing tables, but it will **not** alter existing tables when you add or rename columns later. In production, use [Alembic](https://alembic.sqlalchemy.org/) to version-control schema changes:

```bash
uv add alembic
uv run alembic init alembic
```

1. **Set the database URL** in `alembic.ini` (e.g., `sqlalchemy.url = sqlite:///heroes.db` or `postgresql+psycopg://user:pass@localhost/db`).
2. **Point `alembic/env.py` at your metadata:**
   ```python
   from sqlmodel import SQLModel
   import myapp.models  # Ensure models are imported so tables register!

   target_metadata = SQLModel.metadata  # Or Base.metadata for SQLAlchemy ORM
   ```
3. **Generate and apply migrations:**
   ```bash
   # Auto-generate a migration script by diffing models vs. database
   uv run alembic revision --autogenerate -m "add hero age column"

   # Review the generated script in alembic/versions/, then apply it:
   uv run alembic upgrade head

   # Roll back one migration if needed:
   uv run alembic downgrade -1
   ```

---

## Comparison & When to Choose Each

| Feature | `sqlite3` / DBAPI | SQLAlchemy Core | SQLAlchemy ORM | SQLModel |
|---|---|---|---|---|
| **Abstraction** | Raw SQL strings | Python SQL builder | Object-Relational Mapper | ORM + Pydantic v2 |
| **Type safety** | None (raw tuples) | Column-level | Full (`Mapped[T]`) | Full (DB + JSON validation) |
| **Migrations** | Manual SQL | Alembic | Alembic | Alembic |
| **Dependencies** | Zero (stdlib) | `sqlalchemy` | `sqlalchemy` | `sqlmodel` |
| **Best for** | Small scripts, embedded local state, learning SQL | Complex analytical queries, existing fixed schemas | Large backend services with rich domain models | **FastAPI apps** & modern typed Python backends |

- **Pick `sqlite3` (DBAPI)** for zero-dependency CLI tools, local caches, or small robotics/IoT logs.
- **Pick SQLAlchemy Core** when you want composable, injection-safe SQL expressions without ORM object tracking.
- **Pick SQLAlchemy ORM** for large services where database models and API schemas are intentionally separate layers.
- **Pick SQLModel** when building FastAPI services where you want one set of type-annotated classes to power both your database and your REST endpoints.

---

## What's Next?

Now that we can persist data cleanly with SQLModel, let's expose it over HTTP with automatic validation and interactive OpenAPI docs in [Chapter 13: REST APIs with FastAPI](./13-fastapi.md).

[← Previous: LLMs](./11-llms.md) | [Back to README](./README.md) | [Next: FastAPI →](./13-fastapi.md)
