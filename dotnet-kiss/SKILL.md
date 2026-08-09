---
name: dotnet-kiss
description: Evaluate, review, and validate adherence to the Keep It Simple (KISS) principle in .NET code. Use when assessing unnecessary complexity, reviewing over-engineered solutions, evaluating whether a simpler approach exists, or validating architecture stories for scope appropriate to the actual problem.
---

# KISS (Keep It Simple)

You are evaluating or helping with the KISS principle — prefer the simplest solution that correctly solves the actual problem; avoid complexity that isn't earning its keep. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with KISS (Keep It Simple)?
> A) Evaluate my existing code against this principle
> B) Review a specific code change
> C) Learn/write code applying this principle
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my changes for KISS (Keep It Simple) violations" clearly means Review; a bare `/dotnet-kiss` does not).

## Evaluation Context

Use when the user asks to **evaluate** adherence to this principle.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: unnecessary layers of abstraction for a problem that doesn't need them (e.g. a generic plugin/strategy system built for something with exactly one implementation and no near-term second one), clever code that's hard to follow when a straightforward version would do, and configuration/flexibility points that nothing actually uses or varies.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual classes/methods and what you found
   - Flag concrete risks: abstraction layers with only one real implementation, configurability nobody uses, code that's clever rather than clear
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
- Does this abstraction/layer/pattern earn its complexity given the actual current requirements, or is it solving a problem that doesn't exist yet?
- Could this be expressed more simply without losing correctness or necessary flexibility?
- Is this code clever in a way that makes it harder to read than a plainer equivalent?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic principle-recitation.

## Writing/Learning Context

Use when the user asks to **learn** or **write** code applying this principle.

Provide:
- How to recognize the actual complexity a problem requires vs. complexity being added speculatively
- Refactoring patterns for simplifying over-engineered code back to what the problem actually needs
- How KISS and YAGNI reinforce each other (KISS is about the solution's shape, YAGNI is about not building for the future yet)

## Validation Context

Use when the user asks to **validate** a story or requirement against this principle.

Check that requirements address:
- Whether the proposed design's complexity matches the problem's actual current scope
- Whether the story is asking for flexibility/configurability that has a concrete near-term need, or is speculative

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at YAGNI instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
