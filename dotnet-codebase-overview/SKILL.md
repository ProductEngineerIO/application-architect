---
name: dotnet-codebase-overview
description: Analyze an entire codebase (not a single commit or diff) and explain what it does, producing both a technical summary and a plain-language business summary. Use when the user wants to understand, explain, or onboard someone to what a codebase/project/repo does overall. Distinct from dotnet-commit-review and dotnet-business-impact-review, which analyze one commit/diff rather than the whole codebase as it currently stands.
---

# Codebase Overview

You are analyzing an entire codebase as it currently stands (not a diff or commit history) and explaining what it does — producing two distinct summaries, one technical and one business-facing, because both audiences need to understand the system and neither summary alone serves both.

This is a workflow skill, not a topic skill — it doesn't have Evaluate/Review/Write/Validate contexts. It always does one thing: explore a codebase and produce both summaries.

## Step 1: Scope it

If it isn't already clear which codebase or part of it to analyze, ask — e.g. "the whole repo, or a specific project/service within it?" Don't guess at scope for a large monorepo; confirm it.

## Step 2: Explore

Look at: overall directory/project structure, entry points (`Program.cs`, `Startup.cs`, hosted service registrations), the major projects/modules and what each one's name and contents suggest it's responsible for, key domain types and their names (these usually reveal business vocabulary directly), `.csproj`/`packages` for the tech stack and notable dependencies (databases, message brokers, external service clients), and README/docs if present — though don't just repurpose an existing README's wording verbatim, use it as one input among several.

## Step 3: Write the Technical Summary

Aimed at a technical reader (a new engineer, an architect). Cover:
- Overall architecture style, if it's inferable (layered, microservices, event-driven, etc.) — don't force a label if the evidence is thin, say what you actually observed instead
- Major components/services and each one's responsibility
- Key technologies, frameworks, and notable architectural patterns in use
- How data/requests flow through the system, entry points in
- Notable external dependencies (databases, queues, third-party services)

## Step 4: Write the Business Summary

Aimed at a non-technical reader (product, business analyst, new stakeholder). This is where confidence is inherently lower than the technical summary, since business intent has to be inferred from code, naming, and comments rather than read directly off the syntax — be upfront about that rather than presenting inferred business purpose as fact.

- Describe what the system actually does *for the business or its users*, in plain language — lead with capability and purpose, not component names
- Use the domain's own vocabulary where it's evident from the code (consistent with the ubiquitous-language discipline in `dotnet-ddd` and `dotnet-business-impact-review`)
- Explicitly separate what you're confident about (clearly reflected in code/naming/comments) from what you're inferring (plausible but not certain) — don't blend the two without saying which is which
- Where the business purpose of a component genuinely isn't clear from the code alone, say so directly rather than guessing at something plausible-sounding

## Step 5: Suggest where comments would help

As part of the deliverable, call out specific files/classes/modules where you had to infer intent rather than read it directly, and suggest adding a comment there explaining the business "why" — not the technical "what" (the code already shows that). Frame this as helping both future human readers and future AI assistance (including future Claude sessions) understand intent faster next time, rather than having to re-infer it. Be specific — name the actual file/class and what kind of comment would help, not a generic suggestion to "add more comments."

## Navigation

At any natural pause point, offer the user a way to move on. They can say:
- "back to menu" — returns to the `/dotnet-architect` topic menu
- "dig into [area]" — go deeper into a specific topic skill for one part of what was found
- "done" / "that's all" — ends here

If the user says any of these, act on it immediately rather than continuing to add more analysis unprompted.
