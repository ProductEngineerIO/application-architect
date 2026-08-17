---
name: dotnet-coupling-cohesion
description: Evaluate, review, and validate coupling and cohesion in .NET code. Use when assessing how tightly modules depend on each other, reviewing whether related logic is grouped together, evaluating change-impact ripple effects, or validating architecture stories for module boundaries.
---

# Coupling & Cohesion

You are evaluating or helping with coupling (how much modules depend on each other's internals) and cohesion (how tightly related the responsibilities within one module are) — the two properties that together determine how easy a codebase is to change safely. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Coupling & Cohesion?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Coupling & Cohesion violations" clearly means Review; a bare `/dotnet-coupling-cohesion` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: classes that reach into other classes' internal state or implementation details rather than their public contract, changes that would ripple across many unrelated files (a sign of tight coupling), classes whose methods use wildly different subsets of the class's own fields (a sign of low cohesion — the class is really two classes), and modules connected through many small dependencies rather than one clean interface.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: tight coupling that makes isolated changes risky, low-cohesion classes bundling unrelated responsibilities, chatty inter-module communication that suggests a missing abstraction boundary
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
- If this class/module changed, how many other files would likely need to change too? Is that ripple justified by genuine relatedness?
- Do this class's methods form one cohesive theme, or do they cluster into groups that barely touch each other's data?
- Is this dependency going through a clean interface, or reaching into another module's internals?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to recognize high coupling (change ripple, reaching into internals) vs. low coupling (interacting only through a stable public contract)
- How to recognize high cohesion (everything in the class serves one purpose) vs. low cohesion (unrelated responsibilities bundled together)
- Refactoring patterns: extracting a class to raise cohesion, introducing an interface/facade to reduce coupling

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's design introduces tight coupling between modules that should be able to evolve independently
- Whether the responsibilities the story describes for one component actually belong together (cohesion check) or are being bundled for convenience

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at separation of concerns instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
