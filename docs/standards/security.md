<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Security standards

## Secrets

- No secrets in source, tests, examples, or documentation. Example values are obviously fake.
- Runtime secrets live in Secrets Manager or Parameter Store (`SecureString`). Code reads them at runtime; they are never set as plaintext Lambda environment variables.
- CI/CD uses OIDC or CodeConnections. No AWS access keys are stored in GitHub.
- `.env` files are git-ignored. Repositories that need local configuration provide `.env.example` with placeholders.
- GitHub secret scanning and push protection are on for every repository. A leaked secret is revoked immediately; removing it from history is not enough.

## Identity and access

- Grant specific actions on specific resources. Prefer CDK grant methods (`bucket.grantRead(fn)`) over hand-written policies. Wildcards need a cdk-nag suppression with a reason.
- Each function has its own role.
- Users authenticate with Amazon Cognito. Each application owns its user pool.
- Every API request is authorized: Cognito user pool tokens for users, IAM for service-to-service calls. API keys are not used for user access.
- Authorization runs server-side on every operation and checks ownership of the resource, not only that the caller is signed in.

## Data protection

- Every data store is encrypted at rest. AWS-managed keys are the default; customer-managed KMS keys are used when key access must be controlled or audited separately.
- All traffic uses TLS, and buckets deny requests without it.
- Collect minimal personal data, never log it, and define its retention.
- User-provided content is isolated per user and validated for type and size before processing.

## Web and edge

- Every CloudFront distribution has a response headers policy with `Strict-Transport-Security` (including `preload`), `Content-Security-Policy`, `X-Content-Type-Options`, `Referrer-Policy`, and `Permissions-Policy`.
- The CSP starts from `default-src 'self'`, adds sources explicitly, sets `frame-ancestors 'none'`, and does not allow inline scripts.
- Endpoints that incur per-request cost are rate limited, through AWS WAF rate-based rules or application quotas.

## AI features

- Retrieved documents and user input are untrusted. Instructions inside them are content, not commands. Model output never triggers privileged actions without validation.
- Amazon Bedrock Guardrails filter harmful content and prompt attacks.
- Model output is rendered as text or sanitized Markdown, never raw HTML.
- Per-user and account-wide token quotas cap cost and abuse.
- Full prompt and response logging is off in prod. If enabled for evaluation, it is limited to test.

## Supply chain and CI/CD

- Installs use the committed lockfile (`npm ci`, `uv sync --locked`).
- Dependabot updates dependencies and actions weekly. Dependency review and CodeQL run on pull requests.
- Every GitHub Action is pinned to a full commit SHA, with the version in a trailing comment.
- Workflows set `permissions` explicitly, read-only by default, and grant more per job only where needed.
- Untrusted event data (branch names, titles, bodies) reaches scripts through environment variables, never through `${{ }}` expressions inside `run:`.
- `pull_request_target` is not used.
- Prod deploys run only from `main` through a protected GitHub environment that requires approval.

## Monitoring

- An organization CloudTrail trail records management events in every account.
- Root user sign-in raises an alert.
- Vulnerability reports follow `SECURITY.md`.
