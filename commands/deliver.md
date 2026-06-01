# deliver

Ships the current branch: runs lint and type-check as a gate, commits with
a ticket-prefixed message, pushes, and opens a pull request with a
generated summary and test plan — failing fast if the checks don't pass or
a secret file is about to be committed.

## When to use

When your work is done and you want to go from "code is written" to "PR is
open" in one step, without forgetting the lint pass, the ticket prefix, or
the test-plan section of the PR body — and without ever committing a `.env`.

## How it works

1. **Detect context.** Read the current branch name and extract the ticket
   key from it (e.g. `proj-1234-add-endpoint` → `PROJ-1234`, uppercased).
   Determine the base branch the work should merge into — from the branch's
   upstream, or your project's default integration branch.

2. **Gate on lint.** Run your project's lint-fix command. If errors remain
   after auto-fix, show them and **stop** — don't commit broken code.

3. **Gate on type-check / build.** Run your type-checker (or build). If it
   fails, show the errors and **stop**.

4. **Review what's about to be committed.** Run `git status` and
   `git diff --stat`. Explicitly exclude anything secret-shaped — `.env`,
   `credentials.json`, key files. Never stage them, even if tracked by accident.

5. **Commit.** Stage the relevant files. Use the user's message if given,
   else generate a concise one from the diff. **Always prefix with the
   ticket key** so the commit is traceable. Add whatever co-author/footer
   trailer your team uses.

6. **Push** the branch to the remote with upstream tracking.

7. **Open the PR** against the base branch, with a body containing:
   - `## Summary` — 1–3 bullets on what changed
   - `## Test plan` — a checklist (lint, type-check, build, manual steps)
   - A link back to the ticket
   - Your team's PR footer, if any

8. **Report** the commit hash, the PR URL, and the ticket link.

## What you'll need

- Git and a PR-capable CLI (`gh`, `glab`, or your platform's equivalent)
- Lint and type-check/build commands defined in the project
- A branch-naming convention that embeds the ticket key (so step 1 works)
- A ticket tracker to link back to

## Why this pattern works

Lint and type-check as a **hard gate before commit** is the whole point —
it makes "I forgot to run the checks" structurally impossible, so red CI
from trivial mistakes stops happening. Prefixing every commit with the
ticket key keeps history traceable from any commit back to the work that
motivated it. The explicit secret-file exclusion turns the single most
expensive mistake (committing credentials) into something the command
refuses to do, rather than something you have to remember. Generating the
test-plan section means PRs ship with one instead of an empty template
nobody fills in.

## Variations

- **Branched off a release branch?** Open the PR against the release
  branch, then also open a forward-port PR to your main branch
  (cherry-pick the range onto a sibling branch); abort the forward-port on
  conflict and tell the user to finish it manually. This keeps hotfixes
  from getting lost on the next release.
- **No ticket tracker?** Drop the key prefix and the ticket link; generate
  the commit message from the diff alone.
- **Trunk-based, no PRs?** Stop after step 6 (push), or push to a
  short-lived branch and auto-merge once checks pass.
- **Monorepo?** Scope the lint/type-check commands to the affected package.
- **Pre-commit hooks already run lint?** Keep step 3 (type-check/build) as
  the gate and let the hook own lint.
