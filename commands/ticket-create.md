# ticket-create

Creates a tracker ticket assigned to you, with an optional type (bug /
task) and an optional description. When you don't give a description, it
derives the whole ticket — summary and structured body — from the current
conversation, without fabricating details.

## When to use

Right after you've debugged something in a session and want to capture it
as a ticket — the failing system, the symptom, the root cause, the
affected PRs/files are all fresh in the conversation, and you'd rather not
retype them into a web form. Also handy for quick one-liners: a hint plus
a type, and the command structures the rest.

## How it works

1. **Parse the input.** An optional leading type token (`bug` / `task`,
   case-insensitive) and the rest as a free-text description hint. Both are
   optional — `bug`, `task Add /metrics endpoint`, a bare hint, or nothing
   at all are all valid. Default the type to **bug**.

2. **Compose the ticket.** You need a one-line **summary**, a **type**, and
   a markdown **body**.
   - **With a hint** — use it as the seed and expand into sections
     (Problem, Context, Proposed fix, Links), but only include sections you
     can actually fill from the hint plus session context.
   - **Without a hint** — derive everything from the conversation: scan for
     the most recent problem discussed, and identify the failing system,
     the symptom, the root cause (if known), the affected builds/PRs/files,
     and any proposed fix.
   - Generate a summary that names the affected system *and* the symptom,
     so the title is searchable later.

3. **Show the draft and wait.** Display type, summary, full rendered body,
   assignee, project, and priority (e.g. High for bug, Medium for task —
   call it out if you deviate). Don't create anything until the user
   confirms; apply any edits and re-show.

4. **Create** via your ticket tracker's CLI, passing the body through a
   shell variable or heredoc so newlines and markdown survive intact.

5. **Report** the created ticket URL and a one-line confirmation (type,
   summary, assignee).

## What you'll need

- A ticket tracker CLI that can create issues with type, priority,
  assignee, and a markdown body (Jira CLI, `gh issue create`, Linear, etc.)
- Sensible project/assignee defaults configured, so a zero-argument
  invocation still produces a complete ticket
- A warm conversation, for the derive-from-context path to be useful

## Why this pattern works

The derive-from-session path is the real value: by the time you decide to
file a ticket, the conversation already contains a *better* description
than you'd write from memory — the actual error, the actual root cause,
the actual files. Pulling from it beats retyping and beats re-investigating.
The hard rule that makes this safe is **never fabricate** — no invented
build numbers, PR links, file paths, or stack traces. If a detail isn't in
the session, the section gets left out rather than guessed. The
mandatory draft-and-confirm step keeps a generated ticket from going live
with a wrong title or a hallucinated detail, which is exactly the failure
mode that makes people distrust auto-filed tickets.

## Variations

- **GitHub Issues / Linear / Notion?** Swap the create CLI; the
  parse → derive → draft → confirm → create shape is identical.
- **Team-assigned, not self-assigned?** Change the default assignee, or
  add a project/assignee token to the input grammar.
- **Chain from error-triage?** Feed a root-cause analysis straight in as
  the description hint, skipping the manual summary.
- **Strict templates?** Replace the freeform body sections with your
  tracker's required template fields, still filled from context.
