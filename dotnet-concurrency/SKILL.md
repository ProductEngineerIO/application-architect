---
name: dotnet-concurrency
description: Evaluate, review, and validate .NET concurrency and thread-safety patterns — shared mutable state, locking, and coordination primitives. Use when assessing race conditions, reviewing lock usage, evaluating thread-safe collection choices, or validating architecture stories involving concurrent access to shared state. Distinct from the dotnet-async skill, which covers async/await mechanics rather than shared-state safety under concurrent access.
---

# Concurrency & Thread Safety

You are evaluating or helping with concurrency and thread safety in .NET — protecting shared mutable state from race conditions when multiple threads or concurrent async operations access it at the same time, using the right coordination primitive (`lock`, `SemaphoreSlim`, `Interlocked`, `Channel<T>`, concurrent collections) for the situation. This is distinct from async/await mechanics (covered by `dotnet-async`) — a method can be perfectly correct async code and still have a race condition if it touches shared state without protection. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with concurrency & thread safety?
> A) Evaluate my existing code for race conditions
> B) Review a specific code change
> C) Learn/write thread-safe patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my locking changes" clearly means Review; a bare `/dotnet-concurrency` does not).

## Evaluation Context

Use when the user asks to **evaluate** their concurrency/thread-safety approach.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: mutable static or instance fields accessed from multiple threads/concurrent requests (e.g. a singleton service holding mutable state, a static cache/dictionary), non-thread-safe collections (`List<T>`, `Dictionary<TKey,TValue>`) used from concurrent contexts instead of `ConcurrentDictionary<T>`/`ConcurrentBag<T>`/etc., `lock` statements around async code (a deadlock risk — you can't `await` inside a `lock`), and read-modify-write sequences on shared state (e.g. `counter++`, or check-then-act patterns) that aren't atomic.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/fields and what you found (e.g. "`RequestCounter` is a singleton with a plain `int _count` incremented via `_count++` in `Handle()` — not atomic, will lose increments under concurrent load")
   - Flag concrete risks: unprotected shared mutable state, non-thread-safe collections used concurrently, `lock` around `await`-containing code (deadlock risk), check-then-act race conditions (e.g. "if not in cache, add to cache" without atomicity), overly broad locks causing contention/serialization that defeats the purpose of concurrency
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** concurrency-related code or changes.

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
- Is shared mutable state actually protected, or does it look protected but has a gap (e.g. one access path uses the lock, another doesn't)?
- Is the coordination primitive appropriate for the situation — `SemaphoreSlim` for async-compatible mutual exclusion, `lock` only for short synchronous critical sections, `Interlocked` for simple counters, a concurrent collection instead of manual locking around a plain one?
- Is there a `lock` wrapping anything that awaits — a deadlock risk?
- Is the locked region as small as it can be, or is it serializing more work than necessary and defeating the point of concurrency?
- Are check-then-act sequences (check a condition, then act on it) actually atomic, or is there a window for another thread to invalidate the check?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic concurrency advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** thread-safe code.

Provide:
- Choosing the right primitive: `lock` for short sync sections, `SemaphoreSlim` when async-compatible mutual exclusion is needed, `Interlocked` for simple atomic counters, `Channel<T>` for producer/consumer coordination
- Thread-safe collection types (`ConcurrentDictionary<T>`, `ConcurrentQueue<T>`, etc.) and when they're preferable to manual locking around a plain collection
- Why immutability (see `dotnet-records`) sidesteps most concurrency problems entirely — shared immutable data needs no locking
- Common pitfalls: locking around `await`, double-checked locking done incorrectly, over-broad locks that serialize unrelated work

## Validation Context

Use when the user asks to **validate** a story or requirement involving concurrent access.

Check that requirements address:
- What state (if any) will be shared across concurrent requests/threads, and who owns protecting it
- Whether the feature introduces a new singleton or cached/shared object that will need thread-safety consideration
- Whether the expected concurrency level is known (low-contention vs. high-throughput) — this changes which primitive is appropriate
- Whether immutability could sidestep the concurrency concern entirely rather than requiring locking

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at async instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
