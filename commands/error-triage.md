# error-triage

Takes an error-tracker issue reference (URL or ID), pulls the issue
metadata, the latest event's full payload (stacktrace, request context,
breadcrumbs), and tag distributions, then produces a structured
root-cause analysis with a suggested fix — without you clicking through
the web UI.

## When to use

When a production error lands in your error tracker and you want the
whole picture in one place — what's failing, how often, for whom, with
the stacktrace and the request that triggered it — plus a first guess at
the root cause, before you decide whether to fix, dig deeper, or file a ticket.

## How it works

1. **Parse the reference.** Accept a full issue URL or a bare numeric ID.
   Pull the issue ID, the project, and the environment out of the URL's
   path/query when present.

2. **Fetch issue details** via your error tracker's CLI/API: title,
   culprit, level, first/last seen, event & user counts, status, and the
   ID of the most recent event.

3. **Fetch the latest event's full payload** (usually JSON). Extract:
   - **Exception** — the full stacktrace with source context; flag which
     frames are your application code vs library code.
   - **Request** — method, URL, query, body (truncate large bodies, note
     it). For headers, show only the useful ones and **mask any
     credential** — e.g. `Authorization: Bearer ***`, never the token.
   - **Breadcrumbs** — the last 5–10 events leading up to the error.
   - **Contexts** — runtime, app, device/release info.

4. **Fetch tag distributions** to understand spread: environment, affected
   URLs/endpoints, client/browser, release, and any custom dimension your
   setup tags events with (account, region, feature flag). Skip tags that
   aren't present.

5. **Analyze:** likely root cause (from stacktrace + message + patterns
   across events), impact (event/user counts, environments, trend up or
   down), and pattern (consistent vs intermittent; tied to a specific
   endpoint, client, or release?).

6. **Present a structured report:** header (issue, level, status,
   first/last seen, counts, environment), error, formatted stacktrace
   (relevant frames highlighted), request context, impact breakdown,
   a 2–5 sentence root-cause analysis, and a concrete suggested fix that
   references specific files/lines from the stacktrace.

7. **Open the next step:** offer to dig into specific events, implement
   the suggested fix, or check whether it's already a known issue in the
   ticket tracker.

## What you'll need

- An error-tracker CLI or API with credentials (Sentry, Rollbar, Bugsnag,
  Datadog, etc.) — load credentials from the environment, don't hardcode them
- A way to fetch a single event's full JSON, not just the issue summary
- Optionally, the application source checked out locally so the suggested
  fix can cite real file/line numbers

## Why this pattern works

The latest *event* payload is where the real signal is — the issue summary
tells you something broke, but the event's request body, breadcrumbs, and
exact stacktrace tell you *why*. Fetching tag distributions turns "an
error" into "an error affecting these environments and this release,
trending up", which is what actually decides priority. Masking
credentials in headers before they ever reach the output is the
non-negotiable safety step: error payloads routinely carry auth tokens,
and a triage note can end up pasted into a ticket or chat.

## Variations

- **Front-end errors?** Lean on the breadcrumbs and the browser/OS tag
  distribution; the "request body" section becomes the user action trail.
- **No local source?** Skip the file/line citations in the suggested fix
  and keep the analysis at the stacktrace level.
- **Want a ticket out of it?** Chain this into a "create ticket" command,
  feeding the root-cause analysis as the body.
- **High-volume issue?** Sample several recent events instead of just the
  latest, and note where they diverge.
