# ai-recipes

A pattern library for AI-assisted development commands.

Each recipe describes the **idea** of a command — the problem it solves,
the algorithm, the tradeoffs — without committing to a specific stack
or tool. You hand the recipe to your AI assistant (Claude Code, Cursor,
Cline, Continue, or any other), and it generates a command tailored to
**your** ticket tracker, **your** CI, **your** conventions.

## Why concepts, not implementations?

Implementations rot. Tools rename. Internal URLs leak into commits.

A concept is a durable pattern. It teaches the *why* and lets the AI
handle the *how* for your specific environment. The same recipe works
whether you use Jira or Linear, Jenkins or GitHub Actions, npm or uv.

## How to use a recipe

1. Browse [commands/](commands/) and pick one that solves a problem you have.
2. Paste it into your AI assistant with a prompt like:

   > Here's a recipe for a slash command. Build me a version for my stack:
   > ticket tracker = Linear, CI = GitHub Actions, package manager = pnpm.
   > Format it for my AI tool (Claude Code slash command, Cursor custom
   > command, Cline mode, etc.).

3. The AI generates a command tailored to your setup. Tweak as needed.

## Recipes

### Commands

- [**start-work**](commands/start-work.md) — turn a ticket reference into
  a ready-to-code workspace: branch in an isolated worktree, dependencies
  installed, env files copied.
- [**finish-work**](commands/finish-work.md) — safely tear down a worktree
  when work is done: safety checks for uncommitted changes and unpushed
  commits, then remove the worktree, optionally delete the local branch.
- [**qa-handoff**](commands/qa-handoff.md) — post a fix summary and QA
  test plan to the ticket so QA picks it up cleanly.
- [**review**](commands/review.md) — review the current branch's diff
  against a fixed checklist (bugs, security, conventions), then apply the
  fixes you select.
- [**error-triage**](commands/error-triage.md) — turn an error-tracker
  issue into a structured root-cause analysis with stacktrace, impact, and
  a suggested fix.
- [**ci-debug**](commands/ci-debug.md) — triage a failing CI build: fetch
  the log, classify code-vs-infra, suggest a fix — read-only, log kept out
  of context.
- [**deliver**](commands/deliver.md) — ship the current branch: gate on
  lint and type-check, commit with the ticket key, push, and open a PR.
- [**ticket-create**](commands/ticket-create.md) — file a tracker ticket
  from the conversation, deriving the summary and body from context without
  fabricating details.

## License

[MIT](LICENSE)