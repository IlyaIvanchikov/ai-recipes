# ci-debug

Takes a failing CI build URL, fetches its console log, localizes the
failure, classifies it as a code problem vs. an infrastructure problem,
and suggests a concrete fix or next step — read-only, with the log kept
out of the chat context.

## When to use

When a CI build goes red and you want a fast, structured triage: is this
*my* code (fix it) or *the build environment* (retry / ping ops)? — without
scrolling thousands of log lines yourself or dumping the whole log into
your AI assistant's context window.

## How it works

1. **Take the build URL.** If none was given, ask for it. Validate its
   shape with a cheap regex before any network call, so a typo fails fast.

2. **Check status before fetching the log.** Hit the CI server's JSON API
   once. If the build is green or still running, say so and stop — there's
   nothing to triage. Only continue on a real failure
   (`FAILURE` / `ABORTED` / `UNSTABLE`). Classify API errors too: 401/403 →
   refresh credentials; 404 → wrong/deleted build; network → check VPN/DNS.

3. **Ask for the local repo path** the build corresponds to, so the
   code-mode analysis can read real source. Validate it exists.

4. **Fetch the console log to a temp file** — never into the chat. Note
   its length. If it's tiny (< ~10 lines), just print it and ask the human
   to look; don't bother classifying.

5. **Localize the failure** with targeted searches over the temp file:
   grep for `error|failed|exception|cannot|denied|timeout|killed|exit code|fatal`,
   grep for stage/pipeline failure markers, and read the tail. Then read
   only the relevant line ranges — never load the whole log.

6. **Classify CODE vs INFRA** from the markers:
   - **CODE** — compiler/type errors, lint failures, failing tests,
     `Cannot find module`, failed DB migrations / `relation does not exist`.
   - **INFRA** — `Cannot connect to the Docker daemon`, `no space left on
     device`, registry push denied, `ETIMEDOUT`/`ECONNRESET` to a package
     registry, `Could not resolve host`, OOM-kill / `signal 9`, build timeout.
   - Both kinds present → **MIXED**.

7. **Output a chat-only suggestion** (no edits, no retriggers):
   - **CODE** — read the named file(s) from the local repo; show where it
     failed (file:line), what the log says, what's actually in the file, a
     concrete fix (before/after if useful), and a local command to verify.
   - **INFRA** — headline symptom with line numbers, why it's not a code
     issue (fails before checkout / in cleanup / in registry push), and a
     next step (retry transient, notify ops for credential/disk/registry,
     bump timeout if consistently slow).
   - **MIXED** — treat as INFRA but note: retry first; if it still fails,
     treat as code.

## What you'll need

- A CI server with a build-log API or CLI (Jenkins, GitHub Actions,
  GitLab CI, CircleCI, etc.) and credentials loaded from the environment
- The corresponding repo checked out locally (for code-mode file reads)
- Shell tools for targeted log reading (`grep`, `tail`, `sed`, `jq`)

## Why this pattern works

The single most important move is **status-check before log-fetch**:
most "failures" you're handed are already green-on-retry or still running,
and you save the whole expensive analysis. The CODE-vs-INFRA split is the
decision that actually matters — it routes the failure to the right human
(you, or ops) instead of everyone staring at the same log. Keeping the log
in a temp file and reading only grep'd line ranges is what makes this
usable on real logs: a CI log can be tens of thousands of lines, and
dumping it into context is both slow and useless. And because the whole
thing is **read-only**, it's safe to run on any build without fear of
retriggering or mutating anything.

## Why read-only matters

State this as a hard rule in the command: no edits, no commits, no
retriggers, no pipeline-config changes. Also redact anything
credential-shaped (tokens, passwords, cloud keys) from log excerpts before
printing, keep each excerpt small (≤ ~30 lines), and never read files
outside the supplied repo path. Triage builds trust; mutation breaks it.

## Variations

- **GitHub Actions / GitLab CI?** Swap the log-fetch and JSON-status calls
  for that platform's API; the classify-and-route logic is identical.
- **Monorepo?** In step 3, ask which package the build maps to and scope
  the file reads there.
- **No local checkout?** Skip step 3 and stay in log-only mode — you lose
  the "what's actually in the file" section but keep the classification.
- **Auto-retry transient failures?** Only if you relax the read-only rule
  deliberately — and even then, retry only on INFRA classification.
