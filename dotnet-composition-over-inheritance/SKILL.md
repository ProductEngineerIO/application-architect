---
name: dotnet-composition-over-inheritance
description: Evaluate, review, and validate the use of composition vs. inheritance in .NET code. Use when assessing deep or fragile inheritance hierarchies, reviewing base class design, evaluating whether a has-a relationship is modeled as is-a, or validating architecture stories involving object relationships.
---

# Composition over Inheritance

You are evaluating or helping with the composition-over-inheritance principle — prefer composing behavior from smaller collaborating objects over deep inheritance hierarchies, which tend to become fragile as requirements evolve (the fragile base class problem). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Composition over Inheritance?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Composition over Inheritance violations" clearly means Review; a bare `/dotnet-composition-over-inheritance` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: inheritance hierarchies more than two levels deep, base classes that have grown many virtual/overridable members to accommodate different subclasses' needs, subclasses that override methods just to disable or bypass inherited behavior (an LSP-adjacent smell), and relationships that are really "has-a"/"uses-a" (better modeled by holding a reference to a collaborator) but are implemented as "is-a" inheritance instead.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: fragile base class problems where changing the base affects distant subclasses unpredictably, inheritance used for code reuse rather than genuine polymorphic substitutability
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
- Is this relationship genuinely "is-a" (subtype should be substitutable — see LSP), or is it really "has-a"/"uses-a" being forced into inheritance for convenience?
- If the base class changed, how confident are you every subclass would still behave correctly?
- Would extracting this shared behavior into a collaborator/service the class holds a reference to be simpler than inheriting it?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to recognize when inheritance is genuinely appropriate (true is-a, stable contract) vs. when composition would be more flexible
- Patterns for refactoring a deep/fragile hierarchy toward composition (extracting behavior into injected collaborators)
- How this relates to LSP — composition sidesteps LSP risk entirely by not claiming a substitutability relationship

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's proposed object relationships are genuinely "is-a" or would be better modeled as composed collaborators
- Whether the story risks extending an existing inheritance hierarchy in a way that could destabilize other subclasses

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at Liskov substitution instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
