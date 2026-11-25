<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Python standards

Python is used for AWS Lambda functions. Infrastructure is TypeScript (see `aws.md`).

## Toolchain

| Tool | Purpose |
|---|---|
| uv | Python version, dependencies, lockfile, virtual environment |
| Ruff | Linting and formatting |
| mypy (strict) | Type checking |
| pytest | Tests |

- Target the newest Python version AWS Lambda supports. Pin it in `.python-version` and `requires-python`.
- All tool configuration lives in `pyproject.toml`.
- Every function signature has type hints. Use of `Any` needs a reason.
- Modules and files use `snake_case`.

## Approved dependencies

| Package | Use |
|---|---|
| `boto3` | AWS SDK. Bundle a pinned version when an API is newer than the runtime's copy, which is common for Bedrock. |
| `aws-lambda-powertools` | Logging, tracing, metrics, event parsing, parameters, idempotency |
| `pydantic` | Event and response models, through the Powertools parser |
| `boto3-stubs` (dev) | Typed AWS clients |
| `pytest`, `ruff`, `mypy` (dev) | Testing and quality |

Anything else follows the dependency policy in `engineering.md`.

## Layout

Each repository with Lambda functions has a single Python project.

```
functions/
  pyproject.toml
  uv.lock
  src/
    query_documents/
      handler.py      Entry point: parse the event, call the service, shape the response
      service.py      Business logic; no event types, no client construction
    shared/           Code used by more than one function
  tests/
    fixtures/events/  Sample Lambda events
```

## Functions

- Keep handlers thin: validate the event, call the service, map results and errors to the response.
- Create AWS clients once at module scope, set timeouts and retry mode explicitly with `botocore.config.Config`, and pass clients into services.
- Read configuration from environment variables, validated at cold start. Read secrets through the Powertools parameters utility with caching.
- Log through Powertools `Logger` with structured keys, not formatted strings. Use `Tracer` and `Metrics` for traces and business metrics.
- Services raise domain exceptions. The handler translates them into responses.

## Testing

- Test services with fake adapters. Prefer simple fakes over `unittest.mock.patch`.
- Test adapters with `botocore.stub.Stubber` where the request shape matters.
- Test handlers with event fixtures from `tests/fixtures/events/`.
- No test calls real AWS.

## Packaging

Each function's bundle contains its own code, `shared/`, and the locked runtime dependencies built for `arm64`. Development dependencies are never bundled.
