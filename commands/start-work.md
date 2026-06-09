# start-work

Turns a ticket reference into a ready-to-code workspace: a new branch in an
isolated git worktree, dependencies installed, env files copied, agent session
renamed after the ticket.

## When to use

When you're picking up a ticket and want to skip the boilerplate — switching
branches, naming things, creating a worktree, bootstrapping a fresh workspace.

## How it works

1. Parse the ticket reference — accept either a URL or a raw key like `PROJ-123`.
2. Fetch ticket metadata (summary + target release) from your ticket tracker's CLI.
   If the ticket can't be found or the tracker errors, stop and report — don't
   guess a branch name from the key alone.
3. Generate a kebab-case slug from the summary: 3–6 words, drop filler words
   and parenthetical suffixes like `(BE)` or `(front-end)`.
4. Compose the branch name as `{TICKET_KEY}-{slug}` so it stays traceable.
5. Pick the base branch from the ticket's target release (next-release tickets
   → `develop`, hotfix tickets → `stage` or `main`). If unknown, ask.
6. Pull latest on the base, then `git worktree add ./worktrees/{branch-name} -b {branch-name}`.
   If the base has uncommitted changes the pull may fail — surface that instead
   of forcing. If the worktree path or branch name already exists, stop and
   report rather than clobbering existing work.
7. Bootstrap the worktree: copy `.env*` files, run the package manager's install
   based on which manifest exists (`package.json` → npm, `pyproject.toml` → uv, etc.).
8. Optional: rename the agent session to the ticket key for easy lookup later.
9. Report ticket summary, link, source branch, new branch, and worktree path.

## What you'll need

- A ticket tracker CLI returning summary + target release (Jira CLI, Linear, `gh issue view`)
- Git with worktree support
- A package manager appropriate to your stack

## Why this pattern works

The worktree-per-ticket model is the unsung hero — you never stash, never lose
context to a hotfix. The `{TICKET_KEY}-{slug}` naming makes branches sortable
and instantly traceable from PRs and CI logs back to the ticket. Picking the
base from the ticket's target release prevents the "I branched off the wrong
base" mistake — the most expensive kind of late-discovered mistake.

## Variations

- **No worktrees?** Replace step 6 with `git checkout -b`.
- **Monorepo?** Scope step 7's install to the affected package.
- **Always branch off main?** Drop step 5.