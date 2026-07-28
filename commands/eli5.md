# eli5

A slash command that re-explains the last thing — or a topic you name — in the simplest possible terms: **one everyday analogy, no jargon, a few short lines**, aimed at a smart non-engineer. Like `tldr` it's a lens on what's already known, not new work — but where `tldr` compresses for a technical reader, `eli5` translates for someone who shouldn't have to know the internals: a PM, a manager, a client, a teammate from another discipline.

## When to use

- An explanation came out too technical and someone non-technical needs to get it.
- You're about to paste a root-cause into a ticket a manager will read.
- You want to check your own understanding — if you can't say it in one analogy, you don't understand it yet.

## How it works

1. **Resolve the target.** No argument → the most recent explanation in the conversation. Argument given → that topic instead.
2. **Rewrite to a fixed four-beat shape** so a smart non-engineer gets it on the first read:
   - **What's wrong** — one plain sentence, no file/function/error names.
   - **Why**, with exactly **one everyday analogy** — a mailing list, a locked door, a queue, a light switch, a form with a missing checkbox. The analogy must map 1:1 to the real cause.
   - **The fix** — one plain sentence: what changes and what the person will see afterward.
   - **Bottom line** — one closing sentence starting with "Bottom line:".
3. **Hold the constraints:** no jargon (any unavoidable term gets a ≤3-word plain meaning in brackets); no code, paths, or acronyms; one analogy only, never stacked; concrete over abstract; ≤ ~8 lines; answer in the user's language.

## What you'll need

- An AI assistant with access to the conversation context (to grab the last explanation).
- Nothing else — it's a pure re-explanation command, no repo or tools.

## Why this pattern works

- **One analogy, mapped 1:1** is what makes an explanation land for a non-expert — and the 1:1 requirement stops the analogy from quietly lying about the real cause.
- **A rigid no-jargon rule** is the whole value: the moment "cache" or "endpoint" slips in, the audience it's written for is lost.
- **Concrete-over-abstract** ("old accounts were never added to the list" beats "the settings weren't backfilled") is what a manager can actually repeat to someone else.
- **Banning re-investigation** keeps it a translation layer, not a rabbit hole — if the cause isn't known yet, it says so in one line instead of guessing.

## Variations

- **Audience knob** — "explain for a designer" vs "for a CEO" shifts which analogy lands.
- **Keep-one-term mode** — allow a single named concept the audience already knows, plain-define the rest.
- **Topic mode** — pass a concept to get a from-scratch simple explanation, not a summary of the last answer.
- **Part of a trio** — pairs with `tldr` (same material compressed for a technical reader) and `flow` (same material as an end-to-end story). Same input, three lenses.

## Ready-to-use command (Claude Code)

Tool-agnostic prompt — the only Claude-Code-specific bit is `$ARGUMENTS` (the slash-command placeholder). On another assistant, drop it and pass the topic inline, or leave it empty to simplify the previous explanation.

````markdown
Re-explain the last thing (or a given topic) in the simplest possible terms — one everyday analogy, no jargon, a few short lines. Use when an explanation was too complex.

## Input

Topic: $ARGUMENTS

- If `$ARGUMENTS` is empty → simplify the **most recent explanation** in this conversation (the thing just described).
- If `$ARGUMENTS` has text → explain that topic simply.

## What to produce

Rewrite the explanation so a **smart non-engineer** gets it on the first read. Follow this exact shape:

1. **What's wrong** — one plain sentence. No file names, no function names, no error strings.
2. **Why (with one everyday analogy)** — 2–3 short sentences. Use a real-world picture: a mailing list, a locked door, a queue, a light switch, a form with a missing checkbox. The analogy must map 1:1 to the real cause.
3. **The fix** — one plain sentence. What changes and what the user will see afterward.
4. **Bottom line** — one final sentence starting with **"Bottom line:"**.

## Hard rules

- **No jargon.** Banned unless immediately defined in ≤3 plain words: cache, socket, async, payload, endpoint, quota gate, broadcast, middleware, marker, notification-type, backend/frontend (say "the server" / "the screen").
- **No code, no file paths, no ticket internals, no acronyms.** If a technical word is truly unavoidable, put a 3-word plain meaning in brackets right after it.
- **Short.** Whole thing ≤ 8 lines / ~120 words. If it's longer, it's not simple enough — cut.
- **One analogy only.** Don't stack metaphors.
- **Concrete over abstract.** "Old accounts were never added to the list" beats "the settings weren't backfilled."
- **Match the user's language.** If the trigger was written in Russian, answer in Russian; otherwise English.
- Do **not** re-run any investigation or tools — just restate what's already known, simpler. If the cause isn't known yet, say so in one line instead of guessing.

## Tiny example (shape only)

> **What's wrong:** The "limit reached" popup never shows.
> **Why:** The popup is like a mailing list. The alert is newer than your account, and only new accounts were auto-subscribed — so your old test account isn't on the list, and the system sends nothing.
> **Fix:** When an account isn't on the list, treat it as subscribed by default. Then the popup shows for everyone.
> **Bottom line:** Old accounts were silently left off the alert list; default them to "on."
````
