---
name: dotnet-engineer
description: Menu of .NET architecture guidance topics, including general software engineering principles (SOLID, DRY, KISS, YAGNI, Idempotency, coupling/cohesion, DDD) and .NET-specific topics (Dependency Injection, Configuration, Async/Await, Messaging & Event-Driven Architecture, Background Services, Data Access & EF Core, Error Handling, Structured Logging, Concurrency & Thread Safety, Input Validation, Authentication & Authorization, Testing, Records/Immutability, Distributed Tracing & Metrics, CQRS & MediatR, API Versioning & Contract Design, Nullable Reference Types, LINQ, Minimal APIs, Feature Flags, Caching, Performance & Memory), plus cross-cutting workflow tools (multi-lens Commit Review, everyday General Code Review, plain-language Business Impact Review, whole-codebase Overview). Use when the user asks for general software architecture or .NET architecture help, wants to see available architecture topics, or isn't sure which specific area to focus on.
---

# .NET Engineer Toolkit

You are presenting a menu of available .NET architectural guidance topics. Show the user this menu and help them pick a direction.

## Available Topics

### Tier 0: Foundational Principles

**SOLID**
- **Single Responsibility (SRP)** (`/dotnet-srp`)
- **Open/Closed (OCP)** (`/dotnet-ocp`)
- **Liskov Substitution (LSP)** (`/dotnet-lsp`)
- **Interface Segregation (ISP)** (`/dotnet-isp`)
- **Dependency Inversion (DIP)** (`/dotnet-dip`)

**Other Core Principles**
- **DRY** (`/dotnet-dry`)
- **KISS** (`/dotnet-kiss`)
- **YAGNI** (`/dotnet-yagni`)
- **Idempotency** (`/dotnet-idempotency`)

**Structural Principles**
- **Coupling & Cohesion** (`/dotnet-coupling-cohesion`)
- **Separation of Concerns** (`/dotnet-separation-of-concerns`)
- **Composition over Inheritance** (`/dotnet-composition-over-inheritance`)

**Code-Level Quality**
- **Naming & Readability** (`/dotnet-naming`)
- **Code Smells & Refactoring** (`/dotnet-code-smells`)
- **Cyclomatic Complexity** (`/dotnet-complexity`)

**Paradigms**
- **Domain-Driven Design (DDD)** (`/dotnet-ddd`)

### Tier 1: Foundational

- **Dependency Injection** (`/dotnet-di`)
- **Configuration & Options** (`/dotnet-config`)
- **Async/Await** (`/dotnet-async`)
- **Messaging & Event-Driven Architecture** (`/dotnet-messaging`)
- **Background Services** (`/dotnet-background-services`)
- **Data Access & EF Core Patterns** (`/dotnet-data-access`)

### Tier 2: Critical

- **Error Handling & Result Types** (`/dotnet-error-handling`)
- **Structured Logging & Observability** (`/dotnet-logging`)
- **Concurrency & Thread Safety** (`/dotnet-concurrency`)
- **Input Validation** (`/dotnet-validation`)
- **Authentication & Authorization** (`/dotnet-auth`)

### Tier 3: Important

- **Testing** (`/dotnet-testing`)
- **Records & Immutability** (`/dotnet-records`)
- **Distributed Tracing & Metrics** (`/dotnet-tracing-metrics`)
- **CQRS & MediatR Patterns** (`/dotnet-cqrs-mediatr`)
- **API Versioning & Contract Design** (`/dotnet-api-versioning`)

### Tier 4: Quality

- **Nullable Reference Types** (`/dotnet-nrt`)
- **LINQ** (`/dotnet-linq`)
- **Minimal APIs** (`/dotnet-minimal-apis`)
- **Feature Flags** (`/dotnet-feature-flags`)
- **Caching** (`/dotnet-caching`)
- **Performance & Memory** (`/dotnet-performance`)

### Tools / Workflows (not tiered)

These aren't single-topic principles or practices — they're workflows that operate across the whole catalog above.

- **Commit Review** (`/dotnet-commit-review`) — deep architectural review of a commit/diff: identifies which areas above it touches, reviews each relevant one, and calls out issues that only emerge from combinations (e.g. a change touching both Dependency Injection and Concurrency).
- **General Code Review** (`/dotnet-code-review`) — a normal, everyday PR-style review of a commit/diff: correctness, edge cases, readability, style consistency, test coverage. The shallower, broader counterpart to Commit Review, not a replacement for it.
- **Business Impact Review** (`/dotnet-business-impact-review`) — reviews a commit/diff and produces a plain-language, shareable summary of what changed for the business, not the architecture — including inferring business rules from code and flagging changes that may need business/compliance sign-off.
- **Codebase Overview** (`/dotnet-codebase-overview`) — analyzes an entire codebase (not just one commit) and explains what it does, with both a technical summary and a plain-language business summary, plus suggestions for where comments would help future understanding.

## What To Do

1. Present the menu above to the user in a clean, readable format — tiers, and within Tier 0 the SOLID / Other Core Principles / Structural Principles / Code-Level Quality / Paradigms categories, plus the Tools/Workflows category, with just names. Do not list out the four contexts (Evaluate/Review/Write/Validate) here; that menu belongs inside each topic skill, shown after the user picks a topic. Tools/Workflows skills don't have that sub-menu — they're single-purpose, so just name them.
2. Ask which topic (or tool/workflow) they want to explore.
3. Once they choose a topic skill, invoke the matching skill and let it ask which context (Evaluate/Review/Write/Validate) they want. Topic skill names: `dotnet-srp`, `dotnet-ocp`, `dotnet-lsp`, `dotnet-isp`, `dotnet-dip`, `dotnet-dry`, `dotnet-kiss`, `dotnet-yagni`, `dotnet-idempotency`, `dotnet-coupling-cohesion`, `dotnet-separation-of-concerns`, `dotnet-composition-over-inheritance`, `dotnet-naming`, `dotnet-code-smells`, `dotnet-complexity`, `dotnet-ddd`, `dotnet-di`, `dotnet-config`, `dotnet-async`, `dotnet-messaging`, `dotnet-background-services`, `dotnet-data-access`, `dotnet-error-handling`, `dotnet-logging`, `dotnet-concurrency`, `dotnet-validation`, `dotnet-auth`, `dotnet-testing`, `dotnet-records`, `dotnet-tracing-metrics`, `dotnet-cqrs-mediatr`, `dotnet-api-versioning`, `dotnet-nrt`, `dotnet-linq`, `dotnet-minimal-apis`, `dotnet-feature-flags`, `dotnet-caching`, `dotnet-performance`. If they instead choose a Tools/Workflows entry, invoke `dotnet-commit-review`, `dotnet-code-review`, `dotnet-business-impact-review`, or `dotnet-codebase-overview` directly — these skip straight into their own diff-gathering (or codebase-scoping) step rather than asking for a context.

If the user already stated what they want (e.g., "evaluate my DI setup") in the same message that triggered this menu, skip straight to that topic and context instead of showing the menu.

This menu can also be reached mid-conversation — if the user says "back to menu," "start over," or similar while working through any of the topic skills, treat that as an invocation of this skill and show the menu again.
