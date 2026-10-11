# explain-diff

A slash command that explains your current git changes **for review** — a short per-file "what + why", a summary table, a **Definition of Done**, and one good-vs-bad example of the key pattern the change establishes. It's **read-only by contract**: it inspects and explains, never stages, commits, or edits. Two modes: all-at-once (default) or a step-through that walks one file per turn. Think of it as the reviewer's-eye pass — it turns "here are 12 changed files" into a story a reviewer (or your future self) can actually follow.

## When to use

- Before opening a PR — sanity-check that the diff says what you meant it to.
- Reviewing someone else's branch and you want the intent, not just the lines.
- Writing a PR description — the per-file What/Why + the summary table are most of it already.
- Coming back to a branch you left a week ago and needing to remember what it does.

## How it works

1. **Collect the changeset (read-only).** Status + staged/unstaged/untracked; or the diff vs a ref; or scoped to a path. Never `git add` / `commit` / edit — this command only reads.
2. **Put the files in review order.** Not the order git lists them (by path), but the order a reviewer should read them: the **cause** (types, schemas, config, the renamed symbol) → the **implementation** → the **wiring** (callers, routes, UI) → **tests** → **docs**. Inside a layer, a file comes after the files it imports. The order is printed up front as a numbered list, so you can see the path and object to it.
3. **Anchor the intent.** Ground the "why" in the branch name, recent commit messages, and any design/plan/spec doc or `CLAUDE.md` in the repo — so the explanation and the Definition of Done are judged against your own conventions, not invented. No context → infer from the diff.
4. **Explain.**
   - **All-at-once (default):** per-file **What + Why** (What becomes a short bullet list when a file bundles 2+ distinct sub-changes) → a **summary table** (`File | What changed | Why`) → a **Definition of Done** (testable criteria; `[x]` only what was actually verified this run, `[ ]` for the rest) → one **good-vs-bad** example of the key pattern → close.
   - **Step mode (`step`):** list the files in review order with a total `N`, then explain **one file per turn** led by a `k/N` progress counter; "go" for the next, "skip" to jump straight to the final wrap-up. The whole-changeset table + Definition of Done come after the last file.
5. **Flags & scope.** Each flag has a one-letter short form, and short forms cluster (`-tdc`):
   - `--skip-tests` / `-t` drops test files from the whole changeset before explaining (they don't count toward `N` or the table).
   - `--skip-docs` / `-d` drops documentation files (README, changelog, `docs/`, `*.md`…) the same way.
   - `--core` / `-c` explains only the files a reviewer has to think about, and **folds** the rest (mechanical fallout of a rename or lint rule, generated files, format-only and prose-only changes) into one group row per cause, so nothing disappears silently.
   - A ref (`main`, a SHA) scopes to the diff vs that ref; a path scopes to that file/dir.

## What you'll need

- Git — for the changeset (works in any git repo).
- An AI assistant that can read the working tree and run read-only git commands.
- Optional but better: a plan / spec / `CLAUDE.md` in the repo, so "why" and the Definition of Done are checked against your real conventions.

## Why this pattern works

- **"What + Why per file, Why tied to a rule or plan"** catches the change that works but doesn't match intent — exactly what a raw line-by-line diff hides.
- **A Definition of Done that marks `[x]` only what was truly verified this run** refuses to fake confidence — the unchecked boxes *are* the reviewer's to-do list.
- **One good-vs-bad example** names the pattern the change establishes or guards against, so the reviewer walks away with the rule, not just this one instance.
- **Read-only by contract** keeps *explain* and *commit* separate — understanding a diff should never mutate it.
- **Review order turns a diff into a story** — read the cause first and every later file makes sense on arrival; read alphabetically and you meet a test before the code it proves, or a caller before the function it calls.
- **Step mode paces a big review** — one file per turn beats a wall of twelve explanations nobody scrolls through.
- **Skip drops, `--core` folds** — tests and docs are known categories you can review elsewhere, so skipping removes them; a folded file is a judgment call, so it stays visible in a group row where the reviewer can pull a misfiled file back out.

## Variations

- **`--skip-tests` / `--skip-docs`** — review only the production change when test or docs churn is noise; combine them (`-td`) for code only.
- **`--core`** — for the sweep after a new lint rule, a rename or a signature change: 40 files changed, 3 carry the decision. The rest fold into group rows you can still check.
- **Scope to a ref/path** — explain the diff vs `main`, or just one directory.
- **PR-description mode** — keep the summary table + Definition of Done as the PR body.
- **Chain it** — `explain-diff` tells you *what changed and why*, `distill` cleans it up, a bug-focused `review` hunts correctness. Different passes, run in sequence.

## Ready-to-use command (Claude Code)

Tool-agnostic — it only needs git. The one Claude-Code-specific bit is the argument placeholder (`<ARG>`, where the slash-command argument lands); on another assistant, pass the mode keyword / flags / ref inline instead.

````markdown
Explain the current changes for review: a short, clear per-file "what + why", a summary table, a Definition of Done, and a good-vs-bad example of the key pattern. Read-only — never stages, commits, or edits anything. Works in any git repo. Two modes: all-at-once (default) or step-through one file at a time.

## Guard

Confirm the working directory is inside a git repo (`git rev-parse --is-inside-work-tree`). If not, tell the user "Not a git repository — nothing to explain." and stop.

## Input

The argument may contain a **mode keyword**, **flags**, and/or a **ref/path** (any order, any combination): <ARG>

- **Mode keyword** (optional): `step` (aliases `--step`, `-s`, `walk`) → **step mode**. Absent → **all-at-once mode** (default).
- **Flags** (optional, combinable). Each has a one-letter short form; the long and short spellings are the same flag:
  - `--skip-tests` (short: `-t`) → exclude test files from the changeset entirely, before any other step. A file counts as a test file if it matches the repo's own test convention (check CLAUDE.md/contributing guide first) or a common one: lives under `test/`, `tests/`, `__tests__/`, `spec/`; or its name matches `*.test.*`, `*.spec.*`, `*_test.*`, `test_*.*`, or a `*Test.*` suffix. Excluded files never appear in the per-file walk, the file count `N`, the summary table, or the step list — they're dropped, not just hidden.
  - `--skip-docs` (short: `-d`) → exclude documentation files from the changeset entirely, before any other step. A file counts as a documentation file if it matches the repo's own docs convention (check CLAUDE.md/contributing guide first) or a common one: lives under `docs/`, `doc/`, `documentation/`, or an ADR directory; or its name is `README.*`, `CHANGELOG.*`, `CONTRIBUTING.*`, `LICENSE*`; or its extension is `.md`, `.mdx`, `.rst`, `.adoc`, `.txt`. Only whole-file prose counts — comments and docstrings inside a code file are part of that code file, and an agent-instruction file the change is actually *about* (CLAUDE.md, a rules file) is documentation like any other, so name it in the close if dropping it makes the changeset look empty. Excluded files never appear in the per-file walk, the file count `N`, the summary table, or the step list — they're dropped, not just hidden.
  - `--skip-tests` and `--skip-docs` are independent and combinable — each drops its own category, and a file matching both is dropped once. If the two together drop **every** file, say so in one line ("all N changed files are tests/docs — nothing left to explain") and stop instead of printing an empty table.
  - `--core` (short: `-c`) → explain only the **core** files — the ones a reviewer has to think about — and **fold** the rest (mechanical fallout of a rule/rename, generated files, format-only and prose-only changes) into one group row per cause or kind. For the sweep that follows a new lint rule, a rename or a signature change, where dozens of files changed but only a few carry the decision. Classification rule under "`--core` classification" below. Folded files are grouped, never dropped.
  - Short forms may be written separately (`-t -d -c`) or clustered into one token (`-td`, `-tdc`, any order). `-s` is **not** a flag — it stays the step-mode keyword from above, so `/explain-diff -s -tc main` is step mode with tests skipped and `--core` on.
- **ref/path** (optional, the remaining token after removing the mode keyword and flags):
  - empty → all current changes: staged + unstaged + untracked, and (if on a feature branch) the diff vs its base/upstream.
  - a git ref (e.g. `main`, `develop`, a tag/SHA) → the diff of the current branch **vs that ref**.
  - a path → scope to that file/dir.

Examples: `/explain-diff` · `/explain-diff step` · `/explain-diff step main` · `/explain-diff main` · `/explain-diff --skip-tests` · `/explain-diff --skip-docs` · `/explain-diff --skip-tests --skip-docs` · `/explain-diff step --skip-tests main` · `/explain-diff --core` · `/explain-diff step --core --skip-tests --skip-docs main`.

Same flags, short: `/explain-diff -t` · `/explain-diff -d` · `/explain-diff -td` · `/explain-diff -c` · `/explain-diff step -tdc main`.

## Steps (both modes share 1–2)

1. **Collect the changeset** (read-only):
   - `git status --porcelain` (modified/added/untracked).
   - `git diff` (unstaged) + `git diff --cached` (staged); for a ref arg, `git diff <ref>...HEAD`.
   - For untracked files, read them directly.
   - If `--skip-tests` and/or `--skip-docs` is set, drop the matching files (per the matching rules in Input) from the changeset now, before any explanation step — they don't get a What/Why, don't count toward `N`, don't appear in the table.
   - Never run `git add`, `git commit`, or modify files — this command only reads.

2. **Anchor the intent:** look for context to ground the "why" — branch name, recent commit messages (`git log --oneline -5`), and any design/plan/spec doc in the repo (under `docs/`, a `*-design.md` / `*-plan.md`, an ADR, a referenced issue/ticket). Read the repo's CLAUDE.md / contributing guide if present, to judge "why" + DoD against its own conventions. If no context, infer from the diff.

**(Both modes) `--core` classification:** when `--core` is set, after step 2 and before any explanation step, sort every file still in the changeset into exactly one of two buckets:

- **Core** — the reviewer has to think about it: the file adds or changes behavior, a decision, a data shape or a contract; or it is the **cause** of a mechanical sweep (the lint rule, config entry, renamed symbol or changed signature the other files were updated for); or it is a test that asserts a new or changed behavior.
- **Folded** — the file's **entire** diff is one of:
  1. *mechanical fallout* — the same repeated pattern applied because of a cause that is itself in the changeset (a rename, a moved import, a new argument at every call site, an autofix rewrite such as early-return for `no-else-return`), with no new branch or behavior;
  2. *generated / derived* — lockfiles, snapshots, build output, generated clients or types, regenerated catalogs;
  3. *format-only* — whitespace, import order, quotes, trailing commas;
  4. *prose-only* — README, changelog, comments.

Rules for the sort:
- Anchor first: name the cause(s) from step 2 and the diff, then match fallout against them — a file is fallout only when its pattern traces to a named cause.
- One non-mechanical hunk makes the whole file core. When unsure, core. Folding hides detail from the reviewer; over-explaining only costs lines.
- A folded file is never dropped. It is reported inside a **group row** — one row per cause or kind — so the reviewer can pull a misfiled file back out. Group row shape: `<count> files (<common dir or glob>)` | `<kind>: <the pattern, one fragment>` | `<the cause file or rule>`. Under the table, add one line per group listing its folded paths (basenames when the row already names the dir), so the fold is checkable at a glance.
- `--core` and the `--skip-*` flags are independent: the skips remove their files first, then `--core` sorts what remains. Without `--skip-tests`, a test file that only got a mechanical update is folded and a test that asserts new behavior is core; without `--skip-docs`, a prose-only doc change is folded under the *prose-only* kind — so `--skip-docs` drops what `--core` would otherwise fold into a visible group row.
- If nothing folds, say so in one line and continue exactly as without the flag.

**(Both modes) Review order:** after the changeset is final (skips applied, `--core` sorted), put the files into the order a reviewer should read them, so the explanation reads as a story from cause to effect rather than in the order git lists them. Both modes walk the files, and build the summary table, in this order:

1. **The cause** — what everything else depends on or was changed for: types, interfaces, schemas, constants, config, migrations, the lint rule or renamed symbol behind a sweep.
2. **The implementation** — the code that carries the new behavior.
3. **The wiring** — callers, routes, handlers, registrations, UI that consumes the implementation.
4. **Tests** — after the code they prove, so the reader already knows what is being asserted.
5. **Docs** — last; they describe what came before.

Inside one layer, follow the dependency: a file comes after the files it imports from the changeset. When a file fits two layers, put it where the reviewer first needs it. State the order once, as a numbered file list, before the first explanation, so the reviewer can see the path and object to it.

**(Both modes) Python technique notes:** if the repo is a Python project and a changed file uses non-standard or non-obvious Python — advanced idioms, decorators, metaclasses/metaprogramming, comprehension/generator tricks, async/await, context managers, typing constructs (e.g. `Protocol`, `overload`, `assert_never`), dataclasses/`__slots__` — add a one-line **Note** at the **end** of that file's explanation (after What/Why and any good/bad snippet) saying what the technique does and why, in plain terms for a reader coming from a Node.js/JS background (compare to the JS equivalent where it helps). Skip for ordinary, idiomatic code.

### All-at-once mode (default)

3. **Per-file explanation** — list the files in review order (see "Review order" above), then for EACH changed file, in that order (with `--core`: each **core** file; folded files get no per-file entry): **What** + **Why**. **Why** stays 1 line (intent/rationale tied to the rule/plan/convention it serves). **What** is 1 line if the change is a single thing; if it bundles 2+ distinct sub-changes (new param, new branch, multiple call sites, etc.), switch **What** to a short bullet list — one bullet per sub-change, each bullet a fragment not a sentence, no nested bullets. Short and clear either way — no walls of text, no re-pasting the whole diff.
4. **Summary table:** `| File | What changed | Why |`, rows in review order. With `--core`: one row per core file, then one **group row** per folded group (shape from the classification rules) — the table still accounts for every file in the changeset.
5. **Definition of Done** — checkbox list of *testable* done-criteria (tests green, type-checker/linter clean, the specific behavior holds, docs updated, no secret/PII, no magic values, follows conventions). Tie to the plan/spec where one exists. `[x]` only what you actually verified this run; `[ ]` for the rest. If `--skip-tests` was used, leave test-related criteria `[ ]` and note they weren't reviewed; same for doc-related criteria ("docs updated") under `--skip-docs`. With `--core`, the DoD still spans the whole changeset — the folded files included (e.g. "lint clean across the N rewritten files").
6. **Good vs bad example** — the single key pattern the change establishes or guards against, side by side (2–8 lines each): ✅ Good (what the change does) / ❌ Bad (the tempting wrong way it avoids).
7. **Close:** note it was read-only; committing is a separate explicit step. If `--skip-tests` and/or `--skip-docs` was used, also note how many files each one excluded, counted separately (e.g. `--skip-tests: 6 test files excluded · --skip-docs: 3 doc files excluded`). If `--core` was used, also note `--core: N files folded into K groups`.

### Step mode (`step`)

3. **List the files** to walk, in review order (see "Review order" above). With `--core`, the walk list holds the **core** files only and `N` is their count; state the folded groups in the same breath ("plus 37 files folded into 2 groups — shown in the final table"). State the total `N`: "N files — I'll explain one at a time; say **go** for the next, or **skip** to jump straight to the final summary."
4. **Explain the FIRST file only.** Lead with the progress counter **`1/N — <path>`**, then: its **What** + **Why** (same shape as all-at-once — What as a bullet list when it bundles 2+ sub-changes), plus — if it helps — a tiny good/bad snippet specific to that file. No per-file Definition of Done. Then **STOP** and wait.
5. **On the user's "go"** (or "next"): explain the **next** file the same way — lead with the progress counter **`k/N — <path>`** (e.g. `4/11`, `5/11`), then What + Why and an optional snippet — then stop. If the user asks a question instead of "go", answer it, then stay on the current file until they say go.
   - **On the user's "skip"** (or "skip rest", "finish", "wrap up"): end the per-file walk immediately. "skip" always means skip the *rest of the walk*, never just the current file. Do not explain the remaining files one by one — go straight to the final wrap-up (step 6), where the summary table covers **all** files, walked and unwalked alike.
6. **After the LAST file:** present the cross-cutting **summary table** (with `--core`: core rows first, then one group row per folded group) + a whole-changeset **Definition of Done** + one **good-vs-bad example** for the entire changeset, then close (read-only; commit is separate). If `--skip-tests` and/or `--skip-docs` was used, note how many files each one excluded, counted separately. If `--core` was used, note `--core: N files folded into K groups`.

## Notes

- Read-only by contract: inspect + explain only. No staging, committing, or editing.
- Optimize for review speed: short per-file lines, one summary table, one good/bad pair. DoD differs by mode — all-at-once: a single changeset DoD; step mode: no per-file DoD, only a final whole-changeset DoD after the last file.
- Step mode spans multiple turns — keep state in the conversation: remember which files are done and resume at the next unreviewed one on each "go".
- If the changeset is large (all-at-once), group by area and still give the table + DoD; keep per-file lines terse. Inside each area, keep the review order.
- Review order is a rule, not a preference: the order git lists files in is sorted by path, which can put a test before the code it proves and a caller before the function it calls.
- The `--skip-*` flags drop, `--core` folds — on purpose. Tests and docs are known categories you can choose to review elsewhere; a folded file is a judgment call the reviewer must be able to check, so it stays visible as part of a group row. Keep the two categories on separate flags: a review that skips tests usually still wants the README change, and a docs-only review is exactly the case `--skip-tests` alone serves.
````
