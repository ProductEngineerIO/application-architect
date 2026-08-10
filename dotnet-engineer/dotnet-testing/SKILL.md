---
name: dotnet-testing
description: Evaluate, review, and validate .NET test suites and test quality in general — unit tests, integration tests, mocking, coverage, and test design. Use when assessing test coverage or quality, reviewing test code, evaluating whether tests actually verify behavior, or validating architecture stories for testability requirements.
---

# Testing Guidance

You are evaluating or helping with .NET test suites in general — unit tests, integration tests, test structure, mocking strategy, and coverage of meaningful behavior (not just line coverage). This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with testing?
> A) Evaluate my existing test suite
> B) Review a specific test code change
> C) Learn/write test patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my test changes" clearly means Review; a bare `/dotnet-testing` does not).

## Evaluation Context

Use when the user asks to **evaluate** their test suite or test quality.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: test project structure and naming conventions, what's covered vs. what isn't (business logic, edge cases, error paths — not just happy path), mocking strategy (over-mocked to the point tests verify nothing real, or under-mocked so tests are slow/flaky), assertion quality (testing actual behavior vs. implementation details that break on harmless refactors), and any code-specific concerns worth flagging — e.g. if the codebase has heavy async or DI usage, whether tests actually exercise that correctly (real `await`, not blocking; DI-wired integration tests, not everything mocked away).
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual test classes/methods and what you found (e.g. "`OrderServiceTests` only covers the successful checkout path — no test for insufficient inventory or payment failure")
   - Flag concrete risks: untested error/edge-case paths, brittle tests coupled to implementation rather than behavior, over-mocking that makes tests pass even when the real integration is broken, missing tests entirely for critical business logic
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** test code or changes.

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
- Does the test name and structure make it clear what's being verified and why?
- Is the assertion testing observable behavior, not internal implementation details?
- Is the mocking level appropriate — enough isolation to be fast/reliable, not so much that the test proves nothing?
- Are edge cases and failure paths covered, not just the happy path?
- If the code under test is async or DI-based: does the test actually exercise that correctly (real `await`, not `.Result`/`.Wait()`; realistic mock behavior, not just instant success)?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic testing advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** tests.

Provide:
- Test structure (Arrange-Act-Assert), naming conventions that describe behavior
- Mocking guidance — when to mock a dependency vs. use a real/in-memory implementation
- Patterns for covering edge cases and failure paths systematically, not just the happy path
- If relevant to what they're testing: patterns for testing async code correctly, or for integration-testing DI-wired services

## Validation Context

Use when the user asks to **validate** a story or requirement involving testability.

Check that requirements address:
- What test coverage is expected (unit vs. integration) for the work
- Whether failure/edge-case scenarios are explicitly called out as needing tests, not just the happy path
- Whether the design as described is actually testable (e.g. can dependencies be substituted, or are they hardcoded?)

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at async instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
