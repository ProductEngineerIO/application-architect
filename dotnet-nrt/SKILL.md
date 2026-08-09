---
name: dotnet-nrt
description: Evaluate, review, and validate .NET nullable reference type usage. Use when assessing null-safety, reviewing nullable annotations, evaluating whether nullable warnings are being addressed, or validating architecture stories involving null handling.
---

# Nullable Reference Types Guidance

You are evaluating or helping with nullable reference types (NRT) in .NET — using the compiler's nullable annotation context to catch null-reference bugs at compile time instead of runtime. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with nullable reference types?
> A) Evaluate my existing null-safety
> B) Review a specific code change
> C) Learn/write nullable-aware patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my nullable changes" clearly means Review; a bare `/dotnet-nrt` does not).

## Evaluation Context

Use when the user asks to **evaluate** their nullable reference type usage.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: whether `<Nullable>enable</Nullable>` is set project-wide (or per-file `#nullable enable`), how many `!` null-forgiving operators are used and whether they're justified or just suppressing real warnings, whether properties that are genuinely optional are marked `?` vs. required properties left non-nullable, and any lingering `#pragma warning disable` for nullable warnings.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual files/classes and what you found (e.g. "`CustomerService.GetByEmail()` returns `Customer` but can return null on no-match — should be `Customer?`, currently forces callers to null-check something the compiler doesn't know can be null")
   - Flag concrete risks: nullable disabled entirely (missing the safety net), heavy `!` usage masking real nullability, public APIs with inaccurate nullable annotations, mismatched annotations vs. actual runtime behavior
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** nullable-related code or changes.

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
- Are nullable annotations accurate — does `?` actually mean "can be null" and non-`?` actually mean "guaranteed non-null"?
- Is `!` used to suppress a real warning that should instead be handled with a null check?
- Are new public APIs annotated correctly so callers get accurate compiler help?
- Is a genuinely optional value being treated as required (or vice versa)?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic nullability advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** nullable-aware code.

Provide:
- How to enable and configure nullable reference types at the project level
- Patterns for handling genuinely optional values (`?`, pattern matching, `??`/`??=`)
- When `!` is actually appropriate vs. when it's a code smell
- Guidance on annotating public APIs accurately for consumers

## Validation Context

Use when the user asks to **validate** a story or requirement involving null handling.

Check that requirements address:
- Which values in the new functionality are genuinely optional vs. always required
- Whether "not found" / "missing" cases are handled explicitly rather than assumed away
- Whether public contracts (APIs, DTOs) will have accurate nullable annotations

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at error handling instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
