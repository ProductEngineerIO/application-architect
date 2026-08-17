---
name: dotnet-minimal-apis
description: Evaluate, review, and validate .NET Minimal API endpoint design. Use when assessing HTTP routing and endpoint structure, reviewing endpoint code, evaluating whether business logic leaks into endpoints, or validating architecture stories involving API surface design.
---

# Minimal APIs Guidance

You are evaluating or helping with .NET Minimal API design — keeping endpoints thin (routing, DI-provided dependencies, response mapping) with business logic living in the service layer, not the endpoint itself. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with Minimal APIs?
> A) Evaluate my existing endpoint design
> B) Review a specific code change
> C) Learn/write endpoint patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my endpoint changes" clearly means Review; a bare `/dotnet-minimal-apis` does not).

## Evaluation Context

Use when the user asks to **evaluate** their Minimal API endpoint design.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: `Program.cs`/route registration files for endpoint definitions, whether endpoint handlers contain business logic (validation, calculations, orchestration) instead of just calling a service, whether DTOs (not domain entities) are used for request/response contracts, and consistency of status-code/error-response mapping across endpoints.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual endpoints and what you found (e.g. "`app.MapPost(\"/orders\", ...)` handler contains 40 lines of inventory-check and pricing logic inline — should be delegated to an order service")
   - Flag concrete risks: business logic embedded directly in route handlers, domain entities exposed directly as API responses instead of DTOs, inconsistent error-to-status-code mapping across endpoints, missing input validation before hitting the service layer
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** endpoint code or changes.

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
- Is the handler thin — just receiving DI-provided dependencies, calling a service, mapping the result to a response?
- Is business logic (validation beyond basic shape, calculations, multi-step orchestration) leaking into the endpoint?
- Are DTOs used for request/response instead of exposing domain entities directly?
- Is the status-code/error mapping consistent with how other endpoints in the codebase handle similar situations?
- Is the route named/grouped sensibly relative to the rest of the API surface?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic API design advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** Minimal API endpoints.

Provide:
- Thin-handler pattern: DI-provided dependencies as parameters, delegate to a service, map result to response
- DTO patterns for request/response contracts distinct from domain entities
- Consistent error-to-status-code mapping approach (e.g. via a shared Result-to-IResult mapper)
- Route grouping and organization patterns for larger API surfaces

## Validation Context

Use when the user asks to **validate** a story or requirement involving API design.

Check that requirements address:
- What the request/response contract looks like (and whether it should be a dedicated DTO)
- Which validation belongs at the API boundary vs. deeper in the service layer
- What error/status-code behavior is expected for failure cases

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at CQRS/MediatR instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
