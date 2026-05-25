# qa-handoff

Posts two comments on a closed-out ticket — a fix summary (root cause,
fix, PRs across all touched repos) and a structured QA testing
checklist — so the work hands off cleanly to QA without anyone
re-reading the diff or the conversation thread.

## When to use

After the fix is merged and PRs are open across one or more repos, and
you want QA to pick it up without you writing the testing plan from
scratch every time. Pairs naturally with a "deliver" command that
creates the PRs.

## How it works

1. **Resolve the ticket key.** Accept it as an argument, or extract it
   from the current branch name (e.g., `PROJ-1234-thing-i-am-fixing` →
   `PROJ-1234`). If neither works, ask the user.

2. **Use the current conversation context** to gather:
   - What the root cause was
   - What the fix does
   - Which repos were touched
   - Which environments / tenants matter for testing

   Do NOT re-read code or re-analyze diffs — the information is already
   in the chat. Re-reading is slow and produces a worse summary than
   what the conversation already contains.

3. **Collect all PRs across every touched repo.** For each repo, list
   PRs whose head matches the branch (or whose URL is already in the
   conversation). Capture both open and merged, all target branches.

4. **Post the fix-summary comment** on the ticket:

   ```
   ## Fix Summary

   **Root cause:** ...
   **Fix:** ...

   **PRs:**

   *{Repo name} (brief change description):*
   - {PR URL} → {target branch} (OPEN/MERGED)
   ```

   Group by repo, include a one-line description per repo, list both
   release-branch and main-branch PRs with their state.

5. **Post the QA-testing comment** as a separate comment:

   ```
   @QA-team-handle

   ### What to test
   1. **Main scenario (the fix):**
      - <concrete clickable steps>
      - **Expected:** <new behavior>
      - **Previously:** <old broken behavior>

   2. **Regression check:**
      - <related flows>
      - **Expected:** <unchanged behavior>

   ### Environment
   - Tenant / account: ...
   - App / surface: ...
   ```

   Keep steps actionable. QA should not need to open the code to follow them.

6. **Report** the ticket link and confirm both comments are posted.

## What you'll need

- A ticket tracker CLI that can post comments (Jira CLI, Linear CLI,
  `gh issue comment`, etc.)
- A way to list PRs by branch across repos (`gh pr list --head ...`)
- The conversation context from the fix session — this command depends
  on the chat history being warm; it's not a cold-start tool.

## Why this pattern works

Two comments instead of one keeps the summary (what happened) separate
from the test plan (what to verify) — QA reads top-down and your fix
summary doesn't get buried under their checklist. Pulling context from
the chat instead of re-reading code is the unsung trick: by the time
you're handing off, the conversation contains *better* explanations
than the diff does, because you've already worked through the noise.
The "previously / expected" framing for the main scenario makes
regression review trivially auditable — anyone reading later sees both
what changed and what to check.

## Variations

- **Single-repo project?** Skip the per-repo grouping; one PR list is enough.
- **No ticket tracker?** Post as a PR comment instead, on the merged PR.
  Same structure.
- **Linear / GitHub Issues / Notion?** Swap the CLI but keep the
  two-comment shape. The structure matters more than the tool.
- **No conversation context (handing off something old)?** This command
  is the wrong tool — you'll have to re-read the PRs first.
