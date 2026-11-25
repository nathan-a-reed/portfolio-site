# CLAUDE.md: Portfolio site / portfolio-site

## Project

Source for `nathanreed.dev`: background, experience, and projects, with links to each project's live demo and repository. Hiring managers and engineers are the audience.

| | |
|---|---|
| Jira project | Product work: space to be created. Repository setup: `FND` (https://nathan-a-reed.atlassian.net/browse/FND) |
| Confluence space | `ENG` (Engineering): https://nathan-a-reed.atlassian.net/wiki/spaces/ENG |
| Default branch | `main` |
| Related repos | `aws-foundation` (accounts, deployment roles, DNS), `engineering-standards` (source of `docs/standards/`) |

## Standards

@CONTRIBUTING.md
@docs/standards/agent-workflow.md
@docs/standards/engineering.md
@docs/standards/security.md
@docs/standards/aws.md
@docs/standards/frontend.md

## Jira workflow

| Status | Transition to reach it |
|---|---|
| To Do | To Do |
| In Progress | In Progress |
| In Review | In Review |
| Done | Done |

Issue types: Epic, Story, Task, Bug, Feature, Subtask

## Tech stack

Planned; confirmed when the site is scaffolded:

- Site: Astro with TypeScript, static output
- Infrastructure: AWS CDK in TypeScript, strict mode; S3 behind CloudFront
- CI/CD: GitHub Actions with OIDC

## Commands

None yet. Commands are added with the site scaffold.

## Architecture

The layout is created with the scaffold.

Rules:

- Shared AWS resources come from `aws-foundation` through the SSM contract in `docs/standards/aws.md`. Do not create or change them here.
- The prod site serves `nathanreed.dev`, with `www` redirecting to the apex. Test serves `test.nathanreed.dev` and returns `X-Robots-Tag: noindex`.
- The design system lives in `src/design-system/` and never imports application code.
- Projects and experience are content collections; adding a project means adding a content file.

## Conventions

- Commit scopes: `web`, `content`, `infra`, `ci`, `docs`
- Avoid: client-side JavaScript outside interactive islands, `fromLookup` methods, hardcoded physical names

## Testing

- Requirements: as defined in `docs/standards/frontend.md`; Lighthouse 95 or higher in every category on mobile
- External services: none; smoke tests run against the test environment
- Known flaky tests: None.

## Configuration and secrets

None yet. Variables will be documented in `.env.example`. Never commit `.env`.

## Pull requests

- Required CI checks: `PR hygiene`; build and test checks are added with the scaffold
- Merge strategy: squash merge
- Extra checklist items: screenshots for visible changes

## Confluence page map

When asked for a Confluence update, these are the pages to change:

| Change | Page |
|---|---|
| Pages or project entries | Portfolio site content plan: https://nathan-a-reed.atlassian.net/wiki/spaces/ENG/pages/1835045 |
| Release | Release notes: https://nathan-a-reed.atlassian.net/wiki/spaces/ENG/pages/1376290 |

## Restrictions and known issues

- Do not edit `docs/standards/` here. Change `engineering-standards`, then sync.
- Written content, images, and the résumé are not MIT licensed; see the README.

## Overrides of standards

None.
