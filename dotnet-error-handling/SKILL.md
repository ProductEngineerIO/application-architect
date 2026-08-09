---
name: dotnet-error-handling
description: Evaluate, review, and validate .NET error handling and Result-type implementations. Use when assessing error handling patterns, reviewing try-catch or Result<T> usage, evaluating circuit breakers and retry logic, or validating architecture stories involving failure handling and resilience.
---

# Error Handling & Result Types Guidance

You are evaluating or helping with error handling patterns in .NET — systematic error management via Result types, circuit breakers, and retry logic, as opposed to scattered try-catch blocks. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with error handling?
> A) Evaluate my existing error handling approach
> B) Review a specific code change
> C) Learn/write error handling patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my error handling changes" clearly means Review; a bare `/dotnet-error-handling` does not).

## Evaluation Context

Use when the user asks to **evaluate** their error handling implementation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: try-catch block usage and what's caught, any `Result<T>` or similar functional-error patterns, circuit breaker/retry libraries (e.g. Polly) and their configuration, and how errors propagate out of services into API responses.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual methods/classes and what you found (e.g. "`PaymentProcessor.Charge()` catches `Exception` broadly and swallows it, returning `null`")
   - Flag concrete risks: bare `catch (Exception)` blocks that hide root causes, missing distinction between transient and permanent errors, no circuit breaker around external calls, exceptions used for expected control flow, inconsistent error responses across endpoints
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** error handling code or changes.

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
- Transient vs. permanent error classification (should transient errors retry, should permanent ones fail fast?)
- Circuit breaker and retry policy correctness
- Whether errors are logged with enough context before being handled
- Whether cascading failures are prevented (one service's failure shouldn't take down its callers)
- Exception types used appropriately (not using exceptions for expected/normal outcomes)

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic error-handling advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** error handling patterns.

Provide:
- `Result<T>` pattern implementation for explicit, type-safe error returns
- Circuit breaker and retry policy setup (e.g. with Polly), including how to distinguish transient from permanent failures
- Guidance on where to catch vs. where to let exceptions propagate
- Patterns for mapping internal errors to consistent API error responses

## Validation Context

Use when the user asks to **validate** a story or requirement involving error handling.

Check that requirements address:
- Which failure modes are expected and how each should behave (retry, fail fast, degrade gracefully)
- Whether cascading failure prevention is called out (circuit breakers, timeouts, bulkheads)
- How errors are surfaced to callers/users (consistent error contract)
- Whether the story distinguishes transient (retryable) from permanent (non-retryable) errors

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at logging instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
