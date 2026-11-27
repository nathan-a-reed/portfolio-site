<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Engineering standards

These apply to every repository. Language and platform rules are in `aws.md`, `python.md`, and `frontend.md`.

## Principles

- **Prefer simple and proven.** Reach for a managed service, the standard library, or an existing module before building something new.
- **Design for reuse; extract on second use.** Give modules clean boundaries from the start. Move them to a shared package when a second consumer exists, not before.
- **Secure and observable by default.** Least privilege, encryption, structured logs, and alarms are part of the change, not follow-up work.
- **Pay for use, not for idle.** Default to services that scale to zero.

## Structure and reuse

- Before writing new code, look for an existing module, construct, or component that does the job or can be extended.
- Extend with an option or variant instead of creating a near-duplicate. Create something new only when the responsibility is different.
- Keep business logic independent of delivery details. Handlers, UI frameworks, and AWS clients are adapters at the edges.
- Reusable code knows nothing about a specific application: no hardcoded names, domains, or environment values.
- Duplicating once is acceptable while an abstraction is unclear. A third copy is not.

## Naming

- Names describe intent, not type or mechanism: `documentStore`, not `s3Helper`.
- Avoid abbreviations other than widely understood ones (`id`, `url`, `api`).
- Follow language conventions. File naming rules are in the language standards.

## Comments and documentation

Names and structure explain what code does. Comments explain what code cannot: why a decision was made, a constraint that is not obvious, or a reference to an external specification.

- Document exported APIs (TSDoc, docstrings) when the contract is not obvious from the signature.
- Do not restate code, narrate steps, or add section banners.
- Do not commit commented-out code.
- `TODO` comments reference a ticket: `TODO(ABC-51): remove after the v2 schema migration`.

Restates the code:

```ts
// Set the removal policy
removalPolicy: config.retainData ? RemovalPolicy.RETAIN : RemovalPolicy.DESTROY,
```

Explains the decision:

```ts
// Uploaded documents cannot be regenerated from source, so prod keeps them if the stack is deleted.
removalPolicy: config.retainData ? RemovalPolicy.RETAIN : RemovalPolicy.DESTROY,
```

Every README covers: purpose and live link, architecture diagram, key decisions (linking ADRs), local development, testing, deployment, and expected cost.

## Errors and logging

- Validate input at boundaries with schemas rather than scattered checks.
- Handle an error where something useful can be done; otherwise let it propagate with context. Never swallow errors.
- User-facing errors are actionable and reveal nothing internal such as stack traces, ARNs, or queries.
- Logs are structured JSON with a correlation ID. Never log secrets, tokens, personal data, or document contents.
- `ERROR` needs action, `WARN` is unexpected but handled, `INFO` records significant events, and `DEBUG` is off in prod.

## Configuration

- Environment-specific values live in typed configuration or SSM parameters, not literals spread through the code.
- No secrets, AWS account IDs, or personal data in source. Use `111122223333` where an example account ID is needed.

## Dependencies

Every dependency is code that must be secured, updated, and shipped. Add one only when it offers substantial value over the platform or a small amount of our own code.

- Check the approved lists in `python.md` and `frontend.md` first.
- A runtime dependency outside those lists requires an ADR covering the problem, alternatives (including writing it ourselves), size, maintenance activity, license, and security history.
- Development tools meet the same bar; justify them in the commit body.
- Prefer packages with few transitive dependencies and permissive licenses (MIT, Apache-2.0, BSD, ISC).
- Commit lockfiles. Update versions through Dependabot pull requests.

## Testing

Tests catch regressions and document behavior. A test that cannot fail for a meaningful reason is deleted.

Test:

- Business logic and data transformations
- Authorization and input validation
- Error paths that users or operators rely on
- Security-critical infrastructure properties: encryption, public access, IAM scope
- Every bug fix, with a regression test that fails without the fix

Do not test:

- Framework or library behavior, including CDK defaults
- Constants, trivial accessors, or pass-through code
- Implementation details that would block refactoring without protecting behavior

Practices:

- Name tests for the behavior they verify: `test_returns_not_found_for_unknown_document`, `it('closes the menu after a selection')`.
- One behavior per test, structured as arrange, act, assert.
- Tests are deterministic and isolated: no network, no real AWS, no shared mutable state, no sleeps.
- Replace dependencies at the adapter boundary with fakes instead of patching internals.
- No whole-template or whole-component snapshots. They change with every edit and get approved without review.
- Never delete or weaken a test to make it pass.
- Coverage is reported, not targeted.

| Layer | Scope | Runs |
|---|---|---|
| Unit | Functions, components, constructs | Every pull request |
| Infrastructure assertions | Synthesized CloudFormation | Every pull request |
| Smoke | Deployed endpoints and critical user paths | After each test deploy, before prod approval |

## Architecture decision records

Significant decisions are recorded in `docs/adr/NNNN-short-title.md` with context, decision, alternatives, and consequences. Accepted ADRs are not rewritten; a new ADR supersedes an old one.

Write one for a new runtime dependency, a new AWS service, a data model, an authentication approach, or a pipeline design.
