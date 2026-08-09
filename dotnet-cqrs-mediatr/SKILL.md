---
name: dotnet-cqrs-mediatr
description: Evaluate, review, and validate CQRS (Command Query Responsibility Segregation) and MediatR pipeline usage in .NET. Use when assessing command/query separation, reviewing handler design, evaluating pipeline behaviors (validation, logging, transactions), or validating architecture stories involving request/response flow through MediatR.
---

# CQRS & MediatR Patterns

You are evaluating or helping with CQRS (separating commands that change state from queries that read it) and MediatR-based implementation of that separation in .NET — commands/queries as explicit request objects, handlers as the single place each is processed, and pipeline behaviors for cross-cutting concerns around that processing. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with CQRS/MediatR?
> A) Evaluate my existing command/query and handler design
> B) Review a specific code change
> C) Learn/write CQRS/MediatR patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my handler changes" clearly means Review; a bare `/dotnet-cqrs-mediatr` does not).

## Evaluation Context

Use when the user asks to **evaluate** their CQRS/MediatR usage.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: whether commands (state-changing) and queries (read-only) are actually kept separate as distinct request types, or mixed/blurred (a "command" that also returns a large read model, or a "query" with side effects), handler size and focus (a handler doing far more than orchestrating one use case), what pipeline behaviors exist (validation, logging, transactions, authorization) and whether cross-cutting concerns are handled there consistently or duplicated inside individual handlers, and whether handlers call other handlers directly (bypassing the mediator, which usually signals a missing shared service rather than genuine command/query composition).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual commands/queries/handlers and what you found (e.g. "`UpdateAccountCommand` handler also returns a fully populated `AccountDto` with related data — blurring command and query responsibilities")
   - Flag concrete risks: commands with query-like read/return responsibilities (or vice versa), fat handlers doing validation/logging/persistence all inline instead of through pipeline behaviors, missing transactional boundaries around commands that touch multiple aggregates, handlers invoking other handlers directly instead of going through the mediator or a shared service
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** CQRS/MediatR code or changes.

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
- Is this genuinely a command (changes state, minimal/no return payload) or a query (reads state, no side effects) — and does the code actually honor that distinction?
- Is the handler focused on orchestrating one use case, or has business logic that belongs in the domain leaked into it?
- Are cross-cutting concerns (validation, logging, transactions, authorization) handled via pipeline behaviors, or duplicated inline in this handler?
- Is a new/changed command wrapped in an appropriate transactional boundary if it touches more than one aggregate?
- Does this handler dispatch to the mediator for related operations, or reach around it?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic CQRS theory.

## Writing/Learning Context

Use when the user asks to **learn** or **write** CQRS/MediatR patterns.

Provide:
- Command and query object design — keeping them as thin, explicit intent-carrying requests
- Handler design focused on orchestration, delegating actual business rules to the domain model
- Pipeline behavior patterns for validation, logging, transactions, and authorization as cross-cutting concerns applied consistently across handlers
- Guidance on when CQRS is worth the structure it adds vs. when a simpler service-method approach is more appropriate (ties back to `dotnet-yagni` — don't introduce this pattern speculatively)

## Validation Context

Use when the user asks to **validate** a story or requirement involving command/query flow.

Check that requirements address:
- Whether the new functionality is a command, a query, or genuinely needs both
- What cross-cutting concerns (validation, authorization, transactions) apply and whether they're expected to go through existing pipeline behaviors
- Whether the command's transactional boundary is clear if multiple aggregates are involved

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at input validation instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
