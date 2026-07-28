# flow

A slash command that explains how a feature works **end-to-end as a single linear story** — page → endpoint → server → background jobs → data → what the user sees — instead of a line-by-line code trace. It's the big-picture lens: where `tldr` compresses and `eli5` translates, `flow` narrates one request from the click to the final effect, keeping the real moving parts (endpoint paths, crons, queues) but never getting precise about files or functions. If the flow hasn't been traced yet, it reads the code just enough to tell the story truthfully — then tells the story, it doesn't dump the findings.

## When to use

- Onboarding onto an unfamiliar feature — you want the shape before the detail.
- A bug spans several hops and you need the map before diving into one.
- Explaining a system to a technical teammate who hasn't seen it.
- Right before a deep-dive, to decide *which* hop is worth zooming into.

## How it works

1. **Resolve the target.** No argument → the flow most recently discussed. Argument given → that flow (investigating the code just enough to narrate it truthfully if it hasn't been traced yet).
2. **Tell one linear story**, one beat per hop:
   - **Page** — where the user is and what they do.
   - **Endpoint** — the one call that fires (method + path is enough).
   - **Server** — does it do the work now, or just record it and return?
   - **Async / background** — jobs, queues, crons, workflows that pick it up later, each named by role and cadence ("a background job, every 12 min").
   - **Data** — where state lives between hops (a table, a queue, a spreadsheet, a cache) and how it changes.
   - **What the user sees** — how and when the result surfaces, including any lag between "done" and "visible".
3. **Close** with a single "where it breaks" line (only if known) and an offer to zoom into any one hop.
4. **Hold the constraints:** roles not coordinates (no file paths, line numbers, function names, or code); one endpoint story at a time; keep the real vocabulary but stay imprecise; call out sync-vs-later at every step; ≤ ~15 lines; describe, don't fix.

## What you'll need

- An AI assistant that can read the codebase when a flow hasn't been traced yet (a code-search / file-read capability).
- The name of a feature or flow — or a recent conversation that already discussed one.

## Why this pattern works

- **A single linear narrative** is how people actually build a mental model; a call graph or a file dump is not a story and doesn't stick.
- **"Roles, not coordinates"** keeps the altitude high — the moment file paths appear, it's a trace, not an overview, and the big picture is lost.
- **Making lag explicit** ("records it and returns" vs "a job picks it up later") surfaces the async gap that is usually the whole point — and the usual source of "it didn't update" confusion.
- **One "where it breaks" line, no fix plan** keeps it a map: it tells you which hop to open next without turning into a debugging session.

## Variations

- **Zoom-in follow-up** — after the overview, deep-dive a single hop (the one beat, in full detail).
- **Data-centric** — tell the same flow from the data's point of view: which store changes at each hop.
- **Failure-flow** — narrate the unhappy path (what happens on error / retry / timeout) instead of the happy path.
- **Sequence-diagram output** — render the beats as a diagram (e.g. Mermaid) instead of prose.
- **Part of a trio** — pairs with `tldr` (same material compressed for a technical reader) and `eli5` (same material for a non-engineer, via one analogy). Same input, three lenses.

## Ready-to-use command (Claude Code)

Tool-agnostic prompt — the only Claude-Code-specific bit is `$ARGUMENTS` (the slash-command placeholder). On another assistant, drop it and name the flow inline. Unlike `tldr`/`eli5`, this one may read code, so it needs an assistant with codebase access.

````markdown
Explain how a feature or flow works end-to-end as a plain narrative — which page triggers which endpoint, what happens next, through the services, background jobs, and data, down to what the user sees. The big-picture story, not a line-by-line trace.

## Input

Flow / feature: $ARGUMENTS

- Empty → explain the **flow most recently discussed** in this conversation.
- Has text → explain that flow. If it hasn't been traced yet, investigate the code just enough to tell the story truthfully — then tell it, don't dump the findings.

## What to produce

A single **linear story** that follows one request from the user's click to the final effect. Walk it as a chain of beats, each beat one hop:

1. **Page** — where the user is and what they click/do.
2. **Endpoint** — the one call that fires (method + path is enough, e.g. `POST /projects/{id}/…`).
3. **Server** — what the backend does with it, in one breath: does it do the work now, or just record it and return?
4. **Async / background** — any jobs, queues, crons, workflows that pick it up later — name each by role and cadence ("a background job, every 12 min").
5. **Data** — where state lives between the hops (a table, a queue, a spreadsheet, a cache) and how it changes.
6. **What the user sees** — how/when the result shows up (and any lag between "done" and "visible").

Close with:
- **Where it breaks** — one line naming the fragile hop, only if it's known.
- An offer to zoom into any single hop next.

Shape: short beats joined by arrows or a tight numbered list. Name the moving parts by **what they are**, not where they live.

## Hard rules

- **Roles, not coordinates.** Name components by their job — "the generator job", "the publish endpoint", "the posts table". **No file paths, no line numbers, no function names, no code.** (This is the opposite of a deep-dive; save those for a follow-up on one hop.)
- **One endpoint story at a time.** Follow a single request path start to finish. Don't branch into every variant — mention alternates in half a sentence, max.
- **Keep the real vocabulary.** Unlike an ELI5, do keep the true moving parts (endpoint path, "n8n workflow", "cron", "queue") — the reader is technical. Just don't get precise.
- **Lag is part of the story.** Always say when a step is synchronous vs. "happens later" — that gap is usually the whole point.
- **Short.** ≤ ~15 lines / one screen. If it's longer, you're being precise, not telling a story — cut.
- **No fixing.** Describe how it works, not how to fix it. A single "where it breaks" line is allowed; a fix plan is not.
- **Match the user's language.** Russian trigger → answer in Russian; else English.

## Tiny example (shape only)

> **Page:** project → Calendar tab → clicks **"TG AI"**, fills a brief (topic, count, start, spacing), submits.
> **Endpoint:** `POST /projects/{id}/telegram-generation-tasks` — the only call.
> **Server:** doesn't generate anything — just writes one "please generate" row into a store, marked `new`, and returns. User sees "task created".
> **Later — generator job (every 12 min):** finds `new` rows → asks the LLM for N posts → writes each into a second store marked `pending`, each stamped with its send time → marks the task `done`.
> **Later — publisher job (every 1 min):** finds `pending` posts whose time has come → sends them to the channel → marks `sent`/`failed`.
> **User sees:** the calendar reads that second store, so posts show up minutes after — not on submit.
> **Where it breaks:** three writers edit the same store with no lock → posts get generated/sent twice or lost.
>
> Want me to zoom into any one hop?
````
