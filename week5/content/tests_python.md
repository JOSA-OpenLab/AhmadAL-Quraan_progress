# pytest Examples: Unit, Integration, and E2E Tests

This file walks through the same small project — a simple "Todo" app with a
database layer and an HTTP API — tested at three different levels using
`pytest` (standard, third party testing library in python). The goal is to show *what changes* as tests get more realistic
(scope, speed, setup cost, confidence).

---

## 0. The code under test

```python
# app/models.py
class Todo:
    def __init__(self, id, title, done=False):
        self.id = id
        self.title = title
        self.done = done

    def mark_done(self):
        self.done = True
        return self
```

```python
# app/repository.py
class TodoRepository:
    """Talks to a real database (or in-memory dict for simplicity here)."""

    def __init__(self, db):
        self.db = db  # e.g. a SQLite/Postgres connection or dict-based fake

    def add(self, todo):
        self.db[todo.id] = todo
        return todo

    def get(self, todo_id):
        return self.db.get(todo_id)

    def complete(self, todo_id):
        todo = self.get(todo_id)
        if todo is None:
            raise ValueError("Todo not found")
        todo.mark_done()
        return todo
```

```python
# app/service.py
class TodoService:
    """Business logic layer — depends on a repository."""

    def __init__(self, repository):
        self.repository = repository

    def create_todo(self, id, title):
        from app.models import Todo
        todo = Todo(id=id, title=title)
        return self.repository.add(todo)

    def complete_todo(self, id):
        return self.repository.complete(id)
```

```python
# app/api.py  (using Flask, but the ideas apply to Django/FastAPI too)
from flask import Flask, jsonify, request
from app.repository import TodoRepository
from app.service import TodoService

def create_app(db=None):
    app = Flask(__name__)
    db = db if db is not None else {}
    repository = TodoRepository(db)
    service = TodoService(repository)

    @app.post("/todos")
    def create_todo():
        data = request.get_json()
        todo = service.create_todo(data["id"], data["title"])
        return jsonify({"id": todo.id, "title": todo.title, "done": todo.done}), 201

    @app.post("/todos/<int:todo_id>/complete")
    def complete_todo(todo_id):
        todo = service.complete_todo(todo_id)
        return jsonify({"id": todo.id, "title": todo.title, "done": todo.done}), 200

    return app
```

---

## 1. Unit Tests

**Scope:** one function/class in isolation. Dependencies are faked/mocked.
**Speed:** milliseconds. **Setup cost:** near zero.

```python
# tests/unit/test_models.py
from app.models import Todo

def test_todo_starts_not_done():
    todo = Todo(id=1, title="Buy milk")
    assert todo.done is False

def test_mark_done_sets_flag_true():
    todo = Todo(id=1, title="Buy milk")
    result = todo.mark_done()
    assert todo.done is True
    assert result is todo  # returns self for chaining
```

```python
# tests/unit/test_service.py
import pytest
from unittest.mock import MagicMock
from app.service import TodoService

def test_create_todo_calls_repository_add():
    fake_repo = MagicMock()
    service = TodoService(repository=fake_repo)

    service.create_todo(id=1, title="Buy milk")

    # We only care that the service talked to the repository correctly —
    # we don't use a real database here.
    fake_repo.add.assert_called_once()
    added_todo = fake_repo.add.call_args[0][0]
    assert added_todo.id == 1
    assert added_todo.title == "Buy milk"

def test_complete_todo_delegates_to_repository():
    fake_repo = MagicMock()
    service = TodoService(repository=fake_repo)

    service.complete_todo(id=5)

    fake_repo.complete.assert_called_once_with(5)
```

```python
# tests/unit/test_repository.py
import pytest
from app.repository import TodoRepository
from app.models import Todo

@pytest.fixture
def repo():
    return TodoRepository(db={})  # in-memory dict = fake DB for a "true" unit test

def test_add_stores_todo(repo):
    todo = Todo(id=1, title="Buy milk")
    repo.add(todo)
    assert repo.get(1) is todo

def test_complete_marks_todo_done(repo):
    repo.add(Todo(id=1, title="Buy milk"))
    completed = repo.complete(1)
    assert completed.done is True

def test_complete_raises_if_not_found(repo):
    with pytest.raises(ValueError):
        repo.complete(999)
```

**Key traits:**
- No real database, no HTTP server.
- `MagicMock` replaces the repository so `TodoService` is tested completely alone.
- Fast enough to run thousands of times per second.

---

## 2. Integration Tests

**Scope:** several real components working together (e.g. service + real
repository + real, disposable database). Mocks are avoided or minimized.
**Speed:** slower (DB I/O), but still reasonably fast. **Setup cost:** medium
(need a real, but isolated, database/fixture).

```python
# tests/integration/conftest.py
import pytest
from app.repository import TodoRepository
from app.service import TodoService

@pytest.fixture
def real_db():
    """
    A 'real' backing store for the test — not mocked. In a Django/SQLAlchemy
    project this would be a real test database created and torn down per test.
    Here we use a plain dict to keep the example runnable without
    extra infrastructure, but the pattern is the same.
    """
    db = {}
    yield db
    db.clear()  # teardown

@pytest.fixture
def service(real_db):
    repository = TodoRepository(real_db)
    return TodoService(repository)
```

```python
# tests/integration/test_todo_workflow.py
import pytest

def test_create_then_complete_todo_end_to_end_in_service_layer(service):
    # Exercises TodoService -> TodoRepository -> real_db together.
    # No mocks: if the repository's SQL/logic is wrong, this test catches it.
    service.create_todo(id=1, title="Buy milk")

    completed = service.complete_todo(id=1)

    assert completed.title == "Buy milk"
    assert completed.done is True

def test_completing_unknown_todo_raises(service):
    with pytest.raises(ValueError):
        service.complete_todo(id=42)
```

If this were Django + `pytest-django`, the same idea looks like:

```python
# tests/integration/test_todo_workflow_django.py
import pytest
from myapp.models import Todo

@pytest.mark.django_db  # uses a REAL test database, wrapped in a transaction
def test_create_then_complete_todo(django_user_model):
    todo = Todo.objects.create(title="Buy milk")
    todo.mark_done()
    todo.save()

    refreshed = Todo.objects.get(id=todo.id)
    assert refreshed.done is True
```

**Key traits:**
- Real database (or realistic fake), real repository — only the outer world
  (e.g. network calls to third parties) might still be mocked.
- Catches bugs at the "seams" between components that unit tests, with all
  their mocks, would miss.

---

## 3. End-to-End (E2E) Tests

**Scope:** the whole system through its real, external interface — an actual
HTTP request/response cycle here, hitting real routes, real serialization,
real status codes. **Speed:** slowest of the three. **Setup cost:** highest
(spin up the app, possibly a browser or test client).

```python
# tests/e2e/conftest.py
import pytest
from app.api import create_app

@pytest.fixture
def client():
    app = create_app(db={})  # fresh app + fresh "database" per test
    app.config["TESTING"] = True
    with app.test_client() as client:
        yield client
```

```python
# tests/e2e/test_todo_api.py
def test_create_todo_via_http_api(client):
    response = client.post("/todos", json={"id": 1, "title": "Buy milk"})

    assert response.status_code == 201
    body = response.get_json()
    assert body == {"id": 1, "title": "Buy milk", "done": False}

def test_complete_todo_via_http_api(client):
    client.post("/todos", json={"id": 1, "title": "Buy milk"})

    response = client.post("/todos/1/complete")

    assert response.status_code == 200
    body = response.get_json()
    assert body["done"] is True

def test_full_user_journey_create_then_complete(client):
    # Simulates an actual client using the API end-to-end, no internals touched.
    create_resp = client.post("/todos", json={"id": 2, "title": "Walk the dog"})
    assert create_resp.status_code == 201

    complete_resp = client.post("/todos/2/complete")
    assert complete_resp.status_code == 200
    assert complete_resp.get_json()["done"] is True
```

For a browser-level E2E test (e.g. testing a real frontend + backend
together), you'd typically pair pytest with **Playwright** or **Selenium**:

```python
# tests/e2e/test_browser_flow.py
import pytest
from playwright.sync_api import Page, expect

def test_user_can_create_and_complete_todo_in_browser(page: Page):
    page.goto("http://localhost:5000")

    page.fill("#todo-title", "Buy milk")
    page.click("#add-todo-button")

    todo_item = page.locator(".todo-item", has_text="Buy milk")
    expect(todo_item).to_be_visible()

    todo_item.locator(".complete-checkbox").check()
    expect(todo_item).to_have_class("todo-item done")
```

**Key traits:**
- Talks to the system exactly the way a real client (or user) would —
  through HTTP, or through an actual rendered browser page.
- Nothing internal is mocked; if the whole stack (routing, serialization,
  business logic, database) doesn't genuinely work together, the test fails.
- Slowest and most brittle to maintain — hence, use these sparingly, for the
  most critical user flows.

---

## 4. Running Each Layer Separately

A common convention is to organize tests into folders by type and give
pytest markers so you can run subsets independently:

```ini
# pytest.ini
[pytest]
markers =
    unit: fast, isolated tests
    integration: tests touching a real DB/service
    e2e: full end-to-end tests through the real interface
```

```bash
tests/
├── unit/
├── integration/
└── e2e/
```

```bash
# Run only unit tests (fast feedback loop during development)
pytest tests/unit

# Run only integration tests
pytest tests/integration

# Run only E2E tests (e.g. in a separate, slower CI stage)
pytest tests/e2e

# Run everything
pytest
```

---

## 5. Summary Table

| Layer       | What's real                          | What's faked/mocked        | Speed  | Example here                          |
|-------------|----------------------------------------|------------------------------|--------|----------------------------------------|
| Unit        | The single function/class under test   | Everything it depends on     | ⚡ fastest | `TodoService` with a `MagicMock` repo |
| Integration | Service + repository + real/test DB    | External third-party APIs    | 🐢 medium  | `TodoService` + real dict/DB           |
| E2E         | Entire app, through its real interface | Nothing internal             | 🐌 slowest | HTTP requests via Flask test client / Playwright browser |

This mirrors the cost/coverage trade-off from the Test Pyramid / Testing
Trophy discussion: as you move down this table, tests get slower and more
expensive to write and maintain, but give you more genuine confidence that
the system works as a whole.
