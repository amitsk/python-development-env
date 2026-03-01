# Chapter 12: Database Libraries

[← Previous: LLMs](./11-llms.md) | [Back to README](./README.md) | [Next: FastAPI →](./13-fastapi.md)

## Database Access in Python

Almost every real-world application needs to store and retrieve data. Python has a rich ecosystem of database libraries spanning a wide spectrum — from raw SQL executed through a minimal standard interface, to full-featured ORMs that let you work entirely in Python objects, to modern type-safe tools that bridge your database and your API layer in a single model definition.

Understanding where each tool sits on this spectrum helps you pick the right one for a given project rather than defaulting to whatever you used last time.

### The Four Levels of Abstraction

| Level | Representative tool | What you write |
|---|---|---|
| Raw SQL (DBAPI) | `sqlite3`, `psycopg2`, `pymysql` | Plain SQL strings + Python tuples |
| SQL abstraction / query builder | SQLAlchemy Core | Python expressions that compile to SQL |
| ORM | SQLAlchemy ORM | Python classes and method calls |
| Type-safe ORM + validation | SQLModel | One model class used for DB and API |

**DBAPI** is the lowest level — the standard interface defined by PEP 249 that every Python database driver implements. You write SQL yourself and get back raw tuples. It is the right choice when you need maximum control, have no dependencies to spare, or are learning SQL.

**SQLAlchemy Core** sits one level up. You describe your schema in Python and build queries using an expression language that generates SQL for you. You retain full control over the SQL being produced, but Python does the string-building and parameterization work.

**SQLAlchemy ORM** lets you map Python classes to database tables and write queries in terms of objects and relationships. It handles the translation to SQL, tracks changes to objects, and manages transactions. This is the most common choice for production web applications.

**SQLModel** layers Pydantic's validation model on top of SQLAlchemy ORM. A single class declaration is both a database table and a Pydantic schema — which means the same object validates incoming API requests and maps to database rows. This is particularly compelling when building FastAPI applications.

### Other Options Worth Knowing

- **[databases](https://www.encode.io/databases/)** — async query support on top of SQLAlchemy Core; useful when you need non-blocking database calls without a full async ORM.
- **[Tortoise ORM](https://tortoise.github.io/)** — Django-inspired async ORM, a common choice for async-first applications built without FastAPI/SQLModel.
- **[Peewee](https://peewee-orm.com/)** — a small, expressive ORM with minimal dependencies; good for small scripts and tools where SQLAlchemy feels heavy.

---

## DBAPI — The Foundation

### PEP 249 and the Standard Interface

[PEP 249](https://peps.python.org/pep-0249/) defines the standard interface that all Python database drivers must follow. Whether you are connecting to SQLite, PostgreSQL, MySQL, or any other database, the calls look the same:

- `connect()` — open a connection
- `connection.cursor()` — create a cursor to execute statements
- `cursor.execute(sql, params)` — run a query
- `cursor.fetchone()` / `cursor.fetchall()` — retrieve results
- `connection.commit()` / `connection.rollback()` — manage transactions

This means that if you learn DBAPI with `sqlite3`, you already understand the basic shape of `psycopg2` (PostgreSQL) and `pymysql` (MySQL).

### sqlite3 — No Installation Required

Python ships with `sqlite3` in the standard library. It is an excellent way to learn DBAPI concepts, and it is genuinely useful for small applications, scripts, and prototypes.

```python
import sqlite3

# Open a connection (creates the file if it doesn't exist)
conn = sqlite3.connect("users.db")
cursor = conn.cursor()

# Create a table
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id    INTEGER PRIMARY KEY AUTOINCREMENT,
        name  TEXT    NOT NULL,
        email TEXT    NOT NULL UNIQUE,
        age   INTEGER
    )
""")
conn.commit()

# Insert a row
cursor.execute(
    "INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
    ("Alice", "alice@example.com", 30),
)
conn.commit()

# Fetch all rows
cursor.execute("SELECT * FROM users")
rows = cursor.fetchall()
for row in rows:
    print(row)  # (1, 'Alice', 'alice@example.com', 30)

# Fetch one row
cursor.execute("SELECT * FROM users WHERE id = ?", (1,))
row = cursor.fetchone()
print(row)

conn.close()
```

### Parameterized Queries

Notice the `?` placeholders in the queries above. This is parameterized query syntax, and it is one of the most important habits to build when working with databases.

**Never** build SQL strings by concatenating user input:

```python
# DANGEROUS — SQL injection vulnerability
name = input("Enter a name: ")
cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")
# A user who enters:  ' OR '1'='1
# turns the query into:  SELECT * FROM users WHERE name = '' OR '1'='1'
# which returns every row in the table.
```

**Always** use parameterized queries:

```python
# SAFE — the database driver handles escaping
name = input("Enter a name: ")
cursor.execute("SELECT * FROM users WHERE name = ?", (name,))
```

The database driver sends the query and the parameters separately. The database engine never interprets the parameter value as SQL, so injection is impossible.

### Commit and Rollback

Database operations are grouped into transactions. Changes made with `execute()` are not persisted until you call `commit()`. If something goes wrong, `rollback()` undoes all changes since the last commit.

```python
conn = sqlite3.connect("users.db")
cursor = conn.cursor()

try:
    cursor.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        ("Bob", "bob@example.com"),
    )
    cursor.execute(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        ("Carol", "carol@example.com"),
    )
    conn.commit()  # Persist both inserts together
    print("Both users inserted successfully")
except Exception as e:
    conn.rollback()  # Undo both inserts if either failed
    print(f"Transaction failed, rolled back: {e}")
finally:
    conn.close()
```

### Context Manager Usage

Python's `sqlite3` module supports the context manager protocol. Using `with conn:` automatically commits the transaction when the block exits cleanly, or rolls it back if an exception is raised. This is the recommended pattern.

```python
import sqlite3

with sqlite3.connect("users.db") as conn:
    conn.execute(
        "INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
        ("Dave", "dave@example.com", 25),
    )
    # Automatically committed on exit; rolled back on exception
```

### Limitations of Raw DBAPI

Raw DBAPI works well, but it has real limitations as a project grows:

- **No type safety.** Rows come back as plain tuples. Accessing `row[2]` instead of `row["email"]` is easy to get wrong and impossible for a type checker to catch.
- **Manual row mapping.** You must write code to convert tuples into objects yourself.
- **No schema management.** Nothing prevents you from making a typo in a column name until the query actually runs.
- **Repetitive boilerplate.** INSERT, UPDATE, and SELECT queries for a ten-column table require writing out every column name repeatedly.

These limitations are exactly what the higher-level tools solve.

---

## SQLAlchemy

- Website: [https://www.sqlalchemy.org](https://www.sqlalchemy.org)
- GitHub: [https://github.com/sqlalchemy/sqlalchemy](https://github.com/sqlalchemy/sqlalchemy)

SQLAlchemy is the most widely used Python SQL toolkit and ORM. It is the foundation that SQLModel, Alembic, and many other tools build on. If you work on Python backend applications, you will encounter SQLAlchemy.

The library is structured in two distinct layers that you can use independently or together:

- **SQLAlchemy Core** — the SQL Expression Language. Provides Python objects that represent tables, columns, and SQL constructs. Gives you full control over the SQL that gets generated while handling parameterization and escaping for you.
- **SQLAlchemy ORM** — the Object Relational Mapper. Builds on Core and lets you work with mapped Python classes and objects instead of tables and rows directly.

### Installation

```bash
uv add sqlalchemy
```

---

### SQLAlchemy Core

Core is the right choice when you want tight control over your SQL — for example, when writing complex queries with multiple joins, window functions, or database-specific features — while still getting the benefits of Python's type system and composable query construction.

#### Defining Tables and Creating an Engine

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, insert, select

# Create an engine. The connection URL specifies the database type and location.
# For SQLite: "sqlite:///mydb.db"
# For PostgreSQL: "postgresql://user:password@localhost/dbname"
engine = create_engine("sqlite:///core_example.db", echo=True)

# MetaData holds all table definitions
metadata = MetaData()

# Define the users table
users_table = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True, autoincrement=True),
    Column("name", String(100), nullable=False),
    Column("email", String(200), nullable=False, unique=True),
    Column("age", Integer),
)

# Create all tables that have been defined on this MetaData object
metadata.create_all(engine)
```

#### Executing Queries with Core

```python
from sqlalchemy import update, delete

# Insert rows
with engine.connect() as conn:
    conn.execute(
        insert(users_table),
        [
            {"name": "Alice", "email": "alice@example.com", "age": 30},
            {"name": "Bob", "email": "bob@example.com", "age": 25},
        ],
    )
    conn.commit()

# Select rows
with engine.connect() as conn:
    result = conn.execute(
        select(users_table).where(users_table.c.age >= 25).order_by(users_table.c.name)
    )
    for row in result:
        print(row.name, row.email, row.age)

# Update rows
with engine.connect() as conn:
    conn.execute(
        update(users_table)
        .where(users_table.c.email == "alice@example.com")
        .values(age=31)
    )
    conn.commit()

# Delete rows
with engine.connect() as conn:
    conn.execute(
        delete(users_table).where(users_table.c.name == "Bob")
    )
    conn.commit()
```

Core queries are Python expressions. They are composable — you can build a base query and add `.where()`, `.order_by()`, and `.limit()` clauses conditionally without string concatenation.

---

### SQLAlchemy ORM

The ORM maps Python classes to database tables. Each class is a model, each instance is a row. The ORM tracks changes to objects and generates the corresponding INSERT, UPDATE, and DELETE statements.

#### Defining ORM Models

```python
from sqlalchemy import create_engine, String, Integer, ForeignKey
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session

engine = create_engine("sqlite:///orm_example.db", echo=True)


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(100), nullable=False)
    email: Mapped[str] = mapped_column(String(200), nullable=False, unique=True)
    age: Mapped[int | None] = mapped_column(Integer, nullable=True)

    # One user has many posts
    posts: Mapped[list["Post"]] = relationship("Post", back_populates="author")

    def __repr__(self) -> str:
        return f"User(id={self.id!r}, name={self.name!r}, email={self.email!r})"


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    body: Mapped[str] = mapped_column(String, nullable=False)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), nullable=False)

    # Many posts belong to one user
    author: Mapped[User] = relationship("User", back_populates="posts")

    def __repr__(self) -> str:
        return f"Post(id={self.id!r}, title={self.title!r})"


# Create all tables
Base.metadata.create_all(engine)
```

The `Mapped[T]` annotation from modern SQLAlchemy (2.0+) makes the ORM fully type-checkable. A type checker knows that `user.name` is a `str` and `user.age` is `int | None`.

#### Session Management and Full CRUD

The ORM uses a `Session` to manage the unit of work. Think of a session as a staging area: you add objects to it, modify them, and then commit to flush all changes to the database in a single transaction.

```python
from sqlalchemy.orm import Session
from sqlalchemy import select

# --- CREATE ---
with Session(engine) as session:
    alice = User(name="Alice", email="alice@example.com", age=30)
    bob = User(name="Bob", email="bob@example.com", age=25)
    session.add_all([alice, bob])
    session.commit()

    # After commit, the ORM populates auto-generated fields like `id`
    print(f"Alice's id: {alice.id}")

# --- READ ---
with Session(engine) as session:
    # Fetch a single user by primary key
    user = session.get(User, 1)
    print(user)

    # Query with filtering
    statement = select(User).where(User.age >= 25).order_by(User.name)
    results = session.execute(statement).scalars().all()
    for u in results:
        print(u.name, u.email)

    # Load a user along with their posts (eager loading)
    from sqlalchemy.orm import selectinload
    statement = select(User).options(selectinload(User.posts))
    users_with_posts = session.execute(statement).scalars().all()

# --- UPDATE ---
with Session(engine) as session:
    user = session.get(User, 1)
    if user:
        user.age = 31         # Modify the object
        session.commit()      # ORM generates the UPDATE statement

# --- DELETE ---
with Session(engine) as session:
    user = session.get(User, 2)
    if user:
        session.delete(user)
        session.commit()
```

#### Relationships in Practice

```python
with Session(engine) as session:
    # Create a user and a post in one transaction
    alice = session.get(User, 1)
    post = Post(
        title="Getting Started with SQLAlchemy",
        body="SQLAlchemy is the most widely used Python SQL toolkit...",
        author=alice,  # Set the relationship directly
    )
    session.add(post)
    session.commit()

    # Access the relationship
    print(alice.posts)   # [Post(id=1, title='Getting Started with SQLAlchemy')]
    print(post.author)   # User(id=1, name='Alice', email='alice@example.com')
```

### When to Use Core vs. ORM

**Use Core when:**
- You need to write complex queries (window functions, CTEs, raw SQL fragments)
- You are working with an existing database where the schema is fixed
- You want full visibility into the SQL being executed
- Performance is critical and you need to avoid ORM overhead

**Use ORM when:**
- You are building a new application and defining the schema yourself
- You want to work with Python objects rather than rows and columns
- You need relationships and lazy/eager loading
- You want the session to track and batch changes automatically

In practice, you can mix both layers in the same project. The ORM builds on Core, so you can drop down to Core expressions inside ORM queries at any time.

---

## SQLModel — Type-Safe ORM for Modern Python

- Website: [https://sqlmodel.tiangolo.com](https://sqlmodel.tiangolo.com)
- GitHub: [https://github.com/tiangolo/sqlmodel](https://github.com/tiangolo/sqlmodel)

SQLModel was created by Sebastián Ramírez, the same developer who built FastAPI. The motivation is straightforward: in a typical FastAPI application, you end up defining two nearly identical classes — a Pydantic model for request/response validation and a SQLAlchemy model for the database table. SQLModel eliminates that duplication.

A SQLModel class is simultaneously:

- A **SQLAlchemy ORM model** — it can be used as a database table
- A **Pydantic model** — it can validate and serialize JSON data

This is "the best of both worlds": SQLAlchemy's production-proven database layer with Pydantic's ergonomic type safety and validation.

### Installation

```bash
uv add sqlmodel
```

### Defining a Model

```python
from sqlmodel import SQLModel, Field

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)
```

The `table=True` argument tells SQLModel that this class maps to a database table. Without it, the class is a plain Pydantic model (useful for defining request/response shapes that are not stored in the database).

### Creating Tables

```python
from sqlmodel import create_engine

DATABASE_URL = "sqlite:///heroes.db"
engine = create_engine(DATABASE_URL, echo=True)

def create_db_and_tables() -> None:
    SQLModel.metadata.create_all(engine)

if __name__ == "__main__":
    create_db_and_tables()
```

### Full CRUD Example

```python
from sqlmodel import SQLModel, Field, Session, select, create_engine

DATABASE_URL = "sqlite:///heroes.db"
engine = create_engine(DATABASE_URL, echo=False)


class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    secret_name: str
    age: int | None = Field(default=None, index=True)


def create_db_and_tables() -> None:
    SQLModel.metadata.create_all(engine)


# --- CREATE ---
def create_hero(hero: Hero) -> Hero:
    with Session(engine) as session:
        session.add(hero)
        session.commit()
        session.refresh(hero)  # Reload from DB to get generated id
        return hero


# --- READ ---
def get_heroes() -> list[Hero]:
    with Session(engine) as session:
        statement = select(Hero)
        return session.exec(statement).all()


def get_hero(hero_id: int) -> Hero | None:
    with Session(engine) as session:
        return session.get(Hero, hero_id)


def get_heroes_by_age(min_age: int) -> list[Hero]:
    with Session(engine) as session:
        statement = select(Hero).where(Hero.age >= min_age)
        return session.exec(statement).all()


# --- UPDATE ---
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


# --- DELETE ---
def delete_hero(hero_id: int) -> bool:
    with Session(engine) as session:
        hero = session.get(Hero, hero_id)
        if not hero:
            return False
        session.delete(hero)
        session.commit()
        return True


# --- Main ---
if __name__ == "__main__":
    create_db_and_tables()

    # Create heroes
    hero_1 = create_hero(Hero(name="Deadpond", secret_name="Dive Wilson"))
    hero_2 = create_hero(Hero(name="Spider-Boy", secret_name="Pedro Parqueador"))
    hero_3 = create_hero(Hero(name="Rusty-Man", secret_name="Tommy Sharp", age=48))

    print("All heroes:", get_heroes())
    print("Hero 1:", get_hero(hero_1.id))
    print("Heroes aged 40+:", get_heroes_by_age(40))

    # Update
    updated = update_hero_age(hero_3.id, 49)
    print("Updated hero:", updated)

    # Delete
    deleted = delete_hero(hero_2.id)
    print("Deleted hero_2:", deleted)
    print("Remaining heroes:", get_heroes())
```

### The Key Advantage: One Class, Two Roles

The real power of SQLModel becomes apparent in a FastAPI application. Consider this pattern:

```python
from sqlmodel import SQLModel, Field, Session, select
from fastapi import FastAPI, HTTPException

app = FastAPI()


# This class is the DB table AND the Pydantic schema
class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str
    secret_name: str
    age: int | None = None


# A separate schema for creation (no id — it's auto-generated)
class HeroCreate(SQLModel):
    name: str
    secret_name: str
    age: int | None = None


# A separate schema for responses (id is always present)
class HeroRead(SQLModel):
    id: int
    name: str
    age: int | None = None


@app.post("/heroes/", response_model=HeroRead)
def create_hero(hero: HeroCreate) -> Hero:
    # HeroCreate validates the incoming JSON
    # Hero maps it to the database
    db_hero = Hero.model_validate(hero)
    with Session(engine) as session:
        session.add(db_hero)
        session.commit()
        session.refresh(db_hero)
        return db_hero


@app.get("/heroes/", response_model=list[HeroRead])
def read_heroes() -> list[Hero]:
    with Session(engine) as session:
        return session.exec(select(Hero)).all()


@app.get("/heroes/{hero_id}", response_model=HeroRead)
def read_hero(hero_id: int) -> Hero:
    with Session(engine) as session:
        hero = session.get(Hero, hero_id)
        if not hero:
            raise HTTPException(status_code=404, detail="Hero not found")
        return hero
```

In a pure SQLAlchemy + Pydantic project, you would maintain three separate classes for this: a SQLAlchemy ORM model, a Pydantic `HeroCreate` schema, and a Pydantic `HeroRead` schema. SQLModel lets you start from a single `Hero` class and derive the others from it, drastically reducing boilerplate and the risk of the two definitions drifting out of sync.

SQLModel and FastAPI are covered in depth in the next chapter. If you are building a FastAPI application, SQLModel is almost certainly the right database layer to use.

---

## Alembic — Database Migrations

SQLAlchemy's `create_all()` is convenient for getting started, but it has a fundamental limitation: it only creates tables that do not yet exist. If you add a column to a model, `create_all()` will not add that column to your existing database. In a production application, you need database migrations.

[Alembic](https://alembic.sqlalchemy.org/) is SQLAlchemy's official migration tool. It tracks the history of schema changes as a series of versioned migration scripts and can apply (upgrade) or reverse (downgrade) them.

### Installation

```bash
uv add alembic
```

### Initializing Alembic

```bash
# Initialize Alembic in your project directory
alembic init alembic
```

This creates:

```
alembic.ini          # Configuration file
alembic/
    env.py           # Migration environment (edit this to point at your models)
    versions/        # Migration scripts are stored here
```

### Configuring Alembic

Edit `alembic.ini` to point at your database:

```ini
# alembic.ini
sqlalchemy.url = sqlite:///myapp.db
```

Edit `alembic/env.py` to import your models so Alembic can detect schema changes:

```python
# alembic/env.py  (relevant section)
from myapp.models import Base  # Import your DeclarativeBase
target_metadata = Base.metadata
```

### Generating and Applying Migrations

```bash
# Generate a migration by comparing your models to the current database schema
alembic revision --autogenerate -m "create users table"
```

Alembic creates a new file in `alembic/versions/` that looks like this:

```python
"""create users table

Revision ID: a1b2c3d4e5f6
Revises:
Create Date: 2025-01-15 10:00:00.000000
"""
from alembic import op
import sqlalchemy as sa

def upgrade() -> None:
    op.create_table(
        "users",
        sa.Column("id", sa.Integer(), autoincrement=True, nullable=False),
        sa.Column("name", sa.String(length=100), nullable=False),
        sa.Column("email", sa.String(length=200), nullable=False),
        sa.Column("age", sa.Integer(), nullable=True),
        sa.PrimaryKeyConstraint("id"),
        sa.UniqueConstraint("email"),
    )

def downgrade() -> None:
    op.drop_table("users")
```

Apply the migration:

```bash
# Apply all pending migrations
alembic upgrade head

# Roll back one migration
alembic downgrade -1

# View migration history
alembic history --verbose

# Show current database version
alembic current
```

Always review autogenerated migrations before applying them — Alembic is good at detecting additions but can occasionally miss nuances like index renames. Treat migration scripts as code: review them, commit them to version control, and test them.

---

## Comparison Table

| | sqlite3 / DBAPI | SQLAlchemy Core | SQLAlchemy ORM | SQLModel |
|---|---|---|---|---|
| **Abstraction level** | Raw SQL | Query builder | ORM | ORM + Pydantic |
| **Schema definition** | None (write DDL manually) | `Table` + `Column` objects | `DeclarativeBase` classes | `SQLModel` classes |
| **Type safety** | None — rows are plain tuples | Partial — columns are typed | Full — `Mapped[T]` annotations | Full — Pydantic + SQLAlchemy types |
| **Migrations** | Manual | Alembic | Alembic | Alembic |
| **Bundle size / deps** | Zero (stdlib) | `sqlalchemy` only | `sqlalchemy` only | `sqlmodel` (includes SQLAlchemy + Pydantic) |
| **Learning curve** | Low (know SQL already) | Medium | Medium–High | Low–Medium (if you know FastAPI/Pydantic) |
| **Async support** | No | Yes (via `AsyncEngine`) | Yes (via `AsyncSession`) | Partial (beta async support) |
| **Best for** | Scripts, learning, zero-dep tools | Complex queries, existing schemas | Production apps, rich domain models | FastAPI apps, modern codebases |

---

## Which Should You Choose?

**DBAPI / sqlite3** is the right choice when:
- You are writing a small script or command-line tool that does not need external dependencies
- You are learning SQL and want to understand what is actually happening
- You need to embed a simple local database without adding packages

**SQLAlchemy Core** is the right choice when:
- You need to write complex queries that benefit from a programmatic query builder
- You are working with an existing database whose schema you do not control
- You want type-safe query construction without the object-mapping overhead of an ORM
- You need fine-grained control over exactly what SQL is executed

**SQLAlchemy ORM** is the right choice when:
- You are building a production web application or service
- You want to work with objects and relationships rather than rows and columns
- You need the rich ecosystem of extensions, plugins, and examples that SQLAlchemy brings
- You are on a team and value the clarity of well-defined model classes

**SQLModel** is the right choice when:
- You are building a FastAPI application and want one class to serve both as an API schema and a database model
- You are starting a new modern Python project and want type safety from the database through the API layer
- You want to reduce boilerplate and keep your model definitions as DRY as possible

When in doubt, **SQLAlchemy ORM** is the safe default for a backend application. If that application uses FastAPI, switch the default to **SQLModel**.

---

## What's Next?

You now have a solid foundation in Python's database ecosystem: raw SQL via DBAPI, flexible query building with SQLAlchemy Core, production-ready ORM patterns with SQLAlchemy ORM, and the type-safe integration story offered by SQLModel.

The natural next step is to put that database layer to work inside a web API. In the next chapter, we will build a FastAPI application from scratch — defining routes, request and response schemas, and wiring up a SQLModel database layer so that the same model class validates API input and persists data to the database.

[← Previous: LLMs](./11-llms.md) | [Back to README](./README.md) | [Next: FastAPI →](./13-fastapi.md)
