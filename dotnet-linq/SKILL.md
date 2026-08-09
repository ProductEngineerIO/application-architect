---
name: dotnet-linq
description: Evaluate, review, and validate .NET LINQ and data access query patterns. Use when assessing query efficiency, reviewing LINQ code, evaluating N+1 problems or unnecessary materialization, or validating architecture stories involving data access.
---

# LINQ Guidance

You are evaluating or helping with LINQ and data access query patterns in .NET — lazy vs. eager evaluation, avoiding N+1 queries, and composing queries efficiently at the right layer. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with LINQ / data access?
> A) Evaluate my existing query patterns
> B) Review a specific code change
> C) Learn/write query patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my query changes" clearly means Review; a bare `/dotnet-linq` does not).

## Evaluation Context

Use when the user asks to **evaluate** their LINQ/data access patterns.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: EF Core queries missing `.Include()` for related data accessed in a loop (classic N+1), queries materialized early with `.ToList()`/`.ToArray()` then filtered further in memory, missing `.AsNoTracking()` on read-only queries, and whether query composition happens at the repository/data layer or leaks into controllers/services.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual methods/classes and what you found (e.g. "`OrderRepository.GetOrdersWithItems()` loops over orders and accesses `order.Items` without `.Include()` — N+1 query against the database")
   - Flag concrete risks: N+1 patterns, unnecessary full materialization before filtering/paging, missing `AsNoTracking()` on read-only paths, business logic embedded inside query expressions that would be clearer (and testable) outside them
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** LINQ/query code or changes.

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
- Will this query trigger N+1 database round trips when related data is accessed?
- Is the query staying lazy (`IQueryable<T>`) until it actually needs to execute, or materializing too early?
- Are async LINQ methods used against the database (`.ToListAsync()`, not `.ToList()`)?
- Is `.AsNoTracking()` appropriate here for a read-only query?
- Is complex logic embedded in the query expression when it would be clearer (and more testable) as a named, separate method?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic LINQ advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** LINQ/query patterns.

Provide:
- Patterns for eager-loading related data to avoid N+1 (`.Include()`, projection with `.Select()`)
- Guidance on lazy vs. eager evaluation and when each is appropriate
- Async LINQ usage against EF Core (`.ToListAsync()`, `.FirstOrDefaultAsync()`, etc.)
- When to use `.AsNoTracking()` and when tracking is actually needed

## Validation Context

Use when the user asks to **validate** a story or requirement involving data access.

Check that requirements address:
- What related data needs to be loaded together (to catch N+1 risk before implementation)
- Whether the story implies filtering/paging that should happen at the database, not in memory
- Whether read-only vs. read-write access patterns are distinguished

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at performance instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
