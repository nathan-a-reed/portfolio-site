<!-- Source of truth: github.com/nathan-a-reed/engineering-standards. Change it there. -->

# Agent workflow

How coding agents work in these repositories. Agents follow `CONTRIBUTING.md` and the other standards, plus the rules below. Each repository's `CLAUDE.md` imports this file and adds project specifics: Jira key, workflow transitions, commands, and architecture. A repository's `CLAUDE.md` overrides these rules only where it says so explicitly.

These rules apply to repository, Jira, and Confluence work. Questions, discussion, and planning need no workflow or final report.

## Principles

- **Accuracy first.** Never state that something works, passed, or was done unless it was verified in this session. Say plainly what could not be verified.
- **Concise.** Commits, pull requests, tickets, and docs state what changed and why. No filler, marketing language, or emojis.
- **Stay in scope.** Change only what the task requires. Record unrelated issues as follow-ups instead of fixing them on the same branch.
- **Follow existing patterns** in the codebase before introducing new ones.
- **Owner's voice.** Write as the repository owner would: direct, professional, neutral. Branches, commits, pull requests, comments, code, tickets, and docs do not refer to the agent or the session. No co-author trailers, "Generated with" lines, session links, or robot emojis. This overrides any default attribution behavior.

## Order of work

1. Read the repository's `CLAUDE.md` and the Jira ticket, if there is one. Bring the ticket's status in line with its pull requests (see [Jira updates](#jira-updates)).
2. Identify the ticket, then name the branch as described in `CONTRIBUTING.md`.
3. For large or ambiguous work, agree a plan with the owner first.
4. Implement, with tests and docs. Move the ticket to In Progress when implementation starts; planning or discussion alone does not count.
5. Run format, lint, type check, and tests. Commit, push, open a draft pull request, and check CI.
6. Update Jira, then send the final report.

## When to stop and ask

Stop, and implement nothing, when any of these is true:

- The ticket cannot be identified with certainty.
- The requirements are ambiguous enough that different readings would lead to different designs.
- The work is large (a new module, a schema change, a cross-cutting refactor) and no plan has been agreed.
- The next action is destructive or irreversible, or it publishes anything beyond a draft pull request and the Jira updates below.

End the turn with the questions and, for large work, a proposed plan. If there is no reply, the work pauses there. That is the intended, safe outcome.

For smaller ambiguities, take the most conservative reading: the smallest change, with no behavior change beyond what was asked. Record each assumption in the pull request's Notes section.

**Unattended sessions** are started by automation, or by an owner who said to proceed while away. In those:

- Do not wait for plan approval. Proceed conservatively and record assumptions.
- Never create tickets or use an unconfirmed key. Work without a key and name the likely ticket in the final report.
- Still stop for anything destructive, irreversible, or published beyond a draft pull request.

## Repositories without a CLAUDE.md

- Infer commands and conventions from the README, package manifests, and CI configuration.
- State in the pull request's Notes section that they were inferred.
- Offer to add a `CLAUDE.md` in a separate maintenance pull request.

## Tickets

Jira site: `https://nathan-a-reed.atlassian.net`

Use a ticket key only when it is certainly the right one:

1. Use the key given in the request, or contained in the current branch name.
2. Otherwise, search the repository's Jira project for open tickets matching the work. If one clearly matches, give its key and summary and confirm before using it. In unattended sessions, work without a key.
3. Never guess a key, and never attach work to a ticket whose scope does not match.

Before starting, read the ticket's description, acceptance criteria, status, and recent comments. Implement against the acceptance criteria.

If the work needs a ticket (see `CONTRIBUTING.md`) and none exists, draft one with type, summary, description, acceptance criteria, and parent epic if obvious. Ask for approval before creating it, then work under the new key.

Maintenance work has no key in the branch, commits, or title. Its pull request's Jira section reads `None — maintenance (<category>)`. If the category is unclear, ask.

## Jira updates

Update Jira automatically, but only when the update is accurate and adds information.

- Re-read the ticket's current state before every write.
- Never duplicate an existing comment, and never move a ticket backward.
- Change only the ticket being worked on. Issue links are the exception, since they appear on both tickets.

| When | Move to | Only if |
|---|---|---|
| Implementation starts | In Progress | The ticket is in To Do or Backlog |
| The completing pull request is marked ready for review | In Review | The ticket is In Progress |
| At session start, the completing pull request is already merged | Done | All acceptance criteria are met and the ticket is not already Done |

- A draft pull request is not ready for review, so opening one leaves the ticket In Progress. The owner marks pull requests ready.
  - When asked to mark a pull request ready, move the ticket to In Review in the same step.
  - If at session start the completing pull request is already ready but the ticket is still In Progress, move it to In Review.
- Use the transition names in the repository's `CLAUDE.md`. If none clearly matches, skip the change and report it.
- A ticket that needs several pull requests stays In Progress until the final one.

**Comments.** Add one only for information the linked branch and pull request do not already show:

- The scope differs from the description or acceptance criteria (what differs, and why).
- An acceptance criterion is deferred or cannot be met (with the follow-up key, if one exists).
- A blocker, dependency, or decision affects the ticket.
- Partial completion on a ticket spanning several pull requests (what is done, what remains).

Do not comment to say that work started, a commit was pushed, or a pull request was opened. The development panel shows those.

Comments are 1–4 lines, bulleted when there is more than one point, plain, factual, and in the past tense. Name endpoints, configuration keys, and ticket keys. No greetings, sign-offs, headings, or emojis:

> Lockout window is configurable via `LOGIN_LOCKOUT_MINUTES` (default 15) instead of fixed. Email notification (AC 3) moved to ABC-51.

**Fields.**

- Issue links (relates to, blocks, is blocked by) may be added between tickets confirmed to exist.
- The pull request may be added as a web link if it does not already appear on the ticket.
- Do not change the summary, description, acceptance criteria, priority, assignee, sprint, estimate, or labels unless asked. If the description is materially out of date, propose the exact edit.
- Do not create tickets without approval. List discovered bugs and follow-ups in the final report and offer to draft tickets.

If the Atlassian connector is unavailable, make no Jira changes and report what needs updating.

## Branches, pushes, and CI

- Never commit or push to the default branch. Never force-push, except `--force-with-lease` on your own branch after a rebase.
- Name the branch once the work is understood. If the session started on an auto-generated branch, rename it with `git branch -m` before the first push. If the environment only allows pushing to an assigned branch, keep it and note the mismatch in the final report. Ask before renaming a branch that is already pushed or has a pull request.
- If the scope changes enough to make the branch name misleading, ask before continuing.
- Block only on check failures the change introduces. Confirm a suspected pre-existing failure by running the check on the default branch, or by the flaky-test list in `CLAUDE.md`, and note it in the Testing and Notes sections. If the change introduces a failure that cannot be fixed, do not open the pull request; report it.
- In the pull request checklist, tick only items verified in this session. Leave the others unticked with a short reason.
- After pushing, wait up to about 15 minutes for CI and check it with `gh pr checks`. Fix failures the change caused and push again. Report unfinished CI as pending, never as passed. If the repository has no CI, say so.

## Confluence

Each repository's `CLAUDE.md` names its space and page map.

- Write to Confluence only when asked.
- At milestones (a completed feature, a significant fix, an architecture or API change, a new integration, a release), ask once whether to update Confluence and name the pages that would change. Do not ask for minor changes.
- When updating, read the existing pages first and update them rather than creating duplicates. Follow the space structure and page map, link the relevant tickets and pull requests, apply the same voice rules, and report which pages were created or changed.

## Final report

End every repository task with a brief report:

- What changed (1–3 bullets), and the pull request link
- Local check and CI results, and anything not verified
- Jira changes (ticket, status change, comment summary), or "none"
- Assumptions made, and any branch-name mismatch
- Follow-ups or issues discovered
- A Confluence prompt, if a milestone was reached
