---
name: dotnet-ddd
description: Evaluate, review, and validate Domain-Driven Design (DDD) usage in .NET code — bounded contexts, aggregates, entities vs. value objects, ubiquitous language, and domain events. Use when assessing domain model design, reviewing aggregate boundaries, evaluating whether code reflects the business's own language, or validating architecture stories for domain modeling concerns.
---

# Domain-Driven Design (DDD)

You are evaluating or helping with Domain-Driven Design — modeling software around the business domain itself, using bounded contexts to scope meaning, aggregates to enforce consistency boundaries, entities/value objects to distinguish identity from pure values, ubiquitous language to keep code and business conversation aligned, and domain events to express things that happened in business terms. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with DDD?
> A) Evaluate my existing domain model
> B) Review a specific code change
> C) Learn/write DDD patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my aggregate changes" clearly means Review; a bare `/dotnet-ddd` does not).

## Evaluation Context

Use when the user asks to **evaluate** their domain model or DDD usage.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: how aggregates are defined and whether their consistency boundaries make sense (is an aggregate root actually protecting its invariants, or can internal entities be modified directly from outside?), whether entities (has identity, can change over time) and value objects (no identity, immutable, compared by value) are distinguished or everything is modeled as a mutable entity, whether class/method/variable names match the language the business actually uses (ubiquitous language) or diverge into generic technical terms, presence (or absence) of domain events for things that happened, and whether bounded context boundaries are respected — or whether one model is being stretched to mean different things in different contexts.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/bounded context to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual aggregates/entities/value objects and what you found (e.g. "`Account` aggregate exposes a public setter on `Balance`, so callers can mutate it directly without going through a method that enforces the overdraft invariant")
   - Flag concrete risks: aggregate boundaries that don't actually protect an invariant, anemic domain models (data bags with no behavior, logic living elsewhere instead), value objects modeled as mutable entities, code vocabulary that's drifted from what the business actually calls things, one model being reused across what are really two different bounded contexts
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** domain model code or changes.

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
- Does the aggregate root protect its invariants, or can internal state be modified by bypassing it?
- Is this thing an entity (has identity, mutable over time) or a value object (no identity, should be immutable, equality by value) — and is it modeled accordingly?
- Does the code's vocabulary match the ubiquitous language the business actually uses, or has it drifted into generic technical terms?
- If something notable happened in the domain, is it expressed as a domain event, or silently implied by a state change?
- Does this change respect the current bounded context, or is it importing a concept/model from a different context that means something else there?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic DDD theory.

## Writing/Learning Context

Use when the user asks to **learn** or **write** using DDD patterns.

Provide:
- Aggregate design: how to choose the aggregate root and enforce invariants only through it
- Entity vs. value object modeling — identity and mutability vs. immutability and value equality
- Ubiquitous language practice — deriving names directly from how the business actually talks about the domain, and keeping code and conversation in sync
- Domain event patterns for expressing things that happened, and how they can decouple bounded contexts
- Bounded context boundaries — when a "similar" concept in two contexts is actually two different models that shouldn't be unified

## Validation Context

Use when the user asks to **validate** a story or requirement involving domain modeling.

Check that requirements address:
- Which aggregate(s) are affected and what invariant they need to protect
- Whether new concepts are named using the business's own language, not ad hoc technical terms
- Whether anything described is really an event that other parts of the system might care about
- Whether the story is implicitly crossing a bounded context boundary and needs an explicit translation/integration point instead of a shared model

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at input validation instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
