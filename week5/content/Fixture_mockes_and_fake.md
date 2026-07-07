# Fixtures, mocks and fakes

Three tools that often get confused because they all "help set up tests," but they solve different problems. Knowing which one to reach for is a big part of writing tests that stay useful instead of becoming brittle.

---

## 1. Fixtures

**What it is:** Pre-built state or data that a test needs, provided by the test framework so you don't repeat setup code across tests.

**Job:** Handle the *Arrange* step of Arrange/Act/Assert. A fixture doesn't check anything and doesn't pretend to be something else — it just hands you ready-made input.

### Example (pytest)

```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "Alice", "email": "alice@example.com"}

def test_user_has_email(sample_user):
    assert sample_user["email"] == "alice@example.com"

def test_user_has_name(sample_user):
    assert sample_user["name"] == "Alice"
```

Both tests reuse `sample_user` instead of redefining the dict each time.

### Fixtures can also manage setup *and* teardown

```python
@pytest.fixture
def db_connection():
    conn = connect_to_test_db()
    yield conn          # <- test runs here
    conn.close()         # <- teardown, runs after the test finishes
```

Anything before `yield` runs before the test; anything after runs after, even if the test fails. This is how fixtures replace manual `setUp`/`tearDown` boilerplate.

### Fixture scope

Fixtures can be re-created per test, or shared:

```python
@pytest.fixture(scope="function")  # default — new instance per test
@pytest.fixture(scope="module")    # shared across all tests in a file
@pytest.fixture(scope="session")   # shared across the entire test run
```

Use broader scopes for expensive setup (e.g. spinning up a database container once), and function scope when tests need a clean, isolated state (most common default).

### Key point
A fixture is about **supplying state**, not verifying behavior. It has no opinion about what your test asserts.

---

## 2. Mocks

**What it is:** A fake object that doesn't do real work — it just *records how it was used* (was it called, how many times, with what arguments), so you can assert on the interaction itself.

**Job:** Let you test "did my code call the right thing correctly?" without needing the real dependency (e.g. a real email server, payment gateway, or API).

### Example

```python
from unittest.mock import Mock

def send_welcome_email(user, email_sender):
    email_sender.send(user["email"], "Welcome!")

def test_welcome_email_sent():
    # Arrange
    mock_sender = Mock()
    user = {"email": "alice@example.com"}

    # Act
    send_welcome_email(user, mock_sender)

    # Assert
    mock_sender.send.assert_called_once_with("alice@example.com", "Welcome!")
```

`mock_sender` never actually sends anything. It just remembers that `.send()` was called once, with those exact arguments.

### Why mocks are useful

- Fast — no real network/database/service needed.
- Precise — great for asserting a specific interaction must happen exactly once (e.g. "we must not charge a customer's card twice").
- Isolates the unit under test from external systems entirely.

### Why mocks are "coupled" (the main downside)

A mock test has hardcoded knowledge of *how* the code calls its dependency: the method name, argument order, number of calls. If you refactor `EmailSender` to use `.deliver()` instead of `.send()`, or reorder arguments — even if the actual observable behavior (an email gets sent) hasn't changed — the mock-based test breaks.

This is sometimes called testing **implementation detail** rather than **behavior/outcome**. Overusing mocks can lead to a test suite that breaks on every refactor, even correct ones, which erodes trust in the tests.

### Rule of thumb
Use mocks when the *interaction itself* is the thing you care about — e.g. "we must call `payment_gateway.charge()` exactly once."

---

## 3. Fakes

**What it is:** A simplified, but *actually working*, implementation of a dependency. Often in-memory instead of hitting a real external system.

**Job:** Let your test exercise real logic (state changes, computations, lookups) without needing real infrastructure.

### Example

```python
class FakeEmailSender:
    def __init__(self):
        self.sent_emails = []

    def send(self, to, subject):
        self.sent_emails.append((to, subject))  # actually stores it

def test_welcome_email_sent_fake():
    # Arrange
    fake_sender = FakeEmailSender()
    user = {"email": "alice@example.com"}

    # Act
    send_welcome_email(user, fake_sender)

    # Assert
    assert ("alice@example.com", "Welcome!") in fake_sender.sent_emails
```

Compared to the mock version, `FakeEmailSender` genuinely *behaves* like an email sender: it has real internal state (a list of sent emails) and you can query it in realistic ways, not just check "was this called."

### Classic real-world example: in-memory database

```python
class FakeUserRepository:
    def __init__(self):
        self._users = {}

    def save(self, user):
        self._users[user["id"]] = user

    def get(self, user_id):
        return self._users.get(user_id)
```

You can run real queries against this: save a user, then fetch it, and check the actual returned data — testing real logic instead of just recording calls. This is often preferred over mocking every single database method, because it validates actual behavior (e.g., "if I save then get, do I get the same data back?").

### Rule of thumb
Use fakes when you care about *outcomes/behavior* — e.g. "if I add two items to the cart, is the total correct?" — and would rather not spin up real infrastructure per test.

---

## Side-by-side comparison

| | Fixture | Mock | Fake |
|---|---|---|---|
| **Purpose** | Supplies test data/state | Records interactions | Simulates real behavior |
| **Does real work?** | N/A (just data) | No — just a recorder | Yes — simplified but functional |
| **What it checks** | Nothing itself | "Was X called correctly?" | "Does the real workflow produce the right result?" |
| **Breaks when...** | Rarely, unless setup logic changes | Implementation details change (method names, arg order, call count) | Only when the actual contract/behavior changes |
| **Best for** | Providing reusable test input | Asserting critical one-time interactions (e.g. "charge card exactly once") | Testing real logic without real infrastructure (e.g. in-memory DB) |
| **Realism** | N/A | Low | Higher |

---

## How they fit together

These three aren't mutually exclusive — in practice they combine:

```python
@pytest.fixture
def fake_email_sender():
    return FakeEmailSender()

def test_welcome_email_sent(fake_email_sender):
    # Arrange
    user = {"email": "alice@example.com"}

    # Act
    send_welcome_email(user, fake_email_sender)

    # Assert
    assert ("alice@example.com", "Welcome!") in fake_email_sender.sent_emails
```

Here, a **fixture** is used to *supply* a **fake** to the test. This is an extremely common pattern: fixtures are the delivery mechanism, mocks/fakes are what gets delivered.

---

## Quick decision guide

- Need reusable test data/setup? → **Fixture**
- Need to verify a specific call happened (and don't care about deeper logic)? → **Mock**
- Need to test real logic/behavior without real infrastructure? → **Fake**

When in doubt, prefer **fakes over mocks** for anything beyond a trivial interaction check — they make tests more resilient to refactors and closer to testing real behavior rather than internal wiring.

## Related
- [[Testing Overview]]
- [[pytest]]
- [[Arrange Act Assert]]
