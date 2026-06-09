# review

Runs an AI code review on the current branch's changes against a base
branch, scored against a fixed checklist (bugs, security, engineering
principles, plus the repo's own conventions), writes a report, then
optionally applies the fixes you select.

## When to use

Before pushing or opening a PR, when you want a structured second pass
over your own diff — not a vague "looks good", but a checklist-driven
review that flags concrete issues with file/line and a suggested fix,
and can apply the ones you approve.

## How it works

1. **Detect the base.** Use the argument if given; otherwise the branch's
   upstream; otherwise fall back to your repo's default integration branch.
   If the base ref doesn't exist, say so and stop. Diff `{base}..HEAD`. If
   there are no changes, say so and stop.

2. **Gather the diff.** Run `git diff --stat {base}..HEAD` for the summary
   and the full `git diff {base}..HEAD` for the review. Tell the user how
   many files changed. Skip binary and generated files (lockfiles, build
   output) — they're noise. If the diff is very large, review it in batches
   so it doesn't blow the context window, and say you're doing so.

3. **Load project rules.** Look for a conventions file in the repo root
   (e.g. `CLAUDE.md`, `AGENTS.md`, a `CONTRIBUTING.md`). If present, read
   it — these are the project-specific rules to enforce, and they take
   priority over generic advice.

4. **Review against a fixed checklist:**
   - **Project rules** — every convention from the repo's rules file. Don't skip any.
   - **Errors & bugs** — missing `await`/unhandled rejections, null access
     without guards, off-by-one, inverted conditions, race conditions,
     resource leaks (unclosed streams, connections, listeners).
   - **Security (OWASP-style)** — injection, missing input validation at
     boundaries, sensitive data in responses/logs/errors, broken access control.
   - **Engineering principles** — SOLID, DRY, KISS, separation of concerns,
     error handling (swallowed errors, catch-all without re-throw), naming,
     early-return over deep nesting, oversized functions.

5. **Write a report** with a severity (ERROR / WARNING / INFO) per issue,
   each carrying file:line, what's wrong and why, and a concrete fix. Save
   it to a gitignored scratch path (e.g. `temp/review-report.md`) and show
   it. Include a final "ready to push: yes / with fixes" assessment.

6. **Offer to fix.** Ask the user which issues to apply: all, none, or a
   comma-separated subset. Make minimal, focused changes — never refactor
   unrelated code. Mark each issue FIXED or SKIP in the report as you go.

7. **Report** what was fixed vs skipped, remind the user to eyeball
   `git diff`, and print the report path.

## What you'll need

- Git, and a branch with changes to review
- An optional repo conventions file to enforce (the review still works without one)
- Write access to a scratch directory for the report

## Why this pattern works

A fixed checklist is what separates a review from a vibe-check — the model
can't quietly skip the boring categories (security, error handling) when
they're enumerated. Persisting the report to a file before touching code
gives you an auditable record and a stable set of issue numbers to select
from, so "fix 1 and 3" is unambiguous. Letting the human pick which fixes
to apply keeps the AI from "helpfully" rewriting half the file — the
review surfaces, the human decides.

## Variations

- **CI gate?** Stop after step 5 and exit non-zero if any ERROR-severity
  issue is found — no interactive fix step.
- **Review a PR, not a branch?** Replace step 1's diff with the PR diff
  (`gh pr diff`, or your platform's equivalent).
- **Stricter scope?** Add a "no new dependencies / no public API changes"
  rule to the checklist for hotfix branches.
- **Team conventions live elsewhere?** Point step 3 at your shared style
  guide instead of a repo-root file.
