---
name: dotnet-isp
description: Evaluate, review, and validate adherence to the Interface Segregation Principle (ISP) in .NET code. Use when assessing whether interfaces are too broad, reviewing interface design, evaluating classes forced to implement unused members, or validating architecture stories involving interface contracts.
---

# Interface Segregation Principle

You are evaluating or helping with the Interface Segregation Principle (ISP) — clients shouldn't be forced to depend on interface members they don't use; prefer several small, focused interfaces over one large one. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Interface Segregation Principle?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Interface Segregation Principle violations" clearly means Review; a bare `/dotnet-isp` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: interfaces with many members where implementers only use a subset, implementations that throw `NotImplementedException` for members they're forced to implement but don't support, "fat" interfaces that different consumers use only slivers of, and any interface that seems to have grown over time by tacking on unrelated members.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: interfaces forcing irrelevant members on implementers, consumers depending on an entire fat interface when they only call one or two methods
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
- Does every implementer of this interface actually need every member, or are some forced/no-op?
- Would this interface split naturally along how different consumers actually use it?
- Is a consumer depending on a large interface when a smaller, role-specific one would do?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to identify natural interface seams by looking at how different consumers actually use a type
- Patterns for splitting a fat interface into role-based interfaces (e.g. `IReadable`/`IWritable` instead of one `IRepository`)
- How this interacts with DIP — small interfaces make dependency inversion more precise

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's proposed interface/contract is scoped to what this specific consumer actually needs
- Whether the story is adding a member to an existing interface that not all implementers will support

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at DIP instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
