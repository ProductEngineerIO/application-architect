---
name: dotnet-naming
description: Evaluate, review, and validate naming and readability in .NET code. Use when assessing whether identifiers clearly communicate intent, reviewing naming choices, evaluating self-documenting code, or validating architecture stories for clear, unambiguous terminology.
---

# Naming & Readability

You are evaluating or helping with naming and readability — names should clearly communicate intent so the code is understandable without needing comments to explain what a poorly-named thing does. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Naming & Readability?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Naming & Readability violations" clearly means Review; a bare `/dotnet-naming` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: vague or generic names (`data`, `temp`, `Manager`, `Helper`, `DoStuff`) that don't communicate purpose, names that lie about what they actually do (a method called `GetUser` that also has side effects like writing to the database), inconsistent terminology for the same concept across the codebase (a sign the ubiquitous-language discipline has slipped), and comments that exist only to explain what a better name would have made obvious.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: names that actively mislead about behavior (worse than merely vague ones), inconsistent vocabulary for the same domain concept across files
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
- Does this name tell you what it does/represents without needing to read the implementation?
- Does the name accurately describe *all* of what the method does, including side effects, or does it undersell/mislead?
- Is the same concept named consistently across the codebase, or does it go by different terms in different places?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- Naming conventions for different kinds of identifiers (booleans as questions, methods as verbs, classes as nouns)
- How to name for intent rather than implementation (name what it's *for*, not how it currently happens to work)
- When a comment is compensating for a name that should just be better

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story uses terminology consistent with what already exists in the domain/codebase
- Whether the story's proposed names for new concepts would be immediately clear to someone unfamiliar with the current discussion

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at code smells instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
