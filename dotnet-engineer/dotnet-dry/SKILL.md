---
name: dotnet-dry
description: Evaluate, review, and validate adherence to the Don't Repeat Yourself (DRY) principle in .NET code. Use when assessing duplicated logic, reviewing copy-pasted code, evaluating whether abstraction would reduce repetition, or validating architecture stories for shared logic scope.
---

# DRY (Don't Repeat Yourself)

You are evaluating or helping with the DRY principle — every piece of knowledge should have a single, unambiguous representation in the system. This is about duplicated *knowledge/logic*, not superficially similar-looking code that represents genuinely different concepts (over-applying DRY to unrelated code is its own anti-pattern, sometimes called premature abstraction). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with DRY (Don't Repeat Yourself)?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for DRY (Don't Repeat Yourself) violations" clearly means Review; a bare `/dotnet-dry` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: near-identical logic blocks copy-pasted across multiple classes/methods, the same business rule (e.g. a discount calculation, a validation rule) implemented independently in more than one place, and — just as important — cases where someone has already "DRY'd" two things that don't actually represent the same knowledge, creating an awkward shared abstraction that now has to branch internally for unrelated cases.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: the same business rule maintained in multiple places (a correctness risk when one gets updated and the other doesn't), over-abstracted "shared" code that's actually forcing unrelated concepts together
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
- Is this duplicated code representing the exact same piece of business knowledge, or does it just look similar by coincidence?
- If the underlying rule changed, would someone have to remember to update it in more than one place?
- Is there an existing "shared" abstraction here that's actually merging two different concepts and branching awkwardly as a result?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to distinguish true duplication (same knowledge, same reason to change) from coincidental similarity (different reasons to change, shouldn't be merged)
- Extraction patterns for consolidating genuine duplication into one source of truth
- When to *stop* applying DRY — the "rule of three" and the risk of premature abstraction

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's new logic duplicates an existing business rule that already exists elsewhere in the system
- Whether the story assumes a shared abstraction that may not actually represent one coherent piece of knowledge

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at KISS instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
