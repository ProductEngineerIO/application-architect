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
  dotnet-idempotency/                   → /dotnet-idempotency      (Idempotency)
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
  dotnet-messaging/                     → /dotnet-messaging        (Messaging & Event-Driven Architecture)
  dotnet-background-services/           → /dotnet-background-services (Background Services)
  dotnet-data-access/                   → /dotnet-data-access      (Data Access & EF Core Patterns)

Tier 2: Critical
  dotnet-error-handling/                → /dotnet-error-handling   (Error Handling & Result Types)
  dotnet-logging/                       → /dotnet-logging          (Structured Logging)
  dotnet-concurrency/                   → /dotnet-concurrency      (Concurrency & Thread Safety)
  dotnet-validation/                    → /dotnet-validation       (Input Validation)
  dotnet-auth/                          → /dotnet-auth             (Authentication & Authorization)

Tier 3: Important
  dotnet-testing/                       → /dotnet-testing          (Testing)
  dotnet-records/                       → /dotnet-records          (Records & Immutability)
  dotnet-tracing-metrics/               → /dotnet-tracing-metrics  (Distributed Tracing & Metrics)
  dotnet-cqrs-mediatr/                  → /dotnet-cqrs-mediatr     (CQRS & MediatR Patterns)
  dotnet-api-versioning/                → /dotnet-api-versioning   (API Versioning & Contract Design)

Tier 4: Quality
  dotnet-nrt/                           → /dotnet-nrt              (Nullable Reference Types)
  dotnet-linq/                          → /dotnet-linq             (LINQ)
  dotnet-minimal-apis/                  → /dotnet-minimal-apis     (Minimal APIs)
  dotnet-feature-flags/                 → /dotnet-feature-flags    (Feature Flags)
  dotnet-caching/                       → /dotnet-caching          (Caching)
  dotnet-performance/                   → /dotnet-performance      (Performance & Memory)

Tools / Workflows (not tiered — cross-cutting, single-purpose)
  dotnet-commit-review/                 → /dotnet-commit-review          (multi-lens commit review)
  dotnet-code-review/                   → /dotnet-code-review            (everyday PR-style review)
  dotnet-business-impact-review/        → /dotnet-business-impact-review (plain-language business impact review)
  dotnet-codebase-overview/             → /dotnet-codebase-overview      (whole-codebase technical + business overview)
```

43 skills total (16 in Tier 0, 23 across Tiers 1-4, 4 Tools/Workflows).

## Installation

Copy all directories directly into `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
cp -r dotnet-architect \
      dotnet-srp dotnet-ocp dotnet-lsp dotnet-isp dotnet-dip dotnet-dry dotnet-kiss dotnet-yagni dotnet-idempotency \
      dotnet-coupling-cohesion dotnet-separation-of-concerns dotnet-composition-over-inheritance \
      dotnet-naming dotnet-code-smells dotnet-complexity dotnet-ddd \
      dotnet-di dotnet-config dotnet-async dotnet-messaging dotnet-background-services dotnet-data-access \
      dotnet-error-handling dotnet-logging dotnet-concurrency dotnet-validation dotnet-auth \
      dotnet-testing dotnet-records dotnet-tracing-metrics dotnet-cqrs-mediatr dotnet-api-versioning \
      dotnet-nrt dotnet-linq dotnet-minimal-apis dotnet-feature-flags dotnet-caching dotnet-performance \
      dotnet-commit-review dotnet-code-review dotnet-business-impact-review dotnet-codebase-overview \
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
"Give this commit a full review across everything it touches"
"Just do a normal review of this PR"
"Explain this commit to my product manager"
"What does this whole codebase actually do?"
"Is this a command or a query?"
"Will this API change break existing consumers?"
"Is this endpoint safe to retry?"
"Do we have any stale feature flags?"
"Is this event safe to publish and consume more than once?"
"Should this check be validation or a domain rule?"
"Will this worker shut down cleanly?"
"Is this cache actually getting invalidated?"
"Is this allocation actually worth optimizing?"
"Is this endpoint actually protected?"
"Should my DbContext really be a singleton?"
```

## Verifying It's Actually Working

Ask Claude directly:
```
What skills are available?
```

You should see all 43 skills listed. If they're not listed, double check they're directly under `~/.claude/skills/` (not nested inside another folder) and that each has a `SKILL.md` with valid YAML frontmatter.

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
| dotnet-idempotency | Whether repeated/retried operations produce the same end state as running once |

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
dotnet-di, dotnet-config, dotnet-async, dotnet-messaging, dotnet-background-services, dotnet-data-access

### Tier 2: Critical
dotnet-error-handling, dotnet-logging, dotnet-concurrency, dotnet-validation, dotnet-auth

### Tier 3: Important
dotnet-testing, dotnet-records, dotnet-tracing-metrics, dotnet-cqrs-mediatr, dotnet-api-versioning

### Tier 4: Quality
dotnet-nrt, dotnet-linq, dotnet-minimal-apis, dotnet-feature-flags, dotnet-caching, dotnet-performance

### Tools / Workflows (not tiered)

These don't have the Evaluate/Review/Write/Validate contexts the topic skills have — they're single-purpose commands that always start by getting a diff (paste / last-10-commits / hash, same flow as every topic skill's Review context) and then do one specific job with it.

**dotnet-commit-review** — Scans a diff against the *entire* Tier 0-4 catalog to figure out which areas it touches, explicitly calls out when a commit crosses multiple concerns (e.g. Dependency Injection + Concurrency in the same change), reviews each implicated area for adherence/deviation from that area's best practices, and — the main point of this skill — looks specifically for problems that only exist because two or more areas interact (a new singleton with unprotected shared state, a new invariant with no corresponding failure-path test, etc.), not just a concatenation of single-lens findings.

**dotnet-code-review** — The everyday counterpart to Commit Review: a normal PR-style review of a diff — correctness, edge cases, readability, style consistency, test coverage, whether the change does what it claims. Deliberately shallower and broader than Commit Review; points to it (or a specific topic skill) if something deeper is worth a dedicated look, without turning into that review itself.

**dotnet-business-impact-review** — Reviews a diff and produces a plain-language summary of what changed for the business, not the architecture. Infers business rules from code even when they aren't spelled out (e.g. a new "start date can't be in the past" invariant becomes "campaigns can no longer be backdated"), actively looks for business logic hiding in technical-looking code (magic numbers, hardcoded thresholds), flags anything that plausibly needs business/compliance sign-off, and distinguishes explicit facts from inferred ones so the output can be shared as-is with a non-technical stakeholder.

**dotnet-codebase-overview** — Analyzes an entire codebase (not a diff) and explains what it does, producing a technical summary (architecture style, major components, tech stack, data flow) and a separate business summary (what the system does for the business/users, in plain language, using domain vocabulary). Explicitly lower-confidence on the business side since intent is inferred rather than read directly — flags what's confident vs. inferred vs. genuinely unclear, and suggests specific files/classes where a comment would help future readers (human or AI) understand business intent faster next time.

### dotnet-architect — Menu / entry point
Points to all of the above, organized by tier (and by category within Tier 0), plus the Tools/Workflows entries.
