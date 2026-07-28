# tldr

A slash command that re-explains the last answer — or a topic you name — as a short, skimmable digest: a one-line **TL;DR**, then a compact table or a few bullets, and an optional decision line. It is a **lens on what's already known**, not new work: it never re-runs tools or re-investigates, it just restates the same material in a form a technical reader can grasp in seconds instead of re-reading a wall of text.

## When to use

- The assistant just produced a long or dense answer and you want the fast version.
- You need a summary compact enough to paste into a ticket or a chat message.
- You're comparing options / can-vs-can't / before-vs-after — a table reads faster than prose.
- Any time you'd otherwise skim a wall of text and hope you caught the important line.

## How it works

1. **Resolve the target.** No argument → the most recent assistant answer in the conversation. Argument given → that topic instead.
2. **Emit a fixed shape, in this order:**
   - **TL;DR** — one plain sentence: the single most important takeaway.
   - **A compact 2–3 column table OR ≤5 bullets** — a table whenever 2+ items are compared or listed (options, can/can't, before/after, field lists); bullets for a short sequence or a few facts.
   - **A decision line** — only if the reader must choose: `👉 You choose:` then 2–3 one-line options.
3. **Hold the constraints:** lead with the answer, not the reasoning (reasoning earns space only if it changes the decision); plain words (define any jargon in ≤3 words inline); one idea per row/bullet; whole thing ≤ ~12 lines; answer in the user's language.

## What you'll need

- An AI assistant with access to the conversation context (to grab the last answer).
- Nothing else — no repo, no tools, no network. It's a pure re-explanation command.

## Why this pattern works

- **A fixed output shape forces compression.** A bare "summarize this" drifts straight back into prose; TL;DR → table/bullets → decision does not.
- **Lead-with-the-answer** front-loads the one line the reader actually needs; everything else is optional context.
- **Prefer-a-table** is the single biggest readability win for anything being compared — the eye scans columns far faster than sentences.
- **Banning re-investigation** keeps it instant and cheap: it re-frames existing knowledge, it doesn't redo the work. If something is genuinely unknown, it says so in one line instead of digging.

## Variations

- **`/short` alias** — same behavior, shorter trigger.
- **Table-only / bullets-only** — force one shape when you already know the content type.
- **Topic mode** — pass a topic to get a fresh digest instead of summarizing the last answer.
- **Length knob** — cap at N lines: tighter for a chat reply, roomier for a ticket.
- **Part of a trio** — pairs with `eli5` (same material for a non-engineer, via one analogy) and `flow` (same material as an end-to-end story). Same input, three lenses.

## Ready-to-use command (Claude Code)

Tool-agnostic prompt — the only Claude-Code-specific bit is `$ARGUMENTS` (the slash-command placeholder). On another assistant, drop it and pass the topic inline, or leave it empty to summarize the previous answer.

````markdown
Re-explain clearly and prettily — a short, structured version of the last answer (or a given topic). Alias intent: "/short".

## Input

Optional topic: $ARGUMENTS

- Empty → re-explain **the most recent assistant answer** in this conversation.
- Has text → explain that topic.

## Output shape (follow exactly)

Keep it short and skimmable. No wall of text. Use, in this order:

1. **TL;DR:** — one plain sentence. The single most important takeaway.
2. **A compact table OR ≤5 bullets** — whichever fits the content:
   - Comparisons, options, can/can't, before/after, field lists → **table** (2–3 columns).
   - A short sequence or a few facts → **bullets**.
3. **Decision line (only if the user must choose):** `👉 You choose:` then the 2–3 options in one line each.

## Hard rules

- **Plain words.** No jargon unless you define it in ≤3 words in brackets right after.
- **Short.** Whole thing ≤ 12 lines. If longer, cut — brevity is the point.
- **Prefer a table** whenever there are 2+ items being compared or listed; tables read faster than prose.
- **Lead with the answer**, not the reasoning. Reasoning only if it changes the decision.
- **One idea per row/bullet.** No nested sub-points.
- Match the user's language: if they wrote in Russian, answer in Russian; else English.
- Do **not** re-run tools or re-investigate — just restate what's already known, clearer. If something is genuinely unknown, say so in one line.

## Tiny example (shape only)

> **TL;DR:** The migration runs itself on the next deploy — nothing manual.
>
> | Step | What happens |
> |---|---|
> | App starts | applies pending migrations |
> | Records it | won't run twice |
>
> 👉 You choose: deploy now · or wait for the next release.
````
