---
name: dotnet-dip
description: Evaluate, review, and validate adherence to the Dependency Inversion Principle (DIP) in .NET code. Use when assessing whether high-level modules depend on abstractions rather than concrete implementations, reviewing dependency direction, evaluating tight coupling to concrete types, or validating architecture stories involving module dependencies. Distinct from the dotnet-di skill, which covers the DI container mechanics rather than this design principle.
---

# Dependency Inversion Principle

You are evaluating or helping with the Dependency Inversion Principle (DIP) — high-level modules shouldn't depend on low-level modules; both should depend on abstractions, and abstractions shouldn't depend on details. This is the design principle DI containers exist to support — evaluate the dependency *direction*, not the container mechanics (that's the separate dotnet-di skill). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Dependency Inversion Principle?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Dependency Inversion Principle violations" clearly means Review; a bare `/dotnet-dip` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: high-level business/domain classes that directly reference concrete infrastructure types (e.g. a domain service `new`-ing up a concrete `SqlConnection` or HTTP client instead of depending on an interface), interfaces that live next to their single concrete implementation in the infrastructure layer rather than being owned by the high-level module that needs them, and any place a policy/business-rule class depends on a low-level detail class rather than the reverse.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: high-level modules coupled directly to low-level implementation details, missing abstraction layer between business logic and infrastructure concerns
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** code or changes against this principle.

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
- Does this high-level/business code depend on a concrete low-level type, or on an abstraction?
- Who owns the interface here — the high-level module that needs it, or the low-level module that implements it? (It should usually be the former.)
- Would swapping the concrete implementation require touching the high-level code?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to define abstractions owned by the high-level module, implemented by the low-level module
- The difference between DIP (the design principle) and DI (the container mechanism) — DIP is why DI containers are useful, not the same thing
- Patterns for introducing an abstraction layer over an existing direct dependency on infrastructure

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's design has business logic depending on a concrete infrastructure choice that might change later
- Whether the abstraction boundary between policy and detail is defined before implementation starts

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at dependency injection instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
