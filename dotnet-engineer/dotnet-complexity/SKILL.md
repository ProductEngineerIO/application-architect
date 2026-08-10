---
name: dotnet-complexity
description: Evaluate, review, and validate cyclomatic complexity and method/class size in .NET code. Use when assessing whether methods have too many branching paths, reviewing deeply nested conditional logic, evaluating testability impact of complexity, or validating architecture stories for implementation complexity risk.
---

# Cyclomatic Complexity & Method Size

You are evaluating or helping with cyclomatic complexity and method/class size — how many independent execution paths a method has, and how that correlates with difficulty to understand, test, and safely change. High complexity means more test cases needed for full coverage and more risk of an untested path hiding a bug. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Cyclomatic Complexity & Method Size?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for Cyclomatic Complexity & Method Size violations" clearly means Review; a bare `/dotnet-complexity` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: methods with deep nesting (multiple levels of `if`/`for`/`switch` inside each other), methods with many branching points (each `if`, `else if`, `case`, `&&`/`||` in a condition, and loop adds a path), long parameter lists that often correlate with a method doing too much, and methods where you'd need many test cases to cover every path — a direct complexity signal.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: methods complex enough that full test coverage is impractical, deep nesting that makes the actual logic hard to trace by eye
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
- Roughly how many independent paths does this method have (count the branches/conditions)? Would testing all of them be practical?
- Is nesting deep enough that the core logic is hard to follow, and could early returns/guard clauses flatten it?
- Does this method's complexity correlate with it also doing too much (worth cross-checking against SRP)?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to count/estimate cyclomatic complexity by eye (branches + 1)
- Flattening techniques: guard clauses/early returns instead of nested conditionals, extracting branches into named methods
- Where complexity is often hiding a missing abstraction (e.g. a big switch that really wants to be polymorphism — see OCP)

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the story's described logic has many conditional branches/edge cases that should be planned for and tested explicitly rather than discovered during implementation
- Whether the story hints at a method that will need to handle many special cases (a complexity risk worth designing around up front)

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at code smells instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
