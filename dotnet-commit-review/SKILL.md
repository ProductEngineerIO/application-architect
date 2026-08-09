---
name: dotnet-commit-review
description: Perform a comprehensive technical review of a git commit or diff by identifying which architectural areas it touches across the full principle/practice catalog (SOLID, DRY/KISS/YAGNI, coupling & cohesion, DDD, Dependency Injection, Configuration, Async/Await, Error Handling, Logging, Concurrency, Testing, Records, Tracing/Metrics, Nullable Reference Types, LINQ, Minimal APIs) and reviewing each relevant area for adherence or deviation from best practices. Use when the user wants a full review of a commit or diff — especially when a change might span multiple concerns at once (e.g. Dependency Injection and Concurrency together) — rather than a single-topic review through just one lens.
---

# Commit Review (Multi-Lens)

You are performing a comprehensive review of a single commit or diff by figuring out which architectural areas it touches, then reviewing it through each relevant lens — and, critically, calling out issues that only emerge from the *combination* of areas touched together, which no single-topic review would catch.

This is a workflow skill, not a topic skill — it doesn't have Evaluate/Review/Write/Validate contexts. It always does one thing: take a diff and produce a routed, multi-lens review of it.

## Step 1: Get the diff

If it isn't already clear (e.g. the user already pasted code, or already named a commit/hash), ask:

> How would you like to give me the commit to review?
> A) Paste the diff directly
> B) Pick from your last 10 commits
> C) Give me a commit hash or range

- If **B**: run exactly `git log -10 --oneline` (add `-- <path>` only if the user already narrowed it to a project/service). Do not run any other git commands yet — just show the resulting list with numbers and stop to let them pick. Only after they pick a specific commit, run `git show <hash>` or `git diff <hash>^ <hash>` to get that one diff.
- If **C**: take the hash/range they give and run `git show <hash>` (single commit) or `git diff <hash1> <hash2>` (range) to get the diff.
- If **A**: wait for them to paste it.
- If there's no git repo available (e.g. `git log` fails) or they're not in a repo context, skip straight to asking them to paste the diff.

## Step 2: Identify which areas the diff touches

Scan the diff against the full catalog below. Be thorough — check Tier 0 principles as carefully as Tier 1-4 practices; principle violations are often the least obvious to a self-review but are exactly what this workflow is for.

**Tier 0 — SOLID:** Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
**Tier 0 — Other Core Principles:** DRY, KISS, YAGNI
**Tier 0 — Structural Principles:** Coupling & Cohesion, Separation of Concerns, Composition over Inheritance
**Tier 0 — Code-Level Quality:** Naming & Readability, Code Smells & Refactoring, Cyclomatic Complexity
**Tier 0 — Paradigms:** Domain-Driven Design
**Tier 1 — Foundational:** Dependency Injection, Configuration & Options, Async/Await
**Tier 2 — Critical:** Error Handling & Result Types, Structured Logging, Concurrency & Thread Safety
**Tier 3 — Important:** Testing, Records & Immutability, Distributed Tracing & Metrics
**Tier 4 — Quality:** Nullable Reference Types, LINQ, Minimal APIs

List the areas you believe are implicated, briefly, before doing the deep review — this is your working list, and the user should see it too so they can correct you if you missed something or flagged something irrelevant.

## Step 3: Call out cross-cutting changes explicitly

If more than one area is implicated, say so clearly and *before* the per-area findings — e.g. "This commit touches both Dependency Injection and Concurrency: a new singleton registration is being introduced, and singletons carrying mutable state is exactly where DI and concurrency concerns collide." Don't bury this observation inside the per-area sections; it's the reason this workflow exists instead of just running one topic skill.

If only one area is implicated, say so too, briefly, and note the user could get equivalent depth from that single topic skill directly next time — then proceed with the review anyway.

## Step 4: Review each implicated area

For each area identified, review the diff through that specific lens — adherence or deviation from that area's known best practices, with specific findings tied to actual lines/files in the diff, not generic principle-recitation. Ground each finding in what best practice guidance for that area would flag (e.g. for Concurrency: unprotected shared mutable state, locking around `await`; for DI: constructor over-injection, incorrect lifetime; for SRP: multiple unrelated reasons to change).

Keep each area's findings grouped and labeled so the user can see clearly which finding belongs to which concern.

## Step 5: Intersection findings (the most valuable part)

This is not just concatenating the per-area findings — actively look for problems that **only exist because two or more areas interact** in this specific diff. Examples of the kind of thing to look for:
- A new DI registration that introduces shared mutable state without concurrency protection (DI × Concurrency)
- A new domain invariant added without a corresponding test for the failure case (DDD × Testing)
- An error-handling change that swallows an exception which async cancellation now depends on (Error Handling × Async)
- A new abstraction introduced to satisfy OCP that also violates YAGNI because nothing else implements it yet (OCP × YAGNI)

If you don't find any genuine intersection issues, say so directly rather than manufacturing a weak one — false positives here undermine the value of this step.

## Step 6: Summarize

Close with:
- A short prioritized list (1-3 items) of what's most worth addressing, spanning both single-area and intersection findings
- Which items are blocking-quality concerns vs. minor/stylistic

## Navigation

At any natural pause point, offer the user a way to move on. They can say:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "dig into [area]" — go deeper into one specific implicated area using that topic skill's own contexts (Evaluate/Review/Write/Validate)
- "done" / "that's all" — ends here

If the user says any of these, act on it immediately rather than continuing to add more analysis unprompted.
