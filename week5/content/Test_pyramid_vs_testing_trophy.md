## Test Pyramid vs Testing Trophy: Cost/Coverage Trade-offs

Both are mental models for how to allocate testing effort across test types. Both answer the same underlying question:

> As tests get more realistic, they get slower and more expensive to write/maintain, but give more confidence the whole system works. Where should most of the effort go?

### The Test Pyramid (classic model, Mike Cohn)

```
        /\
       /  \      E2E / UI Tests (few)
      /----\
     /      \    Integration Tests (some)
    /--------\
   /          \  Unit Tests (many)
  /____________\
```

- **Unit tests** — cheap, fast (milliseconds), pinpoint exact bug location, but isolated so they don't verify components work *together*.
- **Integration tests** — cost more (slower, more setup, e.g. a real database), but catch issues at component boundaries.
- **E2E tests** — highest confidence (simulate real user flows), but slow, brittle, flaky, and expensive to maintain — so use very few.

**Recommendation:** spend most of your limited testing budget on the cheap, fast, reliable unit-test layer.

### The Testing Trophy (newer model, Kent C. Dodds)

```
        ___
       /   \      E2E (few)
      |-----|
      |     |
      |     |     Integration Tests (biggest chunk)
      |     |
      |-----|
       \   /      Unit Tests (some)
        \ /
         |        Static Analysis (types, linting — the base)
        _|_
```

- Integration tests give the **best cost/confidence ratio**, so they get the biggest investment — not unit tests.
- **Why the shift:**
  - Static analysis (type checkers, linters) now catches many trivial bugs for free.
  - Pure unit tests that mock everything can pass while the *real* system is broken — mocks don't always match reality, creating false confidence.
  - Integration tests (real units working together, e.g., service layer + real test database) catch bugs that matter, at the seams, while staying reasonably fast and less brittle than full E2E.

### Core trade-off table

| Test type   | Speed      | Cost to write/maintain | Confidence given         |
|-------------|------------|-------------------------|---------------------------|
| Unit        | Very fast  | Cheap                   | Low–medium (isolated)     |
| Integration | Medium     | Medium                  | High                       |
| E2E         | Slow       | Expensive, brittle      | Highest, but flaky         |

- **Pyramid's answer:** bulk of effort → unit tests (cheapest, fastest).
- **Trophy's answer:** bulk of effort → integration tests (best confidence-per-dollar).

