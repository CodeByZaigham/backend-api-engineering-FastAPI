<p align="center">
  <img src="Screenshots/topics covered.png" alt="topics covered picture" width="100%">
</p>
<div align="center">

# ⚙️ Backend API Engineering with FastAPI

### Ten hands-on modules covering the building blocks of a production backend: auth, RBAC, rate limiting, caching, background jobs and more.

*Each topic is a small, runnable service built to explain one concept in depth. They are the foundation for a full production system.*

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-async%20API-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Pydantic](https://img.shields.io/badge/Pydantic-validation-E92063?style=for-the-badge&logo=pydantic&logoColor=white)](https://docs.pydantic.dev/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-D71F00?style=for-the-badge)](https://www.sqlalchemy.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-pooled-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-cache%20%26%20broker-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![Celery](https://img.shields.io/badge/Celery-task%20queue-37814A?style=for-the-badge&logo=celery&logoColor=white)](https://docs.celeryq.dev/)
[![JWT](https://img.shields.io/badge/JWT-HS256-000000?style=for-the-badge&logo=jsonwebtokens&logoColor=white)](https://jwt.io/)
[![Argon2](https://img.shields.io/badge/Argon2-password%20hashing-6E4AFF?style=for-the-badge)](https://github.com/frankie567/pwdlib)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

</div>

---

## Overview

Building an API that returns JSON is easy. Building one that is **secure, rate-limited, cached, observable, and doesn't fall over when a task takes ten seconds** is backend engineering.

This repository is my topic-by-topic study of that second problem. Every folder is a **self-contained FastAPI service** that isolates one production concern, keeps the code small enough to read in a few minutes, and runs on its own. The modules follow the order I learned them in, so they read as a progression from a single route to a cached, queued, role-protected API.

The plan, stated openly: cover each advanced topic in its own mini-project, then **combine everything into one production-grade system** (see [Roadmap](#roadmap)).

| Concern | Module | What it demonstrates |
|---|---|---|
| **Foundations** | `routes creation` | Routes, HTTP methods, Pydantic request validation |
| **Persistence** | `postgreSQL connection` | SQLAlchemy ORM, table creation, FK relations, **connection pooling**, `Depends` |
| **Authentication** | `JWT authentication` | Argon2 password hashing, OAuth2 password flow, signed JWTs with expiry |
| **Machine access** | `API KEY Security` | Header-based API keys with `Security` / `APIKeyHeader` |
| **Authorization** | `Oauth2 Authorization (role based)` | Bearer-token dependency, role claims, an admin-only endpoint |
| **Request pipeline** | `Middleware and CORS` | Custom HTTP middleware (timing, logging), CORS policy |
| **Error contract** | `Exception Handling` | Custom exception classes and global handlers with consistent JSON bodies |
| **Abuse protection** | `Rate Limiting` | Per-client fixed-window limiter with rate-limit headers and `429` responses |
| **Performance** | `Redis caching` | Cache-aside pattern, app lifespan, Redis client wiring |
| **Async work** | `Background tasks (Celery + Redis)` | Task queue, separate broker and result backend, task-status polling |

---

## Table of Contents

- [Highlights](#highlights)
- [Learning Path](#learning-path)
- [Where Each Module Fits](#where-each-module-fits)
- [Module Deep Dives](#module-deep-dives)
- [Concepts Demonstrated](#concepts-demonstrated)
- [Design Decisions](#design-decisions)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Known Gaps and Hardening Backlog](#known-gaps-and-hardening-backlog)
- [Roadmap](#roadmap)
- [License](#license)

---

## Highlights

- **Ten independent services, one concern each.** Every module runs with a single `uvicorn` command and can be read in isolation.
- **A complete authentication path.** Argon2 password hashing (`pwdlib`), OAuth2 password flow, and HS256 JWTs with a 30-minute expiry.
- **Token-based authorization.** A reusable `Depends` that decodes the bearer token and exposes `userid` and `role` to any route.
- **Three layers of API protection:** API keys for machine clients, JWT + roles for users, and a rate limiter for abuse.
- **Connection pooling done deliberately.** SQLAlchemy engine with `pool_size=10` and `max_overflow=5`, and a `get_db` dependency that always closes its session.
- **A global error contract.** Domain exceptions map to structured JSON responses through registered handlers instead of ad hoc `try/except` in routes.
- **Cache-aside with Redis** on an admin endpoint, wired through the FastAPI lifespan.
- **Real background processing.** Celery workers consume from a Redis broker and write to a separate Redis result backend, and a status endpoint polls `AsyncResult`.
- **Honest about its limits.** A dedicated [hardening backlog](#known-gaps-and-hardening-backlog) lists what separates each demo from production.

---

## Learning Path

The order the modules were built, with each one adding a layer on top of the last:

```mermaid
flowchart LR
    A["1 · routes creation<br/>routes, schemas"] --> B["2 · postgreSQL connection<br/>ORM, pooling, Depends"]
    B --> C["3 · JWT authentication<br/>hashing, tokens"]
    C --> D["4 · API KEY Security<br/>header keys"]
    D --> E["5 · Role-based authorization<br/>bearer dependency, RBAC"]
    E --> F["6 · Middleware and CORS<br/>timing, logging, origins"]
    F --> G["7 · Exception Handling<br/>global error contract"]
    G --> H["8 · Rate Limiting<br/>slowapi, 429"]
    H --> I["9 · Redis caching<br/>cache-aside"]
    I --> J["10 · Celery + Redis<br/>background jobs"]
```

---

## Where Each Module Fits

The modules are separate demos today. This diagram shows where each one sits in the request lifecycle of the production API they are building toward.

```mermaid
flowchart TD
    C(["Client"]) --> CORS["CORS policy<br/>Middleware and CORS"]
    CORS --> MW["Timing and logging middleware<br/>Middleware and CORS"]
    MW --> RL["Rate limiter<br/>Rate Limiting"]
    RL --> AUTH{"Who is calling?"}
    AUTH -->|"machine client"| KEY["API key check<br/>API KEY Security"]
    AUTH -->|"signed-in user"| JWT["JWT decode + role check<br/>JWT and Role-based Authorization"]
    KEY --> H["Route handler<br/>routes creation, Pydantic"]
    JWT --> H
    H --> CACHE{"Cached?"}
    CACHE -->|"hit"| R[("Redis<br/>Redis caching")]
    CACHE -->|"miss"| DB[("PostgreSQL via pooled session<br/>postgreSQL connection")]
    H -.->|"slow work"| Q["Celery task over Redis<br/>Background tasks"]
    H -.->|"failure"| EX["Global exception handlers<br/>Exception Handling"]
```

---

## Module Deep Dives

Every module is run from **inside its own folder**. See [Getting Started](#getting-started).

### 1 · Routes Creation
📁 `routes creation/`

The minimal FastAPI app: a health route and a `POST` that validates its body with a Pydantic model (`name: str`, `age: int`, `is_engineer: bool`). FastAPI rejects malformed input with a `422` before your function runs.

---

### 2 · PostgreSQL Connection
📁 `postgreSQL connection/`

A small quiz API (questions with multiple choices) that introduces the data layer.

```python
engine = create_engine(url=url, pool_size=10, max_overflow=5)     # connection pooling
session = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def get_db():
    db = session()
    try:
        yield db          # one session per request
    finally:
        db.close()        # always returned to the pool
```

- **ORM models** (`Questions`, `Choices`) with a foreign key, and tables created from Python via `Base.metadata.create_all`.
- **Connection pooling:** up to 10 persistent connections plus 5 overflow, so requests reuse connections instead of paying for a new one each time.
- **`Depends(get_db)`** injects a request-scoped session.
- `POST /questions` stores a question and its choices. `GET /get_questions/{id}` returns both, or a `404` if the id doesn't exist.

---

### 3 · JWT Authentication
📁 `JWT authentication/`

User signup and login.

- **`POST /createuser`** hashes the password with Argon2 (`PasswordHash.recommended()`) and stores the user. Plain-text passwords are never persisted.
- **`POST /login`** takes the standard `OAuth2PasswordRequestForm` (so Swagger's **Authorize** button works), verifies the hash, and returns a JWT.
- Tokens carry `sub` (user id), `role`, and an `exp` claim set to 30 minutes.

```mermaid
sequenceDiagram
    participant C as Client
    participant A as FastAPI
    participant D as PostgreSQL
    C->>A: POST /login (username, password)
    A->>D: fetch user by username
    D-->>A: user + Argon2 hash
    A->>A: verify password against hash
    A-->>C: JWT {sub, role, exp = now + 30 min}
    Note over C,A: Client sends "Authorization: Bearer <JWT>" on later requests
```

---

### 4 · API Key Security
📁 `API KEY Security/`

Protects an endpoint for machine-to-machine clients. An `APIKeyHeader(name="x-api-key")` dependency reads the header and compares it to the configured secret.

| Request | Result |
|---|---|
| Valid `x-api-key` header | `200`, `{"message": "Authorized"}` |
| Missing or wrong key | `401` |

The dependency is declared with `Security(...)`, so Swagger UI shows the lock icon and an **Authorize** dialog for the key.

---

### 5 · OAuth2 Authorization (Role-Based)
📁 `Oauth2 Authorization (role based)/`

Builds on module 3 by adding **authorization**: not just *who are you*, but *what may you do*.

```python
Oauth2_token = OAuth2PasswordBearer(tokenUrl="login")

def decode_jwt(token: str = Depends(Oauth2_token)) -> tokendata:
    payload = jwt.decode(token, encryption_key, encryption_algorithm)
    return tokendata(userid=int(payload["sub"]), role=payload["role"])
```

- `decode_jwt` is a reusable dependency: any route that declares it gets the verified `userid` and `role`.
- **`GET /userlist`** is an admin-only endpoint. It checks the `admin` role, confirms the admin account is not disabled, then returns the list of regular users through a response model that never includes password hashes.
- Without a token the endpoint answers `401`. The server log below shows a `401` on `/userlist`, then a successful `POST /login`, then the same call being served.

<img src="Oauth2%20Authorization%20%28role%20based%29/screenshots/authorization%20logs.PNG" alt="Server log: GET /userlist returns 401 before login, then 200 after POST /login" width="70%">

---

### 6 · Middleware and CORS
📁 `Middleware and CORS/`

- **`CORSMiddleware`** configured with origins, headers, methods and credentials. It uses permissive values for demonstration, and comments in the code note what to tighten for production.
- **Custom HTTP middleware** that wraps every request: it logs the path, method and JSON body, times the downstream call with `perf_counter`, and prints the duration. This is the pattern behind request logging, tracing IDs and metrics.

---

### 7 · Exception Handling
📁 `Exception Handling/`

A layered error contract. Domain exceptions live in `exceptions/custom_exceptions.py`, their HTTP translations live in `exceptions/handlers.py`, and `main.py` registers them. Routes just `raise`.

| Route | Result |
|---|---|
| `GET /users/0026` | `200`, user found |
| `GET /users/{other}` | `404`, `{"error": "User not found", "user_id": "..."}` |
| `GET /test-database` | `500`, `{"error": "Database error"}` |
| `GET /test-http` | `404`, `{"error": "HTTP error", "detail": "..."}` |

The point: clients always receive the same JSON shape, and internals never leak into responses.

---

### 8 · Rate Limiting
📁 `Rate Limiting/`

Per-client throttling with **slowapi**: a fixed-window limiter keyed by remote address, applied to a route with `@limiter.limit("2/minute")` and wired through `SlowAPIMiddleware`.

- Calls 1 and 2 within a minute return `200`.
- Call 3 returns **`429`** with a JSON error body.
- `X-RateLimit-*` headers are included on responses (`headers_enabled=True`).
- The limiter is defined once in `limiter.py` and shared, and the storage URI is a single line to swap to Redis for multi-worker deployments.

---

### 9 · Redis Caching
📁 `Redis caching/`

Applies the **cache-aside pattern** to the admin user list from module 5.

```mermaid
flowchart LR
    R["GET /userlist<br/>(admin token)"] --> K{"Redis key<br/>'userlist' present?"}
    K -->|"yes"| H["Return cached JSON"]
    K -->|"no"| Q["Query PostgreSQL"]
    Q --> S["Store JSON in Redis"]
    S --> O["Return result"]
```

- The Redis client is created from environment settings and attached to `app.state` in a **`lifespan`** context manager, which also creates tables on startup and closes the client on shutdown.
- The authentication and authorization code is identical to module 5. Only the caching layer is new.

---

### 10 · Background Tasks (Celery + Redis)
📁 `Background tasks (Celery + Redis)/`

Moves slow work out of the request/response cycle so the API stays responsive.

```mermaid
flowchart LR
    A["FastAPI<br/>POST /process"] -->|".delay()"| B[("Redis broker<br/>db 1")]
    B --> C["Celery worker<br/>process_task()"]
    C --> D[("Redis result backend<br/>db 2")]
    E["FastAPI<br/>GET /task/{task_id}"] -->|"AsyncResult"| D
```

- `POST /process` enqueues a task and **immediately** returns a `task_id`.
- `GET /task/{task_id}` reports the task's status and returns its result once it has succeeded.
- Broker and result backend are on separate Redis databases, and tasks use JSON serialization only.
- The sample task sleeps for 10 seconds to make the asynchronous behavior obvious.
- A note in `tasks.py` records a real lesson: Celery tasks are synchronous, so async code needs to be bridged with `asyncio.run(...)`.

---

## Concepts Demonstrated

| Concept | Where to find it |
|---|---|
| Dependency injection with `Depends` (request-scoped DB session) | `postgreSQL connection/main.py`, `*/database/database.py` |
| Pydantic request and response models | every `schema.py` |
| SQLAlchemy ORM models, FKs, `create_all` | `*/database/tables.py` |
| **Connection pooling** (`pool_size`, `max_overflow`) | `*/database/database.py` |
| Password hashing with Argon2 | `JWT authentication/authentication.py` |
| OAuth2 password flow (`OAuth2PasswordRequestForm`) | `JWT authentication/main.py` |
| JWT creation with expiry, decoding with signature check | `authentication.py`, `authorization.py` |
| Bearer-token dependency (`OAuth2PasswordBearer`) | `Oauth2 Authorization (role based)/authorization.py` |
| Role-based access control | `Oauth2 Authorization (role based)/main.py` → `/userlist` |
| API-key security scheme (`APIKeyHeader` + `Security`) | `API KEY Security/` |
| CORS policy and custom HTTP middleware | `Middleware and CORS/main.py` |
| Global exception handlers and a custom error contract | `Exception Handling/` |
| Rate limiting (fixed window, per IP, headers, 429) | `Rate Limiting/` |
| App `lifespan` for startup and shutdown resources | `Redis caching/main.py` |
| Cache-aside with Redis | `Redis caching/main.py` |
| Task queue with broker + result backend, status polling | `Background tasks (Celery + Redis)/` |

---

## Design Decisions

| Decision | Reasoning |
|---|---|
| **One folder = one concept, runnable alone** | Isolation makes each idea easy to learn, test and reuse, and nothing hides behind unrelated code. |
| **Request-scoped session via a `yield` dependency** | Guarantees the session is closed (and the connection returned to the pool) even when a route raises. |
| **Pool tuned explicitly** | Pooling avoids per-request connection cost and caps total DB connections, which protects PostgreSQL under load. |
| **Argon2 for passwords** | A memory-hard hash is far more resistant to GPU cracking than fast general-purpose hashes. |
| **Short-lived tokens (30 min)** | Limits the damage window of a leaked token. |
| **Role carried as a signed claim** | The role check needs no extra DB read to decide *what class of user this is*, and the signature stops tampering. |
| **Authorization as a dependency** | Any route opts in by declaring one parameter, and the logic lives in one place. |
| **Handlers instead of `try/except` in routes** | Error shape and status codes stay consistent and routes stay readable. |
| **Rate limiter in its own module** | One shared limiter instance, easy to move from memory to Redis storage. |
| **Separate Redis DBs for broker and results** | Keeps task messages and task results from sharing a keyspace. |
| **Celery for slow work** | The request returns instantly and the heavy job survives across workers and restarts. |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Web framework | **FastAPI** + **Uvicorn** |
| Validation | **Pydantic** |
| ORM and database | **SQLAlchemy** + **PostgreSQL** (`psycopg2-binary`) |
| Password hashing | **pwdlib** with Argon2 |
| Tokens | **PyJWT** (HS256) |
| Forms | `python-multipart` (OAuth2 login form) |
| Rate limiting | **slowapi** |
| Cache and broker | **Redis** |
| Background jobs | **Celery** |
| Config | `python-dotenv` |

---

## Project Structure

```
backend-api-engineering/
├── routes creation/                      # 1  routes + Pydantic
├── postgreSQL connection/                # 2  ORM + connection pooling
│   └── database/
├── JWT authentication/                   # 3  hashing + JWT login
│   └── database/
├── API KEY Security/                     # 4  x-api-key header auth
├── Oauth2 Authorization (role based)/    # 5  bearer dependency + RBAC
│   ├── database/
│   └── screenshots/
├── Middleware and CORS/                  # 6  middleware + CORS
├── Exception Handling/                   # 7  custom exceptions + handlers
│   └── exceptions/
├── Rate Limiting/                        # 8  slowapi limiter
├── Redis caching/                        # 9  cache-aside on top of module 5
│   └── database/
├── Background tasks (Celery + Redis)/    # 10 Celery worker + status API
├── screenshots/
├── topics covered.txt
├── requirements.txt
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.11+
- **PostgreSQL** (modules 2, 3, 5, 9) and **Redis** (modules 9, 10)

The quickest way to get both is Docker:

```bash
docker run -d --name pg    -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:16
docker run -d --name redis -p 6379:6379 redis:7
```

### 1 · Install

```bash
git clone https://github.com/CodeByZaigham/backend-api-engineering.git
cd backend-api-engineering

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

### 2 · Configure

Create a `.env` file in the **repository root**. Every module finds it by searching upward from its own folder.

```env
DATABASE_URL=postgresql+psycopg2://postgres:postgres@localhost:5432/postgres
SECRET_KEY=generate_with_the_command_below
ALGORITHM=HS256
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
```

Generate a strong secret:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

### 3 · Run a module

Each module uses flat imports, so **run it from inside its folder**:

```bash
cd "Oauth2 Authorization (role based)"
uvicorn main:app --reload
```

Then open `http://127.0.0.1:8000/docs`. Swagger UI is where you can sign up, log in with the **Authorize** button, and call protected routes.

**Celery module** needs a worker alongside the API, in two terminals:

```bash
cd "Background tasks (Celery + Redis)"

celery -A Celeryfile worker --loglevel=info --pool=solo    # terminal 1: the worker
uvicorn main:app --reload                                  # terminal 2: the API
```

`--pool=solo` is the simplest option for local development, especially on Windows.

---

## Configuration

| Variable | Used by | Purpose |
|---|---|---|
| `DATABASE_URL` | modules 2, 3, 5, 9 | SQLAlchemy connection string for PostgreSQL |
| `SECRET_KEY` | modules 3, 4, 5, 9 | JWT signing secret (modules 3, 5, 9) or the accepted API key (module 4) |
| `ALGORITHM` | modules 5, 9 | JWT algorithm (`HS256`). Module 3 hard-codes it |
| `REDIS_HOST`, `REDIS_PORT`, `REDIS_DB` | module 9 | Redis connection for the cache |
| Redis broker and backend | module 10 | Set in `Celeryfile.py`: `localhost:6379`, DB 1 (broker) and DB 2 (results) |

| Tunable | Where | Value |
|---|---|---|
| Pool size / overflow | `*/database/database.py` | `10` / `5` |
| JWT lifetime | `*/main.py` → `/login` | 30 minutes |
| Rate limit | `Rate Limiting/main.py` | `2/minute`, fixed window, per IP |

---

## Known Gaps and Hardening Backlog

These modules are learning demos, so they intentionally stop short of production. This is what separates them from the finished system I'm building toward.

**Security**

- **Roles are accepted from the signup payload.** In production, roles are assigned server-side, never chosen by the client.
- **Credential and token failures should be clean `401`/`403` responses.** Bad credentials, a malformed token, and a non-admin hitting an admin route currently surface as unhandled errors instead of proper client errors.
- **API keys should be compared in constant time** (`secrets.compare_digest`) and use their own variable rather than sharing the JWT `SECRET_KEY` name.
- **The middleware logs full request bodies**, which would include credentials. Fine for a demo, not for production.
- **CORS is wide open for demonstration** (`*` with credentials). Production needs an explicit origin list.

**Architecture**

- **Redis cache has no TTL and no invalidation.** The cached user list can go stale after new signups.
- **Rate-limit storage is in-memory**, so limits are per process. Redis storage is needed for multiple workers.
- **Schema changes use `create_all`**, not migrations.
- **Database access is synchronous** with SQLAlchemy's legacy `Query` API. No async I/O yet.
- **The quiz endpoint returns the `is_true` flag** for each choice, so a client-facing quiz would need a response model that hides it.
- **No automated tests yet.**

---

## Roadmap

The repository description lists where this is headed. Here it is as a checklist:

- [ ] **Capstone:** one production-grade service that composes every module above
- [ ] **Service-layer architecture:** routers → services → repositories, with dependency boundaries
- [ ] **Async I/O:** async SQLAlchemy with `asyncpg`
- [ ] **Alembic migrations** in place of `create_all`
- [ ] **Redis-backed rate limiting** and **cache TTL + invalidation**
- [ ] **Refresh tokens** and token revocation
- [ ] **Structured logging** and request IDs
- [ ] **Observability:** metrics and tracing
- [ ] **Query optimization:** indexes, N+1 avoidance, `EXPLAIN`
- [ ] **Testing:** `pytest` with FastAPI's `TestClient` and a test database
- [ ] **Docker Compose** for API + PostgreSQL + Redis + Celery worker
- [ ] **CI pipeline** running tests and linting

---

## License

Released under the [MIT License](LICENSE).

---

<div align="center">

**Built by [CodeByZaigham](https://github.com/CodeByZaigham)**

*Learning backend engineering in public, one concern at a time.*

</div>