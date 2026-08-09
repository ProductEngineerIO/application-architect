---
name: dotnet-auth
description: Evaluate, review, and validate .NET authentication and authorization implementations — JWT/token validation, ASP.NET Core Identity, and policy-based authorization. Use when assessing whether endpoints are properly secured, reviewing auth-related code, evaluating authorization policy design, or validating architecture stories involving access control.
---

# Authentication & Authorization

You are evaluating or helping with authentication (proving who a caller is) and authorization (deciding what an authenticated caller is allowed to do) in .NET — token/JWT validation, ASP.NET Core Identity, and policy-based authorization applied consistently across the API surface. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with authentication & authorization?
> A) Evaluate my existing auth setup
> B) Review a specific code change
> C) Learn/write auth patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my authorization policy changes" clearly means Review; a bare `/dotnet-auth` does not).

## Evaluation Context

Use when the user asks to **evaluate** their authentication/authorization setup.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: which endpoints have `[Authorize]` (or an equivalent) and which don't — an endpoint with no attribute at all is easy to miss and easy to get wrong; whether authorization is policy-based (centralized, named, testable) or scattered ad hoc role/claim checks inline in handlers; token validation configuration (issuer, audience, expiration, signing key handling) for anything that looks loosely configured; and whether authorization checks happen at a consistent layer (API boundary) or are duplicated/inconsistently re-implemented deeper in the code.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual endpoints/policies and what you found (e.g. "`DELETE /accounts/{id}` has no `[Authorize]` attribute at all — any authenticated *or unauthenticated* caller can hit it, depending on the global default policy")
   - Flag concrete risks: unprotected endpoints (especially state-changing ones), authorization logic duplicated ad hoc instead of expressed as reusable named policies, overly permissive default policies, token validation configured loosely (e.g. no expiration check, accepting any issuer), authorization checks that can be bypassed by calling a lower-level method directly instead of going through the guarded entry point
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps — treat missing authorization on state-changing endpoints as high priority, not a minor note

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** authentication/authorization code or changes.

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
- Is this new/changed endpoint properly protected — does it have an explicit authorization requirement, or is it relying on an assumption about the default policy?
- If a new authorization rule is added, is it expressed as a named, reusable policy, or an inline check that will need to be copy-pasted anywhere else the same rule applies?
- Does authorization happen at the entry point, or is there a path that reaches the protected logic without going through the check (e.g. a service method called directly from another internal path)?
- For anything touching token validation or identity configuration, does the change loosen a check (accepting a wider audience, longer expiration, weaker signing requirements) without an explicit reason?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic security advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** authentication/authorization code.

Provide:
- Policy-based authorization patterns (named policies, requirement handlers) instead of scattered inline role/claim checks
- JWT/token validation configuration and what each setting (issuer, audience, expiration, signing key) actually protects against
- ASP.NET Core Identity setup basics where relevant
- Guidance on applying authorization consistently — at the boundary, in a way that can't be bypassed by an alternate code path

## Validation Context

Use when the user asks to **validate** a story or requirement involving access control.

Check that requirements address:
- Who should and shouldn't be able to perform this action — stated explicitly, not left implicit
- Whether a new or existing authorization policy covers this, or a new one is needed
- Whether this is a state-changing operation that needs explicit protection called out as part of the work, not assumed to inherit protection from elsewhere

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at input validation instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
