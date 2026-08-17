---
name: dotnet-feature-flags
description: Evaluate, review, and validate .NET feature flag usage — toggle implementation, flag lifecycle, and default/fallback behavior. Use when assessing scattered or stale feature flags, reviewing new flag additions, evaluating flag cleanup debt, or validating architecture stories involving progressive delivery or kill switches.
---

# Feature Flags

You are evaluating or helping with feature flags in .NET — toggles that let functionality be enabled/disabled without a deploy (progressive delivery, kill switches, experimentation) — with particular attention to flag lifecycle, since flags that never get removed after their rollout is done become a permanent source of complexity and a form of technical debt. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with feature flags?
> A) Evaluate my existing feature flag usage
> B) Review a specific code change
> C) Learn/write feature flag patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my new flag" clearly means Review; a bare `/dotnet-feature-flags` does not).

## Evaluation Context

Use when the user asks to **evaluate** their feature flag usage.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: how flags are checked (a centralized flag service/abstraction vs. ad hoc config booleans scattered through the code), how many flag checks exist and whether any look old/permanent (a strong signal of unremoved flag debt — e.g. checked in git blame/history as present for a long time, or a name suggesting a rollout that's clearly long finished), what happens if the flag evaluation service is unavailable (fails open, fails closed, or crashes), and whether both states of a flag (on and off) appear to be covered by tests, or only the currently-active path.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual flags/checks and what you found (e.g. "`UseNewCheckoutFlow` flag has been in the codebase since [earliest visible reference] and is checked in 6 places — looks like a completed rollout that was never cleaned up")
   - Flag concrete risks: stale flags never removed after rollout, flag checks scattered inline rather than isolated at a clean boundary, no defined fallback behavior if the flag service is unreachable, only one branch of a flag actually being tested
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** feature flag code or changes.

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
- Is the flag check localized to one clean boundary (e.g. a strategy selection at startup or request entry), or scattered as inline `if` checks throughout unrelated code?
- Is there a clear, safe default if the flag can't be evaluated (service down, config missing)?
- Is this a genuinely temporary rollout flag, or something that's really a permanent configuration setting mislabeled as a "flag" (in which case it belongs in `dotnet-config` territory instead)?
- Does the change include or reference a plan for removing the flag once the rollout is complete, or does it look like it'll join the pile of stale ones?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic feature-flag advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** feature flag code.

Provide:
- Clean toggle patterns — isolating the flag check at one boundary (e.g. strategy/factory selection) rather than branching inline everywhere the behavior differs
- Default-safe behavior when the flag service is unavailable
- The distinction between a temporary rollout/kill-switch flag (should have a removal plan from day one) and a permanent configuration setting (belongs in `dotnet-config`, not as a flag)
- Testing both flag states explicitly, not just whichever is currently active in each environment

## Validation Context

Use when the user asks to **validate** a story or requirement involving a feature flag.

Check that requirements address:
- What the flag's removal/cleanup criteria are — when does this flag get deleted, and is that tracked anywhere
- What the safe default behavior is if flag evaluation fails
- Whether this is genuinely a temporary rollout mechanism or actually permanent configuration mislabeled as a flag

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at configuration instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
