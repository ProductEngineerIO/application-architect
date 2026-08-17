---
name: dotnet-data-access
description: Evaluate, review, and validate .NET data access and Entity Framework Core patterns — DbContext lifetime and scoping, repository pattern usage, and migration strategy. Use when assessing data access layer design, reviewing DbContext or migration changes, evaluating repository abstraction choices, or validating architecture stories involving persistence. Distinct from dotnet-linq, which covers query-writing mechanics (N+1, lazy vs. eager evaluation) rather than the surrounding data access architecture.
---

# Data Access & EF Core Patterns

You are evaluating or helping with the data access layer in .NET — `DbContext` lifetime and scoping, whether and how a repository abstraction is used over EF Core, and migration strategy. This is the architecture *around* queries; query-writing mechanics themselves (N+1, lazy vs. eager evaluation, `.AsNoTracking()`) are covered by `dotnet-linq`. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with data access & EF Core patterns?
> A) Evaluate my existing data access layer
> B) Review a specific code change
> C) Learn/write data access patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my DbContext changes" clearly means Review; a bare `/dotnet-data-access` does not).

## Evaluation Context

Use when the user asks to **evaluate** their data access layer.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: `DbContext` lifetime and registration (should almost always be scoped — a singleton `DbContext` is a common and serious bug, especially combined with concurrency, see `dotnet-concurrency`), whether a repository abstraction exists and, if so, whether it's a genuine abstraction (interface owned by the domain, per `dotnet-dip`) or a thin pass-through that just wraps EF Core 1:1 without adding value, migration history for signs of drift (migrations that were edited after being applied, or a schema that looks like it's diverged from what migrations describe), and whether transactions are used appropriately for operations that need atomicity across multiple saves.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual contexts/repositories and what you found (e.g. "`AppDbContext` is registered as a singleton in `Program.cs` — under concurrent requests this will cause serious threading issues since `DbContext` isn't thread-safe")
   - Flag concrete risks: `DbContext` with an inappropriate lifetime, a repository layer that's pure ceremony with no actual abstraction value (worth questioning against `dotnet-yagni`), missing transactional boundaries around multi-step saves that need atomicity, migrations that look manually edited post-application (a common source of environment drift)
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** data access code or changes.

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
- Is `DbContext` lifetime correct for how this code is used — no singleton holding a scoped-lifetime context, no context reused across unrelated requests?
- If a new migration is included, does it match the model changes, and does it look safe to apply against existing data (no silent data loss on a column type change, for instance)?
- Does this change need a transaction (multiple saves that must succeed or fail together), and if so, is one actually used?
- If a repository method is added, does it add real value over calling EF Core directly, or is it ceremony?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic data-access advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** data access code.

Provide:
- Correct `DbContext` lifetime/registration for typical web app scenarios
- When a repository abstraction actually earns its keep (e.g. genuinely swappable persistence, or a domain that shouldn't know about EF Core directly) vs. when it's unnecessary ceremony over EF Core's own abstraction
- Transaction patterns for multi-step operations that need atomicity
- Migration workflow and safe patterns for schema changes against existing data

## Validation Context

Use when the user asks to **validate** a story or requirement involving persistence.

Check that requirements address:
- What data needs to be persisted and whether any multi-step save needs transactional atomicity
- Whether a schema change is implied, and what migration/data-safety considerations that carries
- Whether the domain should be insulated from the specific persistence technology here, or direct EF Core usage is acceptable for this context

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at LINQ instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
