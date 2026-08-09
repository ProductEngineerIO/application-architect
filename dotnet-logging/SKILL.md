---
name: dotnet-logging
description: Evaluate, review, and validate .NET structured logging and observability implementations. Use when assessing logging patterns, reviewing log statements or correlation ID usage, evaluating production visibility and tracing, or validating architecture stories involving observability and monitoring.
---

# Structured Logging & Observability Guidance

You are evaluating or helping with structured logging and observability in .NET — using structured, correlatable log properties rather than string-concatenated messages, so requests can be traced through production systems. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with structured logging?
> A) Evaluate my existing logging setup
> B) Review a specific code change
> C) Learn/write logging patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my logging changes" clearly means Review; a bare `/dotnet-logging` does not).

## Evaluation Context

Use when the user asks to **evaluate** their logging implementation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: `ILogger<T>` usage and whether messages use structured properties (`{PropertyName}`) or string interpolation/concatenation, presence of correlation/trace IDs threaded through calls, what's logged at request start/end and around external calls, and log level usage (is everything `Information`, or are levels meaningful?).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual files/classes and what you found (e.g. "`OrderController` logs `$"Order {id} failed"` — string-interpolated, not structured, so it can't be queried by order ID in the log backend")
   - Flag concrete risks: no correlation ID propagation across service calls, missing logging around external dependency calls, sensitive data (PII, secrets) logged in plaintext, log levels that don't distinguish real problems from noise
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** logging code or changes.

Before reviewing, determine the source of the code. If it isn't already clear (e.g. the user already pasted code, or already named a commit/hash), ask:

> How would you like to give me the code to review?
> A) Paste the code directly
> B) Pick from your last 10 commits
> C) Give me a commit hash or range

- If **B**: run exactly `git log -10 --oneline` (add `-- <path>` only if the user already narrowed it to a project/service). Do not run any other git commands yet — just show the resulting list with numbers and stop to let them pick. Only after they pick a specific commit, run `git show <hash>` or `git diff <hash>^ <hash>` to get that one diff.
- If **C**: take the hash/range they give and run `git show <hash>` (single commit) or `git diff <hash1> <hash2>` (range) to get the diff.
- If **A**: wait for them to paste it.
- If there's no git repo available (e.g. `git log` fails) or they're not in a repo context, skip straight to asking them to paste the code.

Once you have the code (pasted or pulled from git), review it — don't just restate it. Focus on:
- Structured property usage (`{PropertyName}` templates, not string interpolation)
- Correlation ID presence and propagation across the call
- Appropriate log level for the situation (would this actually help someone at 3am?)
- No sensitive data (passwords, tokens, PII) landing in log output
- Whether failures are logged with enough context to diagnose without reproducing

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic logging advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** logging patterns.

Provide:
- `ILogger<T>` usage with structured message templates
- Correlation ID setup and propagation (e.g. via middleware, `ActivityId`, or a custom header)
- Guidance on what to log at request boundaries and around external calls
- Log level guidance (when to use Debug vs Information vs Warning vs Error)

## Validation Context

Use when the user asks to **validate** a story or requirement involving logging or observability.

Check that requirements address:
- What needs to be traceable end-to-end (which operations need correlation IDs)
- What should be logged at each step for production diagnosis
- Whether alerting/dashboards are called out as part of the work, not just raw logs
- Whether sensitive data handling in logs is considered

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at error handling instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
