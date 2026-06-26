# distill

A slash command that runs a quality-only cleanup pass over your current diff — checking for reuse, simplification, efficiency, and abstraction-level (altitude) issues — by fanning out four parallel review agents and applying the safe fixes. It deliberately does **not** hunt for correctness bugs; that belongs to a separate bug-focused review. The point is fast, focused polish: distill code that already works down to its leaner form.

## When to use

- After a feature works but the diff feels messy or verbose.
- After generating code with an AI assistant (often duplicated or over-built).
- Before opening a PR, when you want a cleanup pass without a full bug audit.
- Any time you'd otherwise eyeball-refactor by hand and miss half of it.

## How it works

0. **Gather the diff.** Run `git diff @{upstream}...HEAD` (fallback `main...HEAD` / `HEAD~1`); if the range is empty or there are uncommitted changes, also include `git diff HEAD`. If a PR/branch/path was passed as an argument, scope to that instead.
1. **Review — 4 agents in parallel.** Launch four independent agents in one batch (so they run concurrently), each handed the diff and exactly one angle. Each returns findings as `file` / `line` / one-line `summary` / concrete cost (what is duplicated, wasted, or harder to maintain):
   - **Reuse** — new code re-implementing an existing helper; grep shared/util modules, name the helper to call instead.
   - **Simplification** — redundant/derivable state, copy-paste variants, deep nesting, dead code; name the simpler form.
   - **Efficiency** — redundant computation/repeated I/O, serial independent work, blocking work on hot paths, closures that pin large scopes alive; name the cheaper alternative.
   - **Altitude** — fixes done at the wrong depth; special cases bolted onto shared infrastructure signal the fix isn't deep enough — prefer generalizing the mechanism.
2. **Apply.** Wait for all four, dedup findings hitting the same line/mechanism, fix each remaining one. Skip anything that would change intended behavior, reach well outside the diff, or is a false positive — and note the skip rather than arguing with it. Finish with a short summary of what was fixed vs skipped (or confirm it was already clean).

## What you'll need

- An agent runtime that can spawn parallel sub-agents (a Task/sub-agent tool or equivalent).
- Git, for scoping the diff under review.
- Write access to the working tree, to apply the fixes.

## Why this pattern works

- **Separating cleanup from bug-hunting** keeps each pass focused; bundling them dilutes both and buries the cleanup signal under bug noise.
- **One narrow lens per agent** beats one agent told to "find everything" — a single angle misses less than a catch-all sweep.
- **"Name the concrete cost"** forces actionable findings instead of vague style nits.
- The **altitude angle** catches bandaid fixes — the failure mode most reviewers skip right over.
- **Auto-apply with explicit skip-and-explain** is fast but reversible-by-reason: nothing silently changes behavior.

## Variations

- **+ Conventions angle:** add a 5th agent that checks a project style guide / `CLAUDE.md` for rule violations (quote the exact rule + line).
- **Small diffs:** skip the fan-out, run a single agent across all angles.
- **Comment-only mode:** post findings as review comments instead of applying — useful for reviewing someone else's PR.
- **Targeted:** pass a file path or PR reference to review a specific scope instead of the working diff.

## Ready-to-use command (Claude Code)

This is the actual slash command behind the recipe. Two parts are Claude-Code-specific: `$ARGUMENTS` (the slash-command placeholder) and the Task-tool parallel fan-out in Phase 1 — on a runtime without parallel sub-agents, drop to the "Small diffs" variation above and run one agent across all four angles.

````markdown
Review the changed code for reuse, simplification, efficiency, and altitude cleanups, then apply the fixes. Quality only — it does not hunt for bugs; use /code-review for that.

## Input

An optional review target — a PR number, branch name, or file path. If empty, the current diff is reviewed: $ARGUMENTS

You are improving the quality of the changed code, not hunting for bugs. Review
it for reuse, simplification, efficiency, and altitude issues, then fix what you
find. Do not look for correctness bugs — that is what `/code-review` is for.

## Phase 0 — Gather the diff

Run `git diff @{upstream}...HEAD` (or `git diff main...HEAD` / `git diff HEAD~1`
if there's no upstream) to get the unified diff under review. If there are
uncommitted changes, or the range diff is empty, also run `git diff HEAD` and
include the working-tree changes in scope — the review often runs before the
commit. If a PR number, branch name, or file path was passed as an argument
($ARGUMENTS), review that target instead. Treat this diff as the review scope.

## Phase 1 — Review (4 cleanup agents in parallel)

Launch **4 independent review agents** via the Task tool, all in a
single message so they run concurrently. Pass each agent the diff and one of
the four angles below. Each returns its findings with `file`, `line`, a
one-line `summary`, and the concrete cost (what is duplicated, wasted, or
harder to maintain).

### Reuse

Flag new code that re-implements something the codebase
already has — Grep shared/utility modules and files adjacent to the change,
and name the existing helper to call instead.

### Simplification

Flag unnecessary complexity the diff adds: redundant or derivable state,
copy-paste with slight variation, deep nesting, dead code left behind. Name
the simpler form that does the same job.

### Efficiency

Flag wasted work the diff introduces: redundant computation or repeated I/O,
independent operations run sequentially, blocking work added to startup or
hot paths. Also flag long-lived objects built from closures or captured
environments — they keep the entire enclosing scope alive for the object's
lifetime (a memory leak when that scope holds large values); prefer a
class/struct that copies only the fields it needs. Name the cheaper
alternative.

### Altitude

Check that each change is implemented at the right depth, not as a fragile
bandaid. Special cases layered on shared infrastructure are a sign the fix
isn't deep enough — prefer generalizing the underlying mechanism over adding
special cases.

## Phase 2 — Apply the fixes

Wait for all four agents to complete, dedup findings that point at the same
line or mechanism, and fix each remaining one directly. Skip any finding whose
fix would change intended behavior, require changes well outside the reviewed
diff, or that you judge to be a false positive — note the skip rather than
arguing with it. Finish with a brief summary of what was fixed and what was
skipped (or confirm the code was already clean).
````
