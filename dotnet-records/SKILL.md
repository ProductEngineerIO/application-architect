---
name: dotnet-records
description: Evaluate, review, and validate .NET record and immutability usage. Use when assessing whether data is safely immutable for concurrent/async access, reviewing record vs. class choices, evaluating thread-safety of shared data, or validating architecture stories involving concurrent data access.
---

# Records & Immutability Guidance

You are evaluating or helping with immutable data patterns in .NET — using `record` types and immutability to prevent race conditions when data is shared across concurrent async operations. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with records/immutability?
> A) Evaluate my existing usage of records and immutability
> B) Review a specific code change
> C) Learn/write immutable data patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my records changes" clearly means Review; a bare `/dotnet-records` does not).

## Evaluation Context

Use when the user asks to **evaluate** their use of records and immutability.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: DTOs/domain value objects defined as mutable classes vs. `record`/`record struct`, mutable collections or public setters on data shared across async boundaries, domain value objects like `Money`, `Status`, `Email` that should be immutable but aren't, and any manual locking/defensive-copying that suggests mutable shared state is causing pain.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes and what you found (e.g. "`OrderDto` is a mutable class with public setters, passed into `Task.WhenAll` calls where multiple tasks could mutate it concurrently")
   - Flag concrete risks: mutable DTOs crossing async/thread boundaries, missing `init`-only or `record` usage for data that's conceptually a value, mutable static/shared state, defensive copying used as a workaround instead of just making the type immutable
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** record/immutability code or changes.

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
- Is this data conceptually a value (DTO, domain value object) that should be a `record` rather than a mutable class?
- Does the type cross an async or concurrency boundary while still being mutable?
- Are collections exposed as mutable (`List<T>`) when they should be read-only (`IReadOnlyList<T>`, `ImmutableArray<T>`)?
- Is `with`-expression usage (non-destructive mutation) used appropriately instead of manual copy-and-modify code?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic immutability advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** immutable data patterns.

Provide:
- `record`/`record struct` definitions for DTOs and domain value objects
- `with`-expression usage for non-destructive updates
- Guidance on read-only collection types for exposing collections safely
- Patterns for making previously-mutable shared state immutable without a large rewrite

## Validation Context

Use when the user asks to **validate** a story or requirement involving concurrent data access.

Check that requirements address:
- Which data is shared across concurrent/async operations and whether it needs to be immutable
- Whether domain value objects are called out as immutable by design
- Whether the story risks introducing mutable shared state without noticing (e.g. a cache, a shared result object)

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at testing instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
