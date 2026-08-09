---
name: dotnet-api-versioning
description: Evaluate, review, and validate .NET API versioning strategy and contract design. Use when assessing backward compatibility of API changes, reviewing breaking vs. non-breaking changes, evaluating versioning scheme consistency, or validating architecture stories involving public API contracts.
---

# API Versioning & Contract Design

You are evaluating or helping with API versioning and contract design in .NET — how a service's public contract evolves over time without breaking existing consumers, what counts as a breaking vs. non-breaking change, and how versioning is actually exposed (URL segment, header, query string, media type). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with API versioning & contract design?
> A) Evaluate my existing versioning strategy and contracts
> B) Review a specific code change
> C) Learn/write versioning and contract patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my API changes for breaking compatibility" clearly means Review; a bare `/dotnet-api-versioning` does not).

## Evaluation Context

Use when the user asks to **evaluate** their API versioning strategy and contract design.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: whether a versioning scheme is used consistently (URL segment like `/v1/`, header, query string) or applied inconsistently/not at all across endpoints, whether request/response DTOs are distinct per version or the same type is reused and quietly changed underneath consumers, presence of deprecated-but-still-live endpoints and whether they're actually marked/communicated as deprecated, and whether contract changes in recent history look like they broke compatibility (removed/renamed fields, changed types, changed required-ness) without a version bump.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual endpoints/DTOs and what you found (e.g. "`GET /orders/{id}` response DTO had a required field removed in a recent change with no version bump — existing consumers deserializing this will break")
   - Flag concrete risks: breaking changes shipped without a version increment, inconsistent versioning scheme across the API surface, no deprecation signal on endpoints being phased out, domain entities exposed directly as the contract instead of a stable DTO that can evolve independently
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** API contract code or changes.

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
- Is this change actually breaking for existing consumers (removed/renamed field, changed type, newly required field, changed status code/error shape), even if it doesn't look breaking at a glance?
- If it is breaking, does it come with an appropriate version increment, or is it silently changing the existing contract?
- Is the DTO used here stable and intentional, or is a domain entity being exposed directly (so unrelated domain changes will ripple into the contract unexpectedly)?
- If an endpoint is being deprecated, is that actually communicated (deprecation header, documentation, sunset plan), not just informally understood?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic API design advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** versioned API contracts.

Provide:
- Versioning scheme options (URL segment, header, query string, media type) and trade-offs between them
- What counts as breaking vs. non-breaking (adding an optional field is usually safe; removing, renaming, retyping, or newly requiring a field usually isn't)
- DTO design that keeps the public contract independent of internal domain model changes
- Deprecation patterns — signaling an endpoint is going away with enough lead time and visibility for consumers

## Validation Context

Use when the user asks to **validate** a story or requirement involving API contract changes.

Check that requirements address:
- Whether the proposed change is breaking, and if so, what version increment and consumer communication it implies
- Whether the contract (request/response shape) is specified explicitly, not left to be whatever the implementation happens to produce
- Whether existing consumers of the endpoint are accounted for if there's a transition period

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at messaging instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
