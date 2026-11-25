<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# AWS and CDK standards

## Accounts

All accounts belong to one AWS Organization. Workloads run in `us-east-1`.

| Account | Contains | Changed by |
|---|---|---|
| Management | Organization, IAM Identity Center, billing, budgets, organization trail, domain registration | Manual deploys from a workstation; rare |
| Test | Test stage of every application, `test.nathanreed.dev` zone | CI/CD on merge to `main` |
| Prod | Prod stage of every application, `nathanreed.dev` zone | CI/CD after approval |

- Test and prod sit in a `Workloads` organizational unit with service control policies that deny leaving the organization, deny actions outside `us-east-1` other than global services, deny root user actions, and protect the organization trail.
- People sign in through IAM Identity Center. There are no IAM users or long-lived access keys. The root user has MFA and is not used day to day.
- The management account runs no workloads, and CI/CD has no access to it.

## Repository responsibilities

`aws-foundation` owns what every application shares. Each application repository owns everything specific to it, including its users and authentication.

| Concern | Owner |
|---|---|
| Organization, OUs, SCPs, Identity Center | `aws-foundation` |
| Budgets and cost anomaly detection | `aws-foundation` |
| GitHub OIDC provider and per-repository deployment roles | `aws-foundation` |
| CodeConnections connection to GitHub (one-time console authorization) | `aws-foundation` |
| CDK bootstrap of test and prod | `aws-foundation` (documented procedure) |
| Hosted zones and subdomain delegation | `aws-foundation` |
| Application resources, Cognito user pools, API authorization, data stores | Application repository |

## Foundation contract

Applications read foundation resources only through SSM parameters under `/foundation/`. They never import CloudFormation exports from another repository or reference foundation resources by hardcoded name or ID.

| Parameter | Value |
|---|---|
| `/foundation/dns/zone-name` | The account's hosted zone name |
| `/foundation/dns/zone-id` | The account's hosted zone ID |

- Read parameters at deploy time with `ssm.StringParameter.valueForStringParameter`.
- Changing or removing a parameter is a breaking change: add the replacement, migrate consumers, then remove the original.

## DNS

The domain is registered in the management account, and its name servers point to the `nathanreed.dev` zone in prod. The prod zone delegates `test.nathanreed.dev` to a zone in test.

| Application | Prod | Test |
|---|---|---|
| `portfolio-site` | `nathanreed.dev` (`www` redirects to apex) | `test.nathanreed.dev` |
| `outage-atlas` | To be decided | To be decided |
| `nasa-rag` | To be decided | To be decided |

Mailbox records (MX, SPF, DKIM, DMARC) for the domain live in the prod zone and are managed by `aws-foundation`. Application email is sent through Amazon SES in prod with Easy DKIM and a custom MAIL FROM subdomain, so the mailbox provider's records stay unchanged.

Test environments return `X-Robots-Tag: noindex`.

## Deployment access

- GitHub Actions authenticate through OIDC. Each repository has one deployment role per account. Its trust policy is limited to that repository and GitHub environment (`repo:<owner>/<repo>:environment:<test|prod>`).
- Deployment roles may only assume the CDK bootstrap roles. CloudFormation performs the changes.
- CodePipeline pipelines reach GitHub through the foundation's CodeConnections connection.
- Account IDs and role ARNs are GitHub environment variables, never source.

## CDK

TypeScript in strict mode. Lambda functions are Python (see `python.md`).

```
bin/app.ts        Entry point; creates a stage per environment
lib/config.ts     Typed per-environment configuration
lib/stages/       One Stage per deployable unit
lib/stacks/       Stacks split by lifecycle (stateful, stateless)
lib/constructs/   Reusable L3 constructs
functions/        Python Lambda project
test/             Infrastructure assertion tests
```

- Use L2 constructs. Use L1 only for properties L2 does not expose, wrapped in a construct.
- Keep stateful resources (tables, buckets, user pools) in separate stacks from stateless ones so stateless stacks can be replaced freely.
- Do not hardcode physical names for buckets, tables, functions, or roles. Pass references. SSM parameter names in the foundation contract are the exception.
- Do not use `fromLookup` methods; they write account details into `cdk.context.json`. Read foundation values from SSM.
- Environment differences come from typed configuration (`config.retainData`), not stage-name checks spread through constructs.
- Stateful resources are retained in prod and destroyed in test.
- Construct IDs are PascalCase and describe the role: `SiteBucket`, `DocumentsTable`.
- Stack names follow `<Project>-<Stage>-<Purpose>`, for example `PortfolioSite-Prod-Web`.
- The cdk-nag `AwsSolutionsChecks` aspect runs on every stack. Suppressions name the specific resource and give a reason.
- The app applies `Project`, `Environment`, and `Repository` tags, activated as cost allocation tags.
- Avoid alpha modules (`@aws-cdk/*-alpha`) unless an ADR accepts the risk.

## Service defaults

| Area | Default |
|---|---|
| Compute | Lambda on `arm64`, memory sized by measurement |
| Logs | Explicit retention: 30 days in test, 90 days in prod |
| Object storage | Block Public Access, encryption, TLS-only bucket policy, versioning for data buckets |
| Tables | DynamoDB on-demand, point-in-time recovery in prod |
| Static hosting | S3 behind CloudFront with origin access control, TLS 1.2 minimum, HTTP/2 and HTTP/3 |
| APIs | AppSync or API Gateway with Cognito or IAM authorization |
| Networking | No VPC unless a service requires one; no NAT gateways |
| Alarms | Errors and throttles for each function and API, sent to an SNS email topic |

## Cost

| Budget | Monthly limit |
|---|---|
| Organization total | $60 |
| Prod | $40 |
| Test | $15 |
| Management | $5 |

- Budgets alert at 50%, 80%, and 100% of actual spend and at 100% of forecast spend. Cost anomaly detection covers all accounts.
- Resources that bill while idle (provisioned capacity, NAT gateways, OpenSearch Serverless, always-on containers) need an ADR.
- Usage that grows with traffic, such as Bedrock tokens and uploads, is capped by server-side quotas per user and per account.
