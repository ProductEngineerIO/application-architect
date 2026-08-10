---
name: dotnet-code-smells
description: Evaluate, review, and validate code for common code smells and refactoring opportunities in .NET. Use when assessing overall code health, reviewing code for smells like long methods or feature envy, evaluating refactoring priorities, or validating architecture stories for maintainability risk.
---

# Code Smells & Refactoring

You are evaluating or helping with code smells — surface-level indicators (long methods, large classes, feature envy, primitive obsession, shotgun surgery, data clumps) that often point to a deeper design problem covered by one of the other principle skills. Use this skill as a scanning/triage pass, then point to the more specific principle skill (SRP, coupling/cohesion, etc.) for a deeper dive. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Code Smells & Refactoring?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Code Smells & Refactoring violations" clearly means Review; a bare `/dotnet-code-smells` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: long methods that do many things sequentially (candidates for extraction), large classes with many fields/methods, "feature envy" methods that use another class's data more than their own, primitive obsession (using raw strings/ints for concepts like email addresses or money that deserve their own type), data clumps (the same group of parameters passed together everywhere, suggesting a missing class), and shotgun surgery (a single logical change requiring edits across many unrelated files).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: smells that correlate strongly with a deeper principle violation — note which specific principle (SRP, DRY, coupling/cohesion, etc.) each finding maps to so the user can dig deeper with that skill if they want
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
- Are there long methods or large classes that would benefit from extraction?
- Is there feature envy — a method more interested in another class's data than its own?
- Is there primitive obsession where a real domain concept is represented as a raw string/int/bool?
- Are there data clumps — the same group of values always passed together, suggesting a missing type?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to recognize each classic smell (long method, large class, feature envy, primitive obsession, data clumps, shotgun surgery) at a glance
- Standard refactoring moves for each smell (extract method/class, introduce parameter object, replace primitive with domain type)
- How smells map to the deeper principles (e.g. feature envy often signals a cohesion/coupling problem; shotgun surgery often signals a DRY or SRP problem) so the user knows which skill to reach for next

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's design is likely to introduce smells that are cheap to avoid now but expensive to refactor later (e.g. planning a data clump that could be a value object from the start)
- Whether existing smells nearby will make implementing this story harder than it needs to be, worth flagging before starting

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at cyclomatic complexity instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
