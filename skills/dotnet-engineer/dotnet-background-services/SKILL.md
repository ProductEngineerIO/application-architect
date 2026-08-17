---
name: dotnet-background-services
description: Evaluate, review, and validate .NET background/hosted service implementations — IHostedService, BackgroundService, worker services, and graceful shutdown handling. Use when assessing long-running background work, reviewing hosted service code, evaluating shutdown/cancellation behavior, or validating architecture stories involving scheduled or continuous background processing.
---

# Background Services

You are evaluating or helping with background and hosted services in .NET — long-running work implemented via `IHostedService`/`BackgroundService`, worker services, and how they handle startup, continuous execution, and graceful shutdown when the host is stopping. This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with background services?
> A) Evaluate my existing background/hosted service implementation
> B) Review a specific code change
> C) Learn/write background service patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my worker service changes" clearly means Review; a bare `/dotnet-background-services` does not).

## Evaluation Context

Use when the user asks to **evaluate** their background/hosted service implementation.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: `BackgroundService`/`IHostedService` implementations and whether `ExecuteAsync` actually honors the `CancellationToken` it's given (a loop that ignores it will hang on shutdown), unhandled exceptions inside the background loop that would silently kill the service without anyone noticing, whether the service creates its own scoped dependencies correctly (a hosted service is typically a singleton, so it needs an `IServiceScopeFactory` to resolve scoped services rather than injecting them directly — a common and subtle DI lifetime bug), and whether long-running or scheduled work has appropriate error isolation so one failure doesn't take down the whole host.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual services and what you found (e.g. "`EmailDigestWorker.ExecuteAsync` runs an infinite `while(true)` loop with no `stoppingToken` check — the host will have to force-kill it on shutdown rather than stopping gracefully")
   - Flag concrete risks: ignored cancellation tokens preventing graceful shutdown, unhandled exceptions silently terminating the background loop, scoped services injected directly into a singleton-lifetime hosted service (see `dotnet-di`), no error isolation between iterations of scheduled/repeating work
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** background service code or changes.

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
- Does the execution loop check/honor the `CancellationToken` so the host can shut it down gracefully rather than forcibly?
- Is there a `try/catch` around each unit of work so one failure doesn't silently kill the entire background loop?
- If this service needs scoped/transient dependencies, is it resolving them per-iteration via `IServiceScopeFactory` rather than holding onto a scoped instance for the service's whole (singleton) lifetime?
- If this is scheduled/periodic work, is the interval/timing logic correct, and does a slow iteration risk overlapping with the next one?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic background-service advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** background/hosted services.

Provide:
- `BackgroundService` implementation patterns that correctly honor `CancellationToken` for graceful shutdown
- Resolving scoped dependencies safely from a singleton-lifetime hosted service via `IServiceScopeFactory`
- Error isolation patterns so one failed iteration doesn't take down the whole worker
- Patterns for periodic/scheduled work (timers, `PeriodicTimer`) vs. continuous processing loops

## Validation Context

Use when the user asks to **validate** a story or requirement involving background/scheduled work.

Check that requirements address:
- What should happen if the host shuts down mid-work — is partial completion acceptable, or does it need to be resumable/idempotent (see `dotnet-idempotency`)?
- What happens if one iteration/run of the work fails — does it affect subsequent runs?
- What dependencies this background work needs, and whether their lifetimes are compatible with running from a long-lived hosted service

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at dependency injection instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-engineer` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
