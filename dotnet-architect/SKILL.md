---
name: dotnet-architect
description: Menu of .NET architecture guidance topics, including general software engineering principles (SOLID, DRY, KISS, YAGNI, coupling/cohesion, DDD) and .NET-specific topics (Dependency Injection, Configuration, Async/Await, Error Handling, Structured Logging, Testing, Records/Immutability, Distributed Tracing & Metrics, Nullable Reference Types, LINQ, Minimal APIs). Use when the user asks for general software architecture or .NET architecture help, wants to see available architecture topics, or isn't sure which specific area to focus on.
---

# NET Architect Toolkit

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

### Tier 2: Critical

- **Error Handling & Result Types** (`/dotnet-error-handling`)
- **Structured Logging & Observability** (`/dotnet-logging`)

### Tier 3: Important

- **Testing** (`/dotnet-testing`)
- **Records & Immutability** (`/dotnet-records`)
- **Distributed Tracing & Metrics** (`/dotnet-tracing-metrics`)

### Tier 4: Quality

- **Nullable Reference Types** (`/dotnet-nrt`)
- **LINQ** (`/dotnet-linq`)
- **Minimal APIs** (`/dotnet-minimal-apis`)

## What To Do

1. Present the menu above to the user in a clean, readable format — tiers, and within Tier 0 the SOLID / Other Core Principles / Structural Principles / Code-Level Quality / Paradigms categories, with just topic names. Do not list out the four contexts (Evaluate/Review/Write/Validate) here; that menu belongs inside each topic skill, shown after the user picks a topic.
2. Ask which topic they want to explore.
3. Once they choose a topic, invoke the matching skill and let it ask which context (Evaluate/Review/Write/Validate) they want. Topic skill names: `dotnet-srp`, `dotnet-ocp`, `dotnet-lsp`, `dotnet-isp`, `dotnet-dip`, `dotnet-dry`, `dotnet-kiss`, `dotnet-yagni`, `dotnet-coupling-cohesion`, `dotnet-separation-of-concerns`, `dotnet-composition-over-inheritance`, `dotnet-naming`, `dotnet-code-smells`, `dotnet-complexity`, `dotnet-ddd`, `dotnet-di`, `dotnet-config`, `dotnet-async`, `dotnet-error-handling`, `dotnet-logging`, `dotnet-testing`, `dotnet-records`, `dotnet-tracing-metrics`, `dotnet-nrt`, `dotnet-linq`, `dotnet-minimal-apis`.

If the user already stated what they want (e.g., "evaluate my DI setup") in the same message that triggered this menu, skip straight to that topic and context instead of showing the menu.

This menu can also be reached mid-conversation — if the user says "back to menu," "start over," or similar while working through any of the topic skills, treat that as an invocation of this skill and show the menu again.
