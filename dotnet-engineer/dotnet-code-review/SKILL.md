---
name: dotnet-code-review
description: Perform a normal, general-purpose code review of a diff or commit — correctness, obvious bugs, edge cases, readability, style consistency, and whether tests cover the change — the kind of review a peer would leave on a pull request. Use when the user wants a typical/standard code review rather than a deep architectural audit. Distinct from dotnet-commit-review, which routes the diff through the full SOLID/DDD/DI/async/etc. catalog and looks for cross-cutting architectural issues; this skill is the everyday review, not the deep multi-lens one.
---

# General Code Review

You are performing an ordinary code review of a diff or commit — the kind of review a competent peer would leave on a pull request. This is deliberately **not** the deep architectural audit (`dotnet-commit-review` covers that, routing through the full principle/practice catalog and looking for cross-cutting issues). This skill stays at the level of: is this correct, is it clear, are edge cases handled, is it consistent with the surrounding code, and is it tested.

This is a workflow skill, not a topic skill — it doesn't have Evaluate/Review/Write/Validate contexts. It always does one thing: take a diff and produce a normal PR-style review of it.

## Step 1: Get the diff

If it isn't already clear (e.g. the user already pasted code, or already named a commit/hash), ask:

> How would you like to give me the commit to review?
> A) Paste the diff directly
> B) Pick from your last 10 commits
> C) Give me a commit hash or range

- If **B**: run exactly `git log -10 --oneline` (add `-- <path>` only if the user already narrowed it to a project/service). Do not run any other git commands yet — just show the resulting list with numbers and stop to let them pick. Only after they pick a specific commit, run `git show <hash>` or `git diff <hash>^ <hash>` to get that one diff.
- If **C**: take the hash/range they give and run `git show <hash>` (single commit) or `git diff <hash1> <hash2>` (range) to get the diff.
- If **A**: wait for them to paste it.
- If there's no git repo available (e.g. `git log` fails) or they're not in a repo context, skip straight to asking them to paste the diff.

## Step 2: Review it like a normal PR reviewer would

Focus on:
- **Correctness** — logic errors, off-by-one mistakes, wrong operators/conditions, incorrect assumptions about inputs
- **Edge cases** — null/empty/zero/boundary values, what happens on the unhappy path, whether error cases are actually handled or just ignored
- **Readability** — can you tell what this does and why without excessive effort; would a teammate unfamiliar with this specific change understand it
- **Consistency** — does it match the style, patterns, and conventions already used nearby, or does it introduce an unnecessary one-off way of doing something
- **Test coverage** — is there a test for this change; if not, should there be one, and what would it need to cover
- **Does it do what it claims** — if there's a commit message or comment describing intent, does the actual diff match it

This is intentionally a shallower, broader pass than the principle-by-principle audit — don't go deep into SOLID/DDD/etc. analysis here. If something does jump out as a deeper architectural concern worth a dedicated look, it's fine to mention it briefly and point to the relevant topic skill or to `dotnet-commit-review` for a fuller treatment — but don't turn this into that review.

## Step 3: Present findings like PR comments

Structure the output the way review comments would actually appear — grouped by file/location, each with a brief note on severity (blocking vs. suggestion vs. nitpick). Close with a short overall verdict: would you approve this, request changes, or approve with minor comments.

## Navigation

At any natural pause point, offer the user a way to move on. They can say:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "do a deeper architectural review" — hands off to `dotnet-commit-review` for the same diff
- "done" / "that's all" — ends here

If the user says any of these, act on it immediately rather than continuing to add more analysis unprompted.
