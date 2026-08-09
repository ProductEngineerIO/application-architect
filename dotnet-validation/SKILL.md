---
name: dotnet-validation
description: Evaluate, review, and validate .NET input validation — where validation happens in the pipeline, validation library usage (FluentValidation, DataAnnotations), and the distinction between shape validation at the boundary and business rule validation in the domain. Use when assessing validation coverage, reviewing validation logic placement, evaluating error message quality, or validating architecture stories involving input handling. Related to but distinct from dotnet-ddd (which covers invariants enforced by the domain model itself, a deeper layer than boundary validation).
---

# Input Validation

You are evaluating or helping with input validation in .NET — checking that data crossing a boundary (API request, message payload, form input) is well-formed and acceptable before it's acted on, and knowing the difference between shape/format validation at the boundary (is this a valid email format, is this field present) and business rule validation that belongs in the domain model itself (see `dotnet-ddd` for invariants like "a campaign's start date can't be in the past" — that's a domain rule, not just input shape). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with input validation?
> A) Evaluate my existing validation coverage
> B) Review a specific code change
> C) Learn/write validation patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my validation changes" clearly means Review; a bare `/dotnet-validation` does not).

## Evaluation Context

Use when the user asks to **evaluate** their input validation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: whether validation is applied consistently at entry points (API endpoints, message handlers) or only on some, what validation approach is used (FluentValidation, DataAnnotations, manual checks) and whether it's applied consistently rather than each endpoint doing its own thing, whether validation is purely shape-level (types, required fields, format) with actual business rules duplicated or missing at the domain layer, and the quality of validation error messages/responses (do they tell the caller what's actually wrong, or just fail generically).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual endpoints/handlers and what you found (e.g. "`POST /campaigns` has no validation on `startDate` at the boundary — a malformed or nonsensical date reaches the domain layer before anything catches it")
   - Flag concrete risks: inconsistent validation coverage across similar endpoints, business rules checked only at the boundary (bypassable if the domain method is called from anywhere else) instead of being real domain invariants, validation logic duplicated across multiple entry points instead of shared, unhelpful/generic error responses that don't tell the caller what to fix
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** validation code or changes.

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
- Is this actually shape/format validation (belongs at the boundary), or a business rule that should be a domain invariant instead (belongs in the model itself, so it can't be bypassed by another code path)?
- Is validation applied at this entry point consistently with how similar entry points elsewhere in the codebase handle it?
- Does the error response tell the caller specifically what was wrong, or just that "something" failed?
- Is validation logic duplicated here when it could be shared with an existing validator?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic validation advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** validation code.

Provide:
- FluentValidation / DataAnnotations patterns for boundary shape validation
- The boundary-validation vs. domain-invariant distinction, and guidance on which one a given rule actually is
- Patterns for sharing validation logic across multiple entry points instead of duplicating it
- Writing error responses that are specific enough to be actionable for the caller

## Validation Context

Use when the user asks to **validate** a story or requirement involving input handling.

Check that requirements address:
- What's genuinely invalid input (wrong shape/type/missing) vs. a business rule that should live in the domain model
- What the expected error behavior is for invalid input — not just "reject it," but what the caller should see
- Whether this validation needs to be consistent with how a similar existing feature already handles it

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at DDD instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
