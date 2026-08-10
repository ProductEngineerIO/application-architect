---
name: dotnet-performance
description: Evaluate, review, and validate .NET performance and memory allocation patterns — Span<T>/Memory<T> usage, object pooling, GC-awareness, and benchmarking discipline. Use when assessing hot-path allocation pressure, reviewing performance-sensitive code, evaluating whether a performance claim is actually measured, or validating architecture stories with explicit performance requirements.
---

# Performance & Memory

You are evaluating or helping with performance and memory allocation in .NET — reducing unnecessary allocations on hot paths (`Span<T>`/`Memory<T>`, object pooling), understanding how allocation pressure affects GC behavior, and insisting that performance claims and "optimizations" are actually measured with a benchmark rather than assumed. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with performance & memory?
> A) Evaluate my existing code for performance/allocation concerns
> B) Review a specific code change
> C) Learn/write performance-conscious patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review this hot path for allocations" clearly means Review; a bare `/dotnet-performance` does not).

## Evaluation Context

Use when the user asks to **evaluate** performance/memory characteristics of their code.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: hot paths (called frequently — request handling, tight loops, high-throughput message processing) that allocate heavily where they don't need to (string concatenation in a loop, unnecessary intermediate collections, boxing of value types, LINQ chains allocating enumerators on a very hot path), whether `Span<T>`/`Memory<T>` would avoid an allocation where a slice/view of existing memory would do, and any object pooling in use (or conspicuously absent for expensive-to-construct objects reused frequently). Also look for whether any existing "performance optimization" in the code is backed by an actual benchmark, or is just an assumption.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual hot paths and what you found (e.g. "`RequestParser.Parse()` runs on every incoming request and builds a new `List<string>` via LINQ `.Split().Select().ToList()` — for a hot path this is worth measuring against a `Span<T>`-based parser")
   - Flag concrete risks: allocation-heavy code on genuinely hot paths (not paths that don't matter), unmeasured "optimizations" that may not actually help, missing pooling for expensive/frequently-reused objects, obvious but low-risk wins (e.g. `StringBuilder` instead of repeated string concatenation in a loop)
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps — and be explicit that anything nontrivial should be validated with a benchmark before and after, not merely assumed to help

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** performance-sensitive code or changes.

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
- Is this actually a hot path where allocation reduction matters, or is the change optimizing something that runs rarely (in which case the added complexity of `Span<T>`/pooling probably isn't worth it — this is a KISS/YAGNI consideration as much as a performance one)?
- Does this change come with a benchmark showing the claimed improvement, or is the performance benefit asserted without measurement?
- If pooling is introduced, are pooled objects correctly reset/returned, avoiding subtle bugs from stale state leaking between uses?
- Does the change trade away readability for a performance gain that hasn't been shown to matter?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic performance advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** performance-conscious code.

Provide:
- `Span<T>`/`Memory<T>` patterns for avoiding unnecessary allocations when working with slices of existing data
- Object pooling patterns (`ObjectPool<T>`) for expensive-to-construct, frequently-reused objects
- How to actually benchmark a change (BenchmarkDotNet or equivalent) rather than assuming an optimization helped
- How to identify whether a piece of code is actually hot enough to be worth this kind of attention in the first place

## Validation Context

Use when the user asks to **validate** a story or requirement with explicit performance requirements.

Check that requirements address:
- What the actual performance target is (a number, not "make it fast") and how it will be measured
- Whether the code path in question is genuinely on the critical/hot path, or the requirement is targeting the wrong place
- Whether achieving the target might trade off readability/maintainability, and whether that trade-off is acceptable here

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at caching instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
