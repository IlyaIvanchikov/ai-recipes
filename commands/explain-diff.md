# explain-diff

A slash command that explains your current git changes **for review** — a short per-file "what + why", a summary table, a **Definition of Done**, and one good-vs-bad example of the key pattern the change establishes. It's **read-only by contract**: it inspects and explains, never stages, commits, or edits. Two modes: all-at-once (default) or a step-through that walks one file per turn. Think of it as the reviewer's-eye pass — it turns "here are 12 changed files" into a story a reviewer (or your future self) can actually follow.

## When to use

- Before opening a PR — sanity-check that the diff says what you meant it to.
- Reviewing someone else's branch and you want the intent, not just the lines.
- Writing a PR description — the per-file What/Why + the summary table are most of it already.
- Coming back to a branch you left a week ago and needing to remember what it does.

## How it works

1. **Collect the changeset (read-only).** Status + staged/unstaged/untracked; or the diff vs a ref; or scoped to a path. Never `git add` / `commit` / edit — this command only reads.
2. **Anchor the intent.** Ground the "why" in the branch name, recent commit messages, and any design/plan/spec doc or `CLAUDE.md` in the repo — so the explanation and the Definition of Done are judged against your own conventions, not invented. No context → infer from the diff.
3. **Explain.**
   - **All-at-once (default):** per-file **What + Why** (What becomes a short bullet list when a file bundles 2+ distinct sub-changes) → a **summary table** (`File | What changed | Why`) → a **Definition of Done** (testable criteria; `[x]` only what was actually verified this run, `[ ]` for the rest) → one **good-vs-bad** example of the key pattern → close.
   - **Step mode (`step`):** list the files in review order with a total `N`, then explain **one file per turn** led by a `k/N` progress counter; "go" for the next, "skip" to jump straight to the final wrap-up. The whole-changeset table + Definition of Done come after the last file.
4. **Flags & scope.** `--skip-tests` drops test files from the whole changeset before explaining (they don't count toward `N` or the table). A ref (`main`, a SHA) scopes to the diff vs that ref; a path scopes to that file/dir.

## What you'll need

- Git — for the changeset (works in any git repo).
- An AI assistant that can read the working tree and run read-only git commands.
- Optional but better: a plan / spec / `CLAUDE.md` in the repo, so "why" and the Definition of Done are checked against your real conventions.

## Why this pattern works

- **"What + Why per file, Why tied to a rule or plan"** catches the change that works but doesn't match intent — exactly what a raw line-by-line diff hides.
- **A Definition of Done that marks `[x]` only what was truly verified this run** refuses to fake confidence — the unchecked boxes *are* the reviewer's to-do list.
- **One good-vs-bad example** names the pattern the change establishes or guards against, so the reviewer walks away with the rule, not just this one instance.
- **Read-only by contract** keeps *explain* and *commit* separate — understanding a diff should never mutate it.
- **Step mode paces a big review** — one file per turn beats a wall of twelve explanations nobody scrolls through.

## Variations

- **`--skip-tests`** — review only the production change when the test churn is noise.
- **Scope to a ref/path** — explain the diff vs `main`, or just one directory.
- **PR-description mode** — keep the summary table + Definition of Done as the PR body.
- **Chain it** — `explain-diff` tells you *what changed and why*, `distill` cleans it up, a bug-focused `review` hunts correctness. Different passes, run in sequence.

## Ready-to-use command (Claude Code)

Tool-agnostic — it only needs git. The one Claude-Code-specific bit is the argument placeholder (`<ARG>`, where the slash-command argument lands); on another assistant, pass the mode keyword / flag / ref inline instead.

````markdown
Explain the current changes for review: a short, clear per-file "what + why", a summary table, a Definition of Done, and a good-vs-bad example of the key pattern. Read-only — never stages, commits, or edits anything. Works in any git repo. Two modes: all-at-once (default) or step-through one file at a time.

## Guard

Confirm the working directory is inside a git repo (`git rev-parse --is-inside-work-tree`). If not, tell the user "Not a git repository — nothing to explain." and stop.

## Input

The argument may contain a **mode keyword**, a **flag**, and/or a **ref/path** (any order, any combination): <ARG>

- **Mode keyword** (optional): `step` (aliases `--step`, `-s`, `walk`) → **step mode**. Absent → **all-at-once mode** (default).
- **Flag** (optional): `--skip-tests` → exclude test files from the changeset entirely, before any other step. A file counts as a test file if it matches the repo's own test convention (check CLAUDE.md/contributing guide first) or a common one: lives under `test/`, `tests/`, `__tests__/`, `spec/`; or its name matches `*.test.*`, `*.spec.*`, `*_test.*`, `test_*.*`, or a `*Test.*` suffix. Excluded files never appear in the per-file walk, the file count `N`, the summary table, or the step list — they're dropped, not just hidden.
- **ref/path** (optional, the remaining token after removing the mode keyword and flag):
  - empty → all current changes: staged + unstaged + untracked, and (if on a feature branch) the diff vs its base/upstream.
  - a git ref (e.g. `main`, `develop`, a tag/SHA) → the diff of the current branch **vs that ref**.
  - a path → scope to that file/dir.

Examples: `/explain-diff` · `/explain-diff step` · `/explain-diff step main` · `/explain-diff main` · `/explain-diff --skip-tests` · `/explain-diff step --skip-tests main`.

## Steps (both modes share 1–2)

1. **Collect the changeset** (read-only):
   - `git status --porcelain` (modified/added/untracked).
   - `git diff` (unstaged) + `git diff --cached` (staged); for a ref arg, `git diff <ref>...HEAD`.
   - For untracked files, read them directly.
   - If `--skip-tests` is set, drop test files (per the matching rule in Input) from the changeset now, before any explanation step — they don't get a What/Why, don't count toward `N`, don't appear in the table.
   - Never run `git add`, `git commit`, or modify files — this command only reads.

2. **Anchor the intent:** look for context to ground the "why" — branch name, recent commit messages (`git log --oneline -5`), and any design/plan/spec doc in the repo (under `docs/`, a `*-design.md` / `*-plan.md`, an ADR, a referenced issue/ticket). Read the repo's CLAUDE.md / contributing guide if present, to judge "why" + DoD against its own conventions. If no context, infer from the diff.

**(Both modes) Python technique notes:** if the repo is a Python project and a changed file uses non-standard or non-obvious Python — advanced idioms, decorators, metaclasses/metaprogramming, comprehension/generator tricks, async/await, context managers, typing constructs (e.g. `Protocol`, `overload`, `assert_never`), dataclasses/`__slots__` — add a one-line **Note** at the **end** of that file's explanation (after What/Why and any good/bad snippet) saying what the technique does and why, in plain terms for a reader coming from a Node.js/JS background (compare to the JS equivalent where it helps). Skip for ordinary, idiomatic code.

### All-at-once mode (default)

3. **Per-file explanation** — for EACH changed file: **What** + **Why**. **Why** stays 1 line (intent/rationale tied to the rule/plan/convention it serves). **What** is 1 line if the change is a single thing; if it bundles 2+ distinct sub-changes (new param, new branch, multiple call sites, etc.), switch **What** to a short bullet list — one bullet per sub-change, each bullet a fragment not a sentence, no nested bullets. Short and clear either way — no walls of text, no re-pasting the whole diff.
4. **Summary table:** `| File | What changed | Why |`.
5. **Definition of Done** — checkbox list of *testable* done-criteria (tests green, type-checker/linter clean, the specific behavior holds, docs updated, no secret/PII, no magic values, follows conventions). Tie to the plan/spec where one exists. `[x]` only what you actually verified this run; `[ ]` for the rest. If `--skip-tests` was used, leave test-related criteria `[ ]` and note they weren't reviewed.
6. **Good vs bad example** — the single key pattern the change establishes or guards against, side by side (2–8 lines each): ✅ Good (what the change does) / ❌ Bad (the tempting wrong way it avoids).
7. **Close:** note it was read-only; committing is a separate explicit step. If `--skip-tests` was used, also note how many test files were excluded.

### Step mode (`step`)

3. **List the files** to walk, in a sensible review order (e.g. constants/types → implementation → wiring → tests, or simply the changeset order). State the total `N`: "N files — I'll explain one at a time; say **go** for the next, or **skip** to jump straight to the final summary."
4. **Explain the FIRST file only.** Lead with the progress counter **`1/N — <path>`**, then: its **What** + **Why** (same shape as all-at-once — What as a bullet list when it bundles 2+ sub-changes), plus — if it helps — a tiny good/bad snippet specific to that file. No per-file Definition of Done. Then **STOP** and wait.
5. **On the user's "go"** (or "next"): explain the **next** file the same way — lead with the progress counter **`k/N — <path>`** (e.g. `4/11`, `5/11`), then What + Why and an optional snippet — then stop. If the user asks a question instead of "go", answer it, then stay on the current file until they say go.
   - **On the user's "skip"** (or "skip rest", "finish", "wrap up"): end the per-file walk immediately. "skip" always means skip the *rest of the walk*, never just the current file. Do not explain the remaining files one by one — go straight to the final wrap-up (step 6), where the summary table covers **all** files, walked and unwalked alike.
6. **After the LAST file:** present the cross-cutting **summary table** + a whole-changeset **Definition of Done** + one **good-vs-bad example** for the entire changeset, then close (read-only; commit is separate). If `--skip-tests` was used, note how many test files were excluded.

## Notes

- Read-only by contract: inspect + explain only. No staging, committing, or editing.
- Optimize for review speed: short per-file lines, one summary table, one good/bad pair. DoD differs by mode — all-at-once: a single changeset DoD; step mode: no per-file DoD, only a final whole-changeset DoD after the last file.
- Step mode spans multiple turns — keep state in the conversation: remember which files are done and resume at the next unreviewed one on each "go".
- If the changeset is large (all-at-once), group by area and still give the table + DoD; keep per-file lines terse.
````
