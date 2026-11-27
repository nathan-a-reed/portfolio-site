<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Contributing

## Workflow

Development is trunk-based. `main` is always deployable and protected: changes land only through pull requests that pass the required checks. Branches are short-lived, cut from the latest `main`, scoped to one ticket, and deleted after merge.

Environments are deployment stages, not branches. A merge to `main` deploys to test; promotion to prod requires approval. Each repository's README describes its pipeline.

## Tickets

Work is tracked in Jira. Each repository's `CLAUDE.md` names its project key.

**Needs a ticket:** anything that changes behavior or would appear in release notes.

- Features and bug fixes
- Refactors beyond trivial ones
- Schema or data migrations
- Security fixes
- Infrastructure or deployment changes
- New dependencies

**Maintenance, no ticket:**

- Typo, comment, or formatting fixes
- Small documentation corrections
- Patch or minor dependency updates that need no code changes
- CI, lint, or tooling configuration tweaks
- Reverts

Ticketed and maintenance changes never share a pull request.

## Branches

| Work | Format | Example |
|---|---|---|
| Ticketed | `<type>/<KEY>-<number>-<description>` | `feat/ABC-12-project-cards` |
| Maintenance | `<type>/<description>` | `chore/bump-eslint-9-12` |

- `<type>` is a commit type from the table below.
- `<description>` is 2–6 lowercase kebab-case words describing what the branch changes. No generic words (update, changes, fixes, misc, wip), dates, or random suffixes.
- One branch per ticket.

## Commits

Commits follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <summary>

<body: why, only when the header cannot say it>

Refs: <KEY>-<number>
```

| Type | Use for |
|---|---|
| `feat` | New capability |
| `fix` | Bug fix |
| `refactor` | Code change that preserves behavior |
| `perf` | Performance improvement |
| `test` | Tests only |
| `docs` | Documentation only |
| `style` | Formatting only |
| `build` | Build tooling, packaging, dependencies |
| `ci` | CI/CD configuration |
| `chore` | Maintenance that fits no other type |
| `revert` | Reverting an earlier change |

- **Header:** imperative mood, lowercase, no trailing period, at most 72 characters. Scopes are listed in each repository's `CLAUDE.md`.
- **Body:** wrapped at 72 characters. Included only when it adds a why: the reason for the change, or consequences that are not obvious.
- **Footer:** `Refs: <KEY>-<number>` for ticketed work, and `BREAKING CHANGE: <what>` when applicable.
- Each commit is one logical change and leaves the build passing.

Commit messages and pull requests describe the change in a neutral voice. They do not include tool attribution, co-author trailers for tools, or session links.

## Pull requests

- Open pull requests as drafts and mark them ready for review when complete. Coding agents push branches but do not open pull requests; see the agent workflow standard, `agent-workflow.md`.
- **Title:** Conventional Commits format with the ticket key at the end, at most 72 characters: `feat(web): add project cards [ABC-12]`. Maintenance titles have no key. The title becomes the squash commit on `main`.
- **Description:** free-form, with no required format.
- **Size:** aim for under about 400 changed lines, excluding lockfiles and generated files. Explain anything larger or split it.
- Merge only when required checks pass and conversations are resolved.

## Required checks

Every repository requires at least:

- **PR hygiene:** branch name, title format, ticket key consistency, no tool attribution
- **Quality:** format, lint, and type checks
- **Tests:** unit and infrastructure assertion tests
- **Synth:** `cdk synth` with cdk-nag, in repositories with infrastructure

## Definition of done

- Acceptance criteria are met, or deviations are documented in the pull request and the ticket.
- Format, lint, type checks, and tests pass.
- Tests are added or updated.
- Docs and ADRs are updated where needed.
- Branch, commits, and title follow this guide.
- The CI result is known.

## Local development

Each README lists prerequisites and commands. Toolchain versions are pinned (`.nvmrc`, `.python-version`) and lockfiles are committed.
