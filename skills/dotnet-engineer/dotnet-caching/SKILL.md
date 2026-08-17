---
name: dotnet-caching
description: Evaluate, review, and validate .NET caching implementations — in-memory vs. distributed caching, invalidation strategy, and the cache-aside pattern. Use when assessing cache usage, reviewing cache read/write/invalidation logic, evaluating staleness risk, or validating architecture stories involving cached data.
---

# Caching

You are evaluating or helping with caching in .NET — deciding between in-memory (`IMemoryCache`) and distributed (`IDistributedCache`, Redis, etc.) caching, implementing the cache-aside pattern correctly, and — the part that causes the most real-world bugs — making sure cached data actually gets invalidated when the underlying data changes, rather than silently going stale. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with caching?
> A) Evaluate my existing caching implementation
> B) Review a specific code change
> C) Learn/write caching patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my cache invalidation changes" clearly means Review; a bare `/dotnet-caching` does not).

## Evaluation Context

Use when the user asks to **evaluate** their caching implementation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: what's cached and whether `IMemoryCache` (per-instance, lost across restarts/scale-out) or a distributed cache is used, and whether that choice fits how the service is actually deployed (multiple instances behind a load balancer need a distributed cache or they'll each cache independently and inconsistently); whether cache entries have an expiration set at all, or are cached indefinitely; and — most importantly — whether there's any invalidation logic at all for cached data that changes (a write path that updates the database but never touches the cache is a very common and very real staleness bug).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual cache usages and what you found (e.g. "`ProductService.UpdatePrice()` writes to the database but never invalidates or updates the cached `Product` entry — stale prices will be served until the cache entry naturally expires")
   - Flag concrete risks: no invalidation path when underlying data changes, `IMemoryCache` used in a horizontally-scaled service where a distributed cache is actually needed, no expiration set (unbounded growth / permanent staleness), cache keys that could collide across different logical data, thundering-herd risk on cache miss for hot keys
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** caching code or changes.

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
- If this change writes/updates data that's cached elsewhere, does it also invalidate or update that cache entry — or will the cache now serve stale data?
- Is `IMemoryCache` appropriate here, or does this run in a multi-instance deployment where a distributed cache is actually needed for consistency?
- Is an expiration set, and is it a sensible value for how fresh this data actually needs to be?
- Is the cache key specific enough to avoid unintentional collisions with unrelated cached data?
- On a cache miss for a likely-hot key, is there any protection against many concurrent requests all hitting the underlying source simultaneously (thundering herd)?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic caching advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** caching code.

Provide:
- Cache-aside pattern implementation (check cache, miss → load from source → populate cache → return)
- Choosing `IMemoryCache` vs. a distributed cache based on deployment topology
- Invalidation strategies — explicit invalidation on write, short TTLs as a safety net, or a combination
- Mitigating thundering herd on cache miss for hot keys (locking around the fill, or serving stale-while-revalidate)

## Validation Context

Use when the user asks to **validate** a story or requirement involving cached data.

Check that requirements address:
- Whether the data involved needs to be cached at all, and how fresh it needs to be
- What invalidates the cache when the underlying data changes — this needs to be explicit, not assumed
- Whether the caching approach fits the actual deployment topology (single instance vs. horizontally scaled)

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at performance instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
