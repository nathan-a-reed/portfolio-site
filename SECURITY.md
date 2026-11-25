<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Security policy

## Reporting a vulnerability

Report vulnerabilities privately through the affected repository: **Security → Report a vulnerability**. Please do not open a public issue.

Reports are acknowledged within five business days.

## Scope

- Code in this account's repositories
- Applications deployed at `nathanreed.dev` and its subdomains

## Credentials

These repositories contain no credentials. Deployments authenticate to AWS through OpenID Connect or AWS CodeConnections, and runtime secrets are stored in AWS Secrets Manager or AWS Systems Manager Parameter Store.
