# Application Architect Toolkit

This repository packages a set of agent skills for use in an application
architect's day-to-day work: helping design, review, and reason about
software systems — from figuring out what a domain even looks like before any
code exists, to reviewing and guiding the .NET code that implements it.

Skills live under [skills/](skills/) and are consumed by agent tooling (e.g.
Claude Code, GitHub Copilot) that supports the Agent Skills format — each
skill is a `SKILL.md` with YAML frontmatter plus any supporting files it
needs.

## Skills

### [domain-discovery](skills/domain-discovery/SKILL.md)

A multi-phase, stop-and-resume interview that elicits a domain model from
stakeholders who understand the business but don't think in schemas. Use it
when you're new to a domain (or facing an unfamiliar sub-domain) and need to
figure out what entities/data actually exist — from meeting notes,
stakeholder conversations, and event storming — rather than from an existing
database schema.

It builds toward a capability map an architect can actually use: a glossary,
an entity interview tracker, a confidence-tagged ERD, and a feasibility-rated
field-definition YAML. Every fact captured is tagged **Confirmed**,
**Reported**, or **Assumed** so the resulting model stays honest even while
incomplete. See [skills/domain-discovery/README.md](skills/domain-discovery/README.md)
for the full phase breakdown and asset reference.

### [dotnet-engineer](skills/dotnet-engineer/SKILL.md)

A toolkit of 40+ skills covering software architecture guidance for .NET
codebases — general engineering principles (SOLID, DRY, KISS, YAGNI,
coupling/cohesion, DDD) plus .NET-specific practices (Dependency Injection,
Async/Await, Messaging, Data Access, Error Handling, Logging, Concurrency,
Testing, CQRS/MediatR, API Versioning, and more), organized into tiers from
foundational to advanced.

Each topic skill supports four contexts — **Evaluate** an actual codebase,
**Review** a diff/commit, **Write/Learn** the pattern with examples, and
**Validate** a story/requirement — plus cross-cutting workflow skills for
full commit review, everyday PR-style review, plain-language business impact
review, and whole-codebase overviews. See
[skills/dotnet-engineer/README.md](skills/dotnet-engineer/README.md) for the
complete skill list, installation, and usage.
