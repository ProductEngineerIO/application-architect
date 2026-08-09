---
name: dotnet-idempotency
description: Evaluate, review, and validate idempotency of state-changing operations — APIs, message handlers, and retried operations that should have the same effect whether invoked once or multiple times. Use when assessing safety of retries or duplicate invocations, reviewing operations that could run more than once (network retries, at-least-once message delivery, duplicate user actions), evaluating idempotency key usage, or validating architecture stories involving repeatable operations. General principle, not .NET-specific, though examples and review use .NET code.
---

# Idempotency

You are evaluating or helping with idempotency — the property that performing an operation more than once has the same effect as performing it exactly once. This matters anywhere an operation might genuinely run more than once: client retries after a timeout, at-least-once message delivery, a user double-clicking submit, or automatic retry logic (see `dotnet-error-handling`) re-attempting a call that actually succeeded before the response was lost. This is a general system-design principle, not specific to .NET, though this skill reviews and discusses it in .NET code. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with idempotency?
> A) Evaluate my existing operations for idempotency
> B) Review a specific code change
> C) Learn/write idempotent patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my payment endpoint for idempotency" clearly means Review; a bare `/dotnet-idempotency` does not).

## Evaluation Context

Use when the user asks to **evaluate** idempotency of their operations.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: state-changing operations that could plausibly be retried or duplicated — payment/order creation endpoints, message/event handlers, anything wrapped in retry logic (cross-check against `dotnet-error-handling`'s circuit-breaker/retry usage) — and whether they're naturally idempotent (e.g. "set balance to X") vs. inherently not (e.g. "increment balance by X", "append to list", "send email") without protection. Look for idempotency key support on creation endpoints, deduplication logic in message handlers (checking a processed-message ID before acting), and unique constraints used to prevent duplicate records from a repeated request.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual endpoints/handlers and what you found (e.g. "`POST /orders` has no idempotency key support — a client retry after a timeout will create a duplicate order")
   - Flag concrete risks: non-idempotent operations reachable by retry logic, message handlers with no deduplication under at-least-once delivery, "increment"-style operations where a "set"-style or key-guarded design would be safer, missing unique constraints that would otherwise catch a duplicate at the database level
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** code or changes for idempotency.

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
- If this operation runs twice (client retry, duplicate message, double-click), does it produce the same end state, or a duplicated/incorrect one?
- Is there an idempotency key, natural key, or unique constraint that would catch a repeat, or is nothing guarding against it?
- If this code is reachable by retry logic elsewhere (check for `dotnet-error-handling` patterns nearby), does the retry assume idempotency that doesn't actually hold?
- For message handlers: is there deduplication based on message/event ID, appropriate for an at-least-once delivery guarantee?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic idempotency theory.

## Writing/Learning Context

Use when the user asks to **learn** or **write** idempotent operations.

Provide:
- Idempotency key patterns for creation endpoints (client-supplied key, server checks/stores it, returns the original result on repeat)
- "Set" vs. "increment" framing — designing operations to converge to the same state rather than accumulate effect
- Deduplication patterns for message handlers under at-least-once delivery (tracking processed message IDs, using natural idempotency in the handled effect itself)
- Database-level guarantees (unique constraints, upserts) as a defense-in-depth layer alongside application-level idempotency handling

## Validation Context

Use when the user asks to **validate** a story or requirement involving repeatable/retried operations.

Check that requirements address:
- What should happen if this operation is invoked twice — by a network retry, a duplicate message, or a user double-click
- Whether an idempotency key or equivalent mechanism is expected as part of the design, not left as an afterthought
- Whether this operation is likely to sit behind retry logic (see `dotnet-error-handling`) and, if so, whether idempotency is a precondition for that retry being safe

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at error handling instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
