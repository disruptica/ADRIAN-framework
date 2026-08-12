# CLAUDE.md Templates

A `CLAUDE.md` file lives at the root of every repo. It is read by Claude Code (and similar agents) before the agent acts. It is the single highest-leverage artifact for fighting architectural drift and enforcing house style.

**The compounding loop:** When drift shows up in code review, the fix goes into `CLAUDE.md`, not into the reviewer's head. Over time, the rules accumulate and the agent gets better at producing house-style code on the first attempt.

Three templates follow, one per language archetype. Copy, fill in, commit.

---

## How to use these templates

1. Copy the relevant template into your repo as `CLAUDE.md` (or `AGENTS.md` for tool neutrality)
2. Fill in the `<...>` placeholders
3. Add stack-specific "never do" rules from your house style
4. Add a compliance section if the repo runs in a regulated mode
5. Commit it. Update it whenever a Weekly Quality Review surfaces a pattern.

**Test the rules.** Once committed, ask the agent to deliberately violate one. If it does, the rule isn't strong enough — make it more concrete.

---

## Template 1 — TypeScript / Node service

```markdown
# Repo: <service-name>

## What this service does
<One paragraph. The agent uses this to ground itself.>

## Stack
- Runtime: Node.js <version>
- Language: TypeScript <version>, strict mode ON
- Framework: <Fastify | Express | Hono | NestJS>
- Database: <Postgres via Prisma | etc>
- Test runner: <Vitest | Jest>
- Package manager: <pnpm | npm | yarn>

## Conventions

### Project structure
- `src/api/` — HTTP handlers, thin
- `src/services/` — business logic
- `src/repositories/` — database access
- `src/infrastructure/` — external services, queues, caches
- `tests/` — mirrors `src/` structure

### Imports and layering
- `api` imports from `services`
- `services` imports from `repositories` and `infrastructure`
- `repositories` and `infrastructure` do not import from `services` or `api`
- Violations are caught by the fitness function in `tests/architecture.test.ts`

### TypeScript style
- `strict: true` — no implicit any, no unchecked indexed access
- No `any`. Use `unknown` and narrow.
- Prefer `type` aliases for unions, `interface` for object shapes that may be extended
- Async errors thrown, not returned in tuples
- No default exports — named exports only

### Error handling
- Use the `Result<T, E>` type from `src/lib/result.ts` for expected failures
- Throw only for unexpected failures
- Every thrown error has a `code` and is mapped to an HTTP status in the error middleware
- Never throw a string. Throw `AppError` subclasses.

### Logging
- Use the structured logger from `src/lib/log.ts`
- Log in JSON, never `console.log`
- Never log: passwords, tokens, full card numbers, PHI fields (see schema annotations)
- Include `request_id` in every log line in a request context

### Database
- All access goes through repository classes
- No raw SQL outside `repositories/` — Prisma queries only
- Migrations are reviewed by a human, even when AI-generated
- Money is stored as integer cents, never floats

### Testing
- Tests live in `tests/` mirroring `src/`
- Unit tests for services and repositories with mocked dependencies
- Integration tests against a real Postgres in CI (testcontainers)
- One assertion per test where practical
- Test names: `it("returns 404 when order not found")` — describe behavior, not implementation

### Test commands
- `pnpm test` — unit tests
- `pnpm test:integration` — integration tests
- `pnpm test:mutation` — mutation tests via Stryker (critical paths only)
- Always run `pnpm test` before claiming a task is complete

## Never do
- Never use `any`
- Never use raw SQL outside the repository layer
- Never log secrets, PII, PHI, or card data
- Never `console.log` — use the structured logger
- Never use default exports
- Never `as` cast without a runtime check immediately after
- Never write a test that asserts implementation detail (e.g., "the service called the repository") — assert behavior, not calls

## Always do
- Always run the test suite before declaring done
- Always search the codebase for callsites of any function you modified
- Always add types to function parameters and return values
- Always add JSDoc to exported functions in public modules

## Request-driven workflow
- Every task implements a Request Contract from BugLady, provided in your context (the PR links to it)
- Read the contract — including the attached user interview — before generating code
- Acceptance Criteria in the contract become test cases — generate them first, then implement
- Do not exceed scope: respect Boundaries (when present) and the Summary's intent
- If the contract is ambiguous, check the interview first; if still ambiguous, ask one clarifying question rather than guessing
- Never mutate the contract — if it's wrong, the Request gets killed and respawned by the PM

## Compliance mode
<Set to one of: none / SOC2 / HIPAA / PCI / gov. See FLEX-MODES.md.>

<If HIPAA, add:>
- Never log fields annotated `@phi` in schemas
- All PHI access goes through `src/lib/audit.ts`
- External calls with PHI go only to BAA-verified vendors

<If PCI, add:>
- Never store, log, or return raw card data
- Always use payment provider tokens
- Card-handling code lives only in `src/services/payments/` — never elsewhere
```

---

## Template 2 — Python service

```markdown
# Repo: <service-name>

## What this service does
<One paragraph.>

## Stack
- Python <3.11+>
- Framework: <FastAPI | Litestar>
- Database: <Postgres via SQLAlchemy | etc>
- Test runner: pytest
- Package manager: uv (preferred) or poetry
- Type checker: pyright (strict mode) or mypy

## Conventions

### Project structure
- `src/<package>/api/` — HTTP handlers, thin
- `src/<package>/services/` — business logic
- `src/<package>/repositories/` — database access
- `src/<package>/infrastructure/` — external services
- `tests/` — mirrors `src/`

### Imports and layering
- `api` imports from `services`
- `services` imports from `repositories` and `infrastructure`
- `repositories` and `infrastructure` do not import from `services` or `api`
- Enforced by `pytest tests/architecture/`

### Python style
- Type hints required on all public functions
- `from __future__ import annotations` at top of every module
- Pyright strict mode passes — no implicit Any
- Dataclasses or Pydantic models for data structures, never raw dicts in business logic
- f-strings for formatting, never `.format()` or `%`
- Pathlib for filesystem, never `os.path`

### Error handling
- Custom exceptions inherit from `AppError`
- Every `AppError` has a `code` mapped to HTTP status
- Never raise bare `Exception` or string-based errors
- Use `result` types from `src/<package>/lib/result.py` for expected failures

### Logging
- Use the structured logger from `src/<package>/lib/log.py`
- Log in JSON via `structlog`
- Never `print()` — use the logger
- Never log: passwords, tokens, full card numbers, PHI fields
- Include `request_id` in every log line in a request context

### Database
- All access through repository classes
- No raw SQL outside repositories — SQLAlchemy ORM or `text()` only in approved cases
- Migrations via Alembic, reviewed by a human
- Money as `Decimal` or integer cents — never float

### Testing
- Tests in `tests/` mirroring `src/`
- pytest fixtures over setup/teardown
- One behavior per test
- Test names describe behavior: `def test_returns_404_when_order_not_found():`
- Integration tests use testcontainers for Postgres

### Test commands
- `pytest` — unit tests
- `pytest tests/integration` — integration tests
- `pytest --cov` for coverage
- `mutmut run` for mutation tests on critical paths
- Always run `pytest` before claiming a task is complete

## Never do
- Never use `Any` without a `# type: ignore[explicit-any]` comment explaining why
- Never use raw SQL outside the repository layer
- Never log secrets, PII, PHI, or card data
- Never use `print()` — use the structured logger
- Never use bare `except:` — always catch specific exceptions
- Never mutate function default arguments
- Never use `os.path` — use `pathlib.Path`

## Always do
- Always run pytest before declaring done
- Always search for callsites of any function you modified
- Always add type hints to function parameters and return values
- Always add docstrings to public functions and classes

## Request-driven workflow
- Every task implements a Request Contract from BugLady, provided in your context (the PR links to it)
- Read the contract — including the attached user interview — before generating code
- Acceptance Criteria in the contract become test cases — generate them first, then implement
- Do not exceed scope: respect Boundaries (when present) and the Summary's intent
- If the contract is ambiguous, check the interview first; if still ambiguous, ask one clarifying question rather than guessing
- Never mutate the contract — if it's wrong, the Request gets killed and respawned by the PM

## Compliance mode
<Set to one of: none / SOC2 / HIPAA / PCI / gov. See FLEX-MODES.md.>

<If HIPAA, add:>
- Never log fields annotated with `phi=True` in Pydantic models
- All PHI access goes through `src/<package>/lib/audit.py`
- External calls with PHI go only to BAA-verified vendors

<If PCI, add:>
- Never store, log, or return raw card data
- Always use payment provider tokens
- Card-handling code lives only in `src/<package>/services/payments/`
```

---

## Template 3 — Go service

```markdown
# Repo: <service-name>

## What this service does
<One paragraph.>

## Stack
- Go <1.22+>
- Framework: <stdlib net/http with chi router | Echo | Gin>
- Database: <Postgres via pgx | sqlc-generated queries>
- Test runner: standard `go test` + testify
- Module: `<module path>`

## Conventions

### Project structure
- `cmd/<service>/` — main entry point
- `internal/api/` — HTTP handlers, thin
- `internal/service/` — business logic
- `internal/repository/` — database access
- `internal/infrastructure/` — external services
- `pkg/` — only if intended for external import (rare)

### Imports and layering
- `api` imports from `service`
- `service` imports from `repository` and `infrastructure`
- `repository` and `infrastructure` do not import from `service` or `api`
- Enforced by `internal/archtest/`

### Go style
- gofmt, goimports, golangci-lint clean
- Error wrapping: `fmt.Errorf("context: %w", err)`, always wrap when returning
- Errors are values — `errors.Is` and `errors.As` for comparison
- Context as first parameter on functions that might block
- Receiver name is a short, consistent abbreviation across methods on a type
- Interfaces defined where they're used, not where they're implemented
- Accept interfaces, return concrete types
- No naked returns in functions longer than ~5 lines

### Error handling
- Sentinel errors for known cases: `var ErrNotFound = errors.New("not found")`
- Custom error types implementing `error` for cases that carry context
- Map errors to HTTP status in a single middleware
- Never use panic for control flow — only for genuine programmer error

### Logging
- Use `log/slog` with the structured handler
- Log in JSON in production
- Never `fmt.Println` — use slog
- Never log: passwords, tokens, full card numbers, PHI fields
- Include `request_id` via context

### Database
- All access through repository interfaces
- Use sqlc-generated queries — raw SQL only via sqlc
- Migrations via golang-migrate, reviewed by a human
- Money as int64 cents, never float

### Concurrency
- Always pass context.Context for cancellation
- Goroutines must have a clear lifecycle — no leaks
- Channels for ownership transfer; mutexes for shared state — pick one consistently per type
- Use `sync.WaitGroup` or `errgroup` for fan-out

### Testing
- Tests live next to code: `foo.go` → `foo_test.go`
- Table-driven tests for multiple cases
- Use testify/require for fatal assertions, testify/assert for soft
- Integration tests in `_integration_test.go` files, gated by build tag
- Test names describe behavior: `TestOrder_ReturnsErrorWhenNotFound`

### Test commands
- `go test ./...` — all tests
- `go test -tags=integration ./...` — integration
- `gremlins run` for mutation tests on critical paths
- Always run `go test ./...` before claiming a task is complete

## Never do
- Never use `interface{}` (or `any`) without a strong reason
- Never use raw SQL outside the repository layer
- Never log secrets, PII, PHI, or card data
- Never use `fmt.Println` — use the structured logger
- Never ignore an error — handle it or return it wrapped
- Never panic in library code
- Never start a goroutine without a clear stop mechanism

## Always do
- Always run `go test ./...` before declaring done
- Always run `go vet ./...` and `golangci-lint run`
- Always search for callsites of any function you modified
- Always wrap errors with context: `fmt.Errorf("doing X: %w", err)`
- Always pass context as the first parameter on blocking calls

## Request-driven workflow
- Every task implements a Request Contract from BugLady, provided in your context (the PR links to it)
- Read the contract — including the attached user interview — before generating code
- Acceptance Criteria in the contract become test cases — generate them first, then implement
- Do not exceed scope: respect Boundaries (when present) and the Summary's intent
- If the contract is ambiguous, check the interview first; if still ambiguous, ask one clarifying question rather than guessing
- Never mutate the contract — if it's wrong, the Request gets killed and respawned by the PM

## Compliance mode
<Set to one of: none / SOC2 / HIPAA / PCI / gov. See FLEX-MODES.md.>

<If HIPAA, add:>
- Never log fields tagged `phi:"true"` in struct tags
- All PHI access goes through `internal/audit/`
- External calls with PHI go only to BAA-verified vendors

<If PCI, add:>
- Never store, log, or return raw card data
- Always use payment provider tokens
- Card-handling code lives only in `internal/service/payments/`
```

---

## Common section: agent loop expectations

These apply regardless of language. Add this to every `CLAUDE.md`:

```markdown
## Agent loop expectations

When you take on a task:

1. **Read the Request Contract first.** It comes from BugLady and is provided in your context (the PR links to it). Read the attached interview too. If there's no contract, STOP and ask.
2. **Read this `CLAUDE.md` and any nested ones** in the directory you're working in.
3. **Search for related code** before generating new code — if it exists, reuse or refactor; do not duplicate.
4. **Generate tests first** from the Acceptance Criteria in the contract.
5. **Implement** the minimum that satisfies the tests and the contract.
6. **Run the test suite** before declaring complete.
7. **Search for callsites** of every symbol you modified. If a callsite was not opened, the task is not done.
8. **Summarize the diff** in the PR description, including: what changed, why, what tests cover it, and the BugLady link to the Request Contract. If the contract proved wrong, say so — the PM decides kill-and-respawn; you never mutate the contract.

If at any step you find contract ambiguity, check the interview first; if still ambiguous, ask one clarifying question rather than guessing. Guesses produce slop.
```

---

## How to audit your `CLAUDE.md`

Once a quarter, audit by:

1. Pick three rules from `CLAUDE.md` at random
2. Ask the agent to write code that violates each one
3. If the agent does, the rule is too weak — rewrite to be more concrete
4. If the agent refuses or warns, the rule is working

Rules age. Codebases drift. Audit.

---

## Common mistake: rules vs. preferences

`CLAUDE.md` is for **rules**, not preferences. Rules are things that produce real harm if violated. Preferences are things you'd like but won't fight for.

- "Never log PHI" → rule (regulatory harm)
- "Prefer functional composition over inheritance" → preference

Mixing preferences into the rules section dilutes the rules. Keep preferences in a separate section if you must include them, or leave them to code review.
