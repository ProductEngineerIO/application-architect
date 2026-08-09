---
name: async
description: Evaluate, review, and validate .NET async/await patterns. Use when assessing async implementations, reviewing Task usage, evaluating async method design, or validating architecture stories involving asynchronous operations and concurrent execution.
---

# Async/Await Guidance

You are evaluating or helping with Async/Await patterns in .NET. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with async/await?
> A) Evaluate my existing async implementation
> B) Review a specific code change
> C) Learn/write async patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my async changes" clearly means Review; a bare `/dotnet-async` does not).

## Evaluation Context

Use when the user asks to **evaluate** their async implementation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: I/O-bound calls (HTTP, database, file access), `async`/`await` usage, and blocking patterns like `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`, or `Task.Run` used to fake async over sync code.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual methods/classes and what you found (e.g. "`ReportGenerator.Build()` calls `.Result` on a database query, which can deadlock under load")
   - Flag concrete risks: sync-over-async, missing `ConfigureAwait(false)` in library code, missing `CancellationToken` propagation, `async void` outside event handlers, unnecessary `Task.Run` wrapping already-async work
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment (A/B/C) before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at concurrency instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.

## Review Context

Use when the user asks to **review** async code or changes.

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
- Proper async/await patterns
- Task composition and coordination
- ConfigureAwait usage
- Cancellation token handling
- Deadlock prevention

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic async advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** async patterns.

Provide:
- Async/await examples
- Task composition patterns
- Error handling in async code
- Cancellation token patterns
- ConfigureAwait best practices

## Validation Context

Use when the user asks to **validate** a story or requirement involving async operations.

Check that requirements address:
- Which operations need to be async
- Cancellation handling
- Error propagation
- Performance implications
- Thread safety considerations
