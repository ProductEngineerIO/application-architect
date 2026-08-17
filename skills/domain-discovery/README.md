# Domain Discovery Interview

This skill (`SKILL.md`) runs a multi-phase, stop-and-resume interview that
elicits a domain model from stakeholders who understand the business but
don't think in schemas. It's for situations where you need to reverse-engineer
what data/entities actually exist for an unfamiliar domain — from
conversations, meeting notes, and event storming — rather than from an
existing database schema.

The core problem it solves: someone new to a domain can't yet tell a real
entity from a synonym, or a confirmed fact from something they overheard
once. The skill enforces a confidence-tagging discipline (Confirmed /
Reported / Assumed) throughout, so the resulting model stays trustworthy even
while incomplete, instead of looking finished while secretly being guesswork.

## How it works

Each discovery effort gets its own folder, `discovery/<slug>/`, so a session
can be paused and resumed later — potentially across many separate
conversations with no shared memory. On every invocation, the skill checks
for an existing `discovery/<slug>/state.md` and either resumes from where
that session left off or bootstraps a new folder from the templates in
`assets/`.

Work then proceeds through six phases (looping back between them as needed):

0. **Passive term harvesting** — build a raw glossary of domain nouns/verbs
   from meetings, docs, and transcripts, before modeling anything.
1. **Three-question entity interview** — turn glossary terms into candidate
   entities by asking where each is created, what identifies it, and whether
   another part of the org uses the same word to mean something different.
2. **Facilitated event storming** — walk a real business scenario
   chronologically to surface entities/events directly from a narrative.
3. **Confidence audit** — adversarially double-check that every tag is
   honest (e.g. downgrade anything marked Confirmed that was never actually
   checked against a system).
4. **Draft narrow, then widen** — bound scope to what's needed for the
   current initiative; park everything else instead of deleting it.
5. **Close the loop against real systems** — verify Reported/Assumed items
   against real schemas or system owners, and assign each in-scope
   attribute a feasibility rating (🟢 Green / 🟡 Yellow / 🔴 Red) in
   `field-definitions.yaml`.

`capability-map.md` isn't filled in phase-by-phase like the rest — it's
generated/refreshed from the other artifacts and is the polished,
plain-language output that PO/UX/BA readers (not just engineers) actually
read.

## Assets

The `assets/` folder holds the templates copied into a new
`discovery/<slug>/` folder the first time a discovery session runs for an
initiative:

| Template | Copied to | Purpose |
| --- | --- | --- |
| `state-template.md` | `state.md` | Tracks session progress: current phase, what's been captured, open questions, and the "resume here" note for the next session. |
| `glossary-template.md` | `glossary.md` | Raw list of domain terms harvested in Phase 0, each tagged with a confidence level and who said it. |
| `entity-tracker-template.md` | `entity-tracker.md` | One row/section per candidate entity, recording answers to the three-question interview (creation point, identity, naming conflicts) with independent confidence tags. |
| `erd-template.mmd` | `erd.mmd` | A Mermaid ER diagram capturing entities and relationships as they're discovered, confidence-tagged. |
| `field-definitions-template.yaml` | `field-definitions.yaml` | Feasibility-rated field definitions (Green/Yellow/Red, effort estimates) meant to merge into an existing central field-definition schema. |
| `capability-map-template.md` | `capability-map.md` | Plain-language, capability-per-row summary generated from the other artifacts for non-technical stakeholders. |

Every artifact except `capability-map.md` is raw material filled in
phase-by-phase; `capability-map.md` is the derived, polished output.

## Using this skill

1. Invoke the skill and tell it what initiative the discovery is for (e.g.
   "advisor onboarding"); it derives a slug like `advisor-onboarding`.
2. If `discovery/advisor-onboarding/state.md` already exists, the skill
   summarizes where the session left off and asks whether to resume there or
   jump elsewhere — it will not restart from Phase 0 automatically.
3. If it doesn't exist, the skill creates `discovery/advisor-onboarding/`
   from the templates above and starts at Phase 0.
4. Work through the phases conversationally — supplying meeting notes,
   answering the three interview questions, walking through a scenario for
   event storming, etc. The skill updates the relevant artifact file(s) after
   any meaningful progress rather than leaving state only in the chat
   transcript.
5. Ask to see current state at any time; the skill reads artifacts back from
   disk (rendering `erd.mmd` as an inline Mermaid diagram) rather than
   re-deriving them from memory.
6. Before ending a session, the skill updates `state.md` with the current
   phase and next concrete action, and gives a short summary of what was
   captured — so the next session (by you or anyone else) can resume cleanly.
