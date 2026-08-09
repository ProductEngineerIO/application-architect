# NET Architect Toolkit

Claude Code skills for software architecture guidance — general engineering principles (Tier 0) plus .NET-specific practices (Tiers 1-4).

## Structure (Important)

Per Claude Code's skill discovery rules, each skill must be its own directory directly under `~/.claude/skills/`. Subdirectories nested *inside* a skill are treated as supporting files, not separate skills — so these are packaged flat. Directory names are prefixed with `dotnet-` to reduce the chance of colliding with a skill you already have (even the Tier 0 principle skills, despite being language-agnostic in content, since this toolkit is applied in a .NET context).

```
dotnet-architect/                       → /dotnet-architect       (menu / entry point)

Tier 0: Foundational Principles
  SOLID
  dotnet-srp/                           → /dotnet-srp              (Single Responsibility)
  dotnet-ocp/                           → /dotnet-ocp              (Open/Closed)
  dotnet-lsp/                           → /dotnet-lsp              (Liskov Substitution)
  dotnet-isp/                           → /dotnet-isp              (Interface Segregation)
  dotnet-dip/                           → /dotnet-dip              (Dependency Inversion)
  Other Core Principles
  dotnet-dry/                           → /dotnet-dry              (DRY)
  dotnet-kiss/                          → /dotnet-kiss             (KISS)
  dotnet-yagni/                         → /dotnet-yagni            (YAGNI)
  Structural Principles
  dotnet-coupling-cohesion/             → /dotnet-coupling-cohesion (Coupling & Cohesion)
  dotnet-separation-of-concerns/        → /dotnet-separation-of-concerns (Separation of Concerns)
  dotnet-composition-over-inheritance/  → /dotnet-composition-over-inheritance
  Code-Level Quality
  dotnet-naming/                        → /dotnet-naming           (Naming & Readability)
  dotnet-code-smells/                   → /dotnet-code-smells      (Code Smells & Refactoring)
  dotnet-complexity/                    → /dotnet-complexity       (Cyclomatic Complexity)
  Paradigms
  dotnet-ddd/                           → /dotnet-ddd              (Domain-Driven Design)

Tier 1: Foundational
  dotnet-di/                            → /dotnet-di               (Dependency Injection)
  dotnet-config/                        → /dotnet-config           (Configuration & Options)
  dotnet-async/                         → /dotnet-async            (Async/Await)

Tier 2: Critical
  dotnet-error-handling/                → /dotnet-error-handling   (Error Handling & Result Types)
  dotnet-logging/                       → /dotnet-logging          (Structured Logging)

Tier 3: Important
  dotnet-testing/                       → /dotnet-testing          (Testing)
  dotnet-records/                       → /dotnet-records          (Records & Immutability)
  dotnet-tracing-metrics/               → /dotnet-tracing-metrics  (Distributed Tracing & Metrics)

Tier 4: Quality
  dotnet-nrt/                           → /dotnet-nrt              (Nullable Reference Types)
  dotnet-linq/                          → /dotnet-linq             (LINQ)
  dotnet-minimal-apis/                  → /dotnet-minimal-apis     (Minimal APIs)
```

27 skills total (15 in Tier 0, 12 across Tiers 1-4).

## Installation

Copy all directories directly into `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
cp -r dotnet-architect \
      dotnet-srp dotnet-ocp dotnet-lsp dotnet-isp dotnet-dip dotnet-dry dotnet-kiss dotnet-yagni \
      dotnet-coupling-cohesion dotnet-separation-of-concerns dotnet-composition-over-inheritance \
      dotnet-naming dotnet-code-smells dotnet-complexity dotnet-ddd \
      dotnet-di dotnet-config dotnet-async \
      dotnet-error-handling dotnet-logging \
      dotnet-testing dotnet-records dotnet-tracing-metrics \
      dotnet-nrt dotnet-linq dotnet-minimal-apis \
      ~/.claude/skills/
```

Verify:

```bash
ls ~/.claude/skills/
```

Claude Code watches this directory for changes, so no restart should be needed — but if a new skill doesn't show up in autocomplete, restart Claude Code once.

## Namespacing note

These are plain personal skills, not a Claude Code plugin, so there's no guaranteed namespace — if you already have a skill directory with one of these exact names, installing will overwrite it. The `dotnet-` prefix is meant to make that collision unlikely. If you want a guaranteed collision-proof namespace (e.g. `dotnet-architect:srp`), these would need to be repackaged as a Claude Code plugin instead — which trades away portability to other Agent-Skills-compliant tools (GitHub Copilot, Gemini CLI, etc.), since plugin packaging is Claude Code-specific.

## Usage

**Entry point / menu:**
```
/dotnet-architect
```
Shows the available topics (by tier, with Tier 0 split into SOLID / Other Core Principles / Structural Principles / Code-Level Quality / Paradigms) and helps you pick one — just names, not the sub-menu. Picking a topic then shows that topic's own Evaluate/Review/Write/Validate options.

**Direct access to a topic:** any of the `/dotnet-*` commands listed in the structure above.

**Natural language** (Claude auto-invokes the matching skill based on its description):
```
"Evaluate my dependency injection"
"Does this class violate single responsibility?"
"Review this for DRY violations"
"Is this over-engineered?" (KISS)
"Should I build this generically?" (YAGNI)
"How tightly coupled is this?"
"Should this be a value object?" (DDD)
"Review my error handling"
"How's my test coverage?"
```

## Verifying It's Actually Working

Ask Claude directly:
```
What skills are available?
```

You should see all 27 skills listed. If they're not listed, double check they're directly under `~/.claude/skills/` (not nested inside another folder) and that each has a `SKILL.md` with valid YAML frontmatter.

## What Each Skill Covers

Every topic skill has the same four contexts:

- **Evaluate** — defaults to inspecting your actual codebase (not just asking you to self-report) and reports specific findings with a score
- **Review** — lets you paste code, pick from your last 10 git commits, or give a commit hash/range, then reviews the actual diff
- **Write/Learn** — teaches patterns with concrete code examples
- **Validate** — checks whether a story/requirement addresses the right concerns

### Tier 0: Foundational Principles (general, not .NET-specific)

**SOLID**

| Skill | Covers |
|---|---|
| dotnet-srp | One cohesive reason to change per class |
| dotnet-ocp | Extension without modification (avoiding growing switch/if chains) |
| dotnet-lsp | Subtypes safely substitutable for their base type |
| dotnet-isp | Small, role-specific interfaces over fat ones |
| dotnet-dip | High-level modules depending on abstractions, not concrete details (the design principle behind DI) |

**Other Core Principles**

| Skill | Covers |
|---|---|
| dotnet-dry | True duplicated knowledge vs. coincidental similarity |
| dotnet-kiss | Complexity that earns its keep vs. unnecessary cleverness |
| dotnet-yagni | Speculative generality vs. building for confirmed needs |

**Structural Principles**

| Skill | Covers |
|---|---|
| dotnet-coupling-cohesion | How tightly modules depend on each other; how related a class's own responsibilities are |
| dotnet-separation-of-concerns | Business logic, persistence, presentation, and cross-cutting concerns kept distinct |
| dotnet-composition-over-inheritance | Fragile deep hierarchies vs. composed collaborators |

**Code-Level Quality**

| Skill | Covers |
|---|---|
| dotnet-naming | Names that communicate intent accurately and consistently |
| dotnet-code-smells | Triage pass for long methods, feature envy, primitive obsession, data clumps, shotgun surgery — maps findings to the deeper principle skill |
| dotnet-complexity | Branching/path count, nesting depth, testability impact |

**Paradigms**

| Skill | Covers |
|---|---|
| dotnet-ddd | Bounded contexts, aggregates, entities vs. value objects, ubiquitous language, domain events |

### Tier 1: Foundational (.NET-specific)
dotnet-di, dotnet-config, dotnet-async

### Tier 2: Critical
dotnet-error-handling, dotnet-logging

### Tier 3: Important
dotnet-testing, dotnet-records, dotnet-tracing-metrics

### Tier 4: Quality
dotnet-nrt, dotnet-linq, dotnet-minimal-apis

### dotnet-architect — Menu / entry point
Points to all of the above, organized by tier (and by category within Tier 0).
