---
name: dotnet-messaging
description: Evaluate, review, and validate .NET messaging and event-driven architecture — publishing/consuming events, message broker integration (Service Bus, Kafka, MassTransit, etc.), and the outbox pattern. Use when assessing event schema design, reviewing publisher/consumer code, evaluating message reliability guarantees, or validating architecture stories involving asynchronous cross-service communication. Related to but distinct from dotnet-idempotency (message deduplication is one specific application of it) and dotnet-error-handling (retry/circuit-breaker around message processing).
---

# Messaging & Event-Driven Architecture

You are evaluating or helping with messaging and event-driven architecture in .NET — publishing and consuming events/messages across service boundaries via a broker, designing event schemas as a stable contract, and handling the reliability guarantees (at-least-once delivery, ordering, the dual-write problem) that come with asynchronous cross-service communication. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with messaging & event-driven architecture?
> A) Evaluate my existing messaging setup
> B) Review a specific code change
> C) Learn/write messaging patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my event publisher changes" clearly means Review; a bare `/dotnet-messaging` does not).

## Evaluation Context

Use when the user asks to **evaluate** their messaging/event-driven setup.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: event/message schema definitions (are they versioned and treated as a stable contract, or just whatever the current internal model happens to look like — related to `dotnet-api-versioning`'s contract-stability concerns), whether database writes and message publishes that should be atomic together are actually protected against the dual-write problem (an outbox pattern, or something more fragile like "save to DB, then publish, hope both succeed"), consumer-side deduplication for at-least-once delivery (related to `dotnet-idempotency`), and whether consumers assume ordering guarantees the broker doesn't actually provide.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual publishers/consumers/events and what you found (e.g. "`OrderService` saves the order to the database, then publishes `OrderCreatedEvent` in a separate step — if the process crashes between the two, the event is silently lost with no retry")
   - Flag concrete risks: dual-write problems (DB write and publish not atomic), event schemas treated as internal DTOs rather than a stable external contract, missing consumer deduplication under at-least-once delivery, ordering assumptions that the broker/topic configuration doesn't actually guarantee, poison messages with no dead-letter handling
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** messaging code or changes.

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
- If this change writes to a database and publishes an event for the same operation, are they actually atomic (outbox pattern, or a transactional guarantee), or is there a window where one succeeds and the other doesn't?
- Is the event schema being changed here a breaking change for existing consumers, the same way an API contract change would be (see `dotnet-api-versioning`)?
- Does the consumer handle a duplicate delivery of the same message correctly (see `dotnet-idempotency`)?
- Is there handling for a message that repeatedly fails to process (dead-letter, poison message handling), or would it retry forever / get silently dropped?
- Does this consumer assume an ordering guarantee that the topic/partition/queue configuration doesn't actually provide?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic messaging theory.

## Writing/Learning Context

Use when the user asks to **learn** or **write** messaging/event-driven code.

Provide:
- The outbox pattern for atomic "save state + publish event" operations
- Event schema design as a stable, versioned contract (not the internal domain model serialized directly)
- Consumer deduplication patterns for at-least-once delivery
- Dead-letter queue / poison message handling
- When ordering actually matters and how to get it (partition keys, single-consumer-per-partition) vs. when the system should be designed to not need it

## Validation Context

Use when the user asks to **validate** a story or requirement involving events/messaging.

Check that requirements address:
- What event(s) this feature needs to publish or consume, and whether their schema is being treated as a versioned contract
- Whether the "save state and publish" step needs atomicity guarantees (outbox) or can tolerate eventual consistency
- What happens if this message is delivered twice, out of order, or fails processing repeatedly

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at idempotency instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
