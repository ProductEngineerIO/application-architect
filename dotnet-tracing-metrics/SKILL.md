---
name: dotnet-tracing-metrics
description: Evaluate, review, and validate .NET distributed tracing and metrics instrumentation. Use when assessing OpenTelemetry setup, reviewing span/trace propagation across service calls, evaluating metrics coverage, or validating architecture stories involving cross-service observability. Distinct from the dotnet-logging skill, which covers structured log statements rather than traces and metrics.
---

# Distributed Tracing & Metrics

You are evaluating or helping with distributed tracing and metrics in .NET — using OpenTelemetry (or similar) to see a request's path across service boundaries (traces/spans) and to measure system behavior over time (metrics: counters, histograms, gauges). This is the complement to structured logging (which captures discrete events) — tracing answers "where did this request go and how long did each hop take," and metrics answer "how is the system behaving in aggregate." This skill has four contexts: Evaluation, Review, Writing/Learning, and Validation.

**If it's not already clear from the user's message which context they want**, ask before proceeding — don't default to Evaluation. For example:

> What would you like to do with distributed tracing & metrics?
> A) Evaluate my existing tracing/metrics setup
> B) Review a specific code change
> C) Learn/write instrumentation patterns
> D) Validate a story or requirement

Only skip this and go straight to a context if the user's triggering message already made it obvious (e.g. "review my tracing changes" clearly means Review; a bare `/dotnet-tracing-metrics` does not).

## Evaluation Context

Use when the user asks to **evaluate** their tracing/metrics setup.

Default to inspecting the actual codebase rather than asking the user to self-report. Self-reported answers only produce a summary of what the user already believes — the value is in Claude finding things the user didn't already know.

1. If you have access to the codebase, just start looking: whether OpenTelemetry (or an equivalent) is configured at all (`AddOpenTelemetry()`, exporters, instrumentation registrations), whether trace context propagates across service/process boundaries (HTTP calls, message queue publishes) or each service's traces are isolated islands, what custom spans/activities exist around meaningful units of work (vs. relying only on auto-instrumentation), and what metrics are actually emitted — request counts/latencies, custom business metrics, or nothing beyond framework defaults.
2. Only ask the user a clarifying question first if you have no code access, or need direction on which project/service to focus on.
3. Once you've looked, report **specific findings**, not a generic score:
   - Name actual services/call paths and what you found (e.g. "`OrderService` calls `PaymentService` over HTTP without propagating the trace `traceparent` header — the two services' traces won't link up in the backend")
   - Flag concrete risks: broken trace propagation across service/queue boundaries, no custom spans around slow or business-critical operations (so a trace shows a black box where the interesting work happens), missing metrics for things that matter operationally (error rates, queue depth, external call latency), auto-instrumentation relied on exclusively with no domain-specific signal
   - Give a score (0-100) as a summary *after* the findings, grounded in what you actually saw
   - Prioritize the 1-3 things most worth fixing, with concrete next steps

If the user answers with a self-assessment before you've looked at code, treat it as a hypothesis to verify, not a final answer — offer to confirm it against the actual code, the same as you would for "I don't know."

## Review Context

Use when the user asks to **review** tracing/metrics code or changes.

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
- Does trace context actually propagate across this boundary (HTTP headers, message metadata), or will this call start a disconnected trace?
- Is a new span/activity started around a unit of work that's actually worth seeing individually, with useful tags/attributes (not just a bare span with no context)?
- Are metrics recorded with appropriate cardinality — tags that are useful for filtering, not high-cardinality values (like a raw user ID) that will blow up the metrics backend?
- Is instrumentation code cluttering business logic, or kept reasonably separate (middleware, decorators, dedicated instrumentation points)?

Give specific, line/file-referenced feedback tied to what's actually in the diff or pasted code — not generic observability advice.

## Writing/Learning Context

Use when the user asks to **learn** or **write** tracing/metrics instrumentation.

Provide:
- OpenTelemetry setup for traces and metrics (`AddOpenTelemetry()`, instrumentation packages, exporters)
- Trace context propagation patterns across HTTP calls and message queues (ensuring `traceparent` or equivalent flows through)
- Custom `Activity`/span creation around meaningful business operations, with useful tags
- Metric types (counter, histogram, gauge) and when each fits; cardinality guidance for tags
- How tracing/metrics complement structured logging rather than duplicate it (logs = discrete events with detail; traces = request path and timing; metrics = aggregate trends)

## Validation Context

Use when the user asks to **validate** a story or requirement involving cross-service observability.

Check that requirements address:
- Which cross-service call paths need to be traceable end-to-end for this feature
- What metrics would actually answer "is this working correctly in production" for this feature (not just default framework metrics)
- Whether trace propagation is considered for any new external call, queue publish, or background job introduced by the story
- Whether alerting/dashboards built on these metrics are in scope, not just emitting the data

## Navigation

At any natural pause point (after an evaluation summary, after a deep-dive, after answering a follow-up), offer the user a way to move on rather than only prompting for more depth. For example, alongside a "want to dig deeper?" question, mention they can also say things like:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "done" / "that's all" — ends this skill's guidance here
- or name another topic directly (e.g. "let's look at logging instead")

If the user says any of these (or similar), stop the current line of questioning immediately and either invoke the `dotnet-architect` menu skill, end helpfully, or switch to the topic they named — don't keep probing for more detail first.
