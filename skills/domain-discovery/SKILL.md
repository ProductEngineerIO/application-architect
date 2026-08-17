---
name: domain-discovery-interview
description: Runs a multi-phase, stop-and-resume interview to discover an unfamiliar business domain and build a capability map (glossary, entity interview tracker, confidence-tagged ERD, and a feasibility-rated field-definition YAML). Use this whenever the user wants to figure out what data/entities actually exist for a domain they're new to, wants to build or extend an ERD from stakeholder conversations rather than an existing schema, mentions terms like "capability map," "data feasibility," "event storming," "domain discovery," or says they're new to a codebase/org and need to reverse-engineer the domain model from what people tell them. Also use to resume a discovery session already in progress — check for an existing discovery/ folder first. Push toward this skill even if the user just says "help me understand this domain" or "I don't know what data exists for X" without naming the artifacts explicitly.
---

# Domain Discovery Interview

Elicits a domain model from stakeholders who know the business but don't think
in schemas, one phase at a time, and turns it into artifacts an architect can
actually use: a glossary, an entity interview tracker, a confidence-tagged ERD,
and a feasibility-rated field-definition YAML.

The core problem this solves: a newcomer (or anyone facing an unfamiliar
sub-domain) can't yet distinguish a real entity from a synonym, or a confirmed
fact from something they overheard once. This skill enforces that discipline
so the resulting model is trustworthy even when incomplete, rather than
looking finished while secretly being guesswork.

## Session model: stop and resume

Every discovery effort lives in its own folder so it can be paused and picked
up later, potentially across many separate conversations.

**On every invocation of this skill:**

1. Ask the user (if not already stated) which initiative this is for, and
   derive a slug, e.g. "advisor onboarding" → `advisor-onboarding`.
2. Look for `discovery/<slug>/state.md`.
   - **If it exists**, read it, summarize where the session left off (current
     phase, what's captured, open questions), and ask the user whether to
     resume there or jump elsewhere. Do not restart from Phase 0.
   - **If it doesn't exist**, create the folder from the templates in
     `assets/` (see Setup below) and start at Phase 0.
3. After any meaningful progress in a phase (new terms captured, an entity
   interview completed, a confidence tag changed), update `state.md` and the
   relevant artifact file(s) before ending the turn. Never let progress live
   only in the chat transcript — the whole point is that a conversation can
   end mid-phase and resume cleanly later, potentially by a different
   session that has no memory of this one.

### Setup (first run for an initiative)

```
discovery/<slug>/
├── state.md                  (copy from assets/state-template.md)
├── glossary.md                (copy from assets/glossary-template.md)
├── entity-tracker.md          (copy from assets/entity-tracker-template.md)
├── erd.mmd                    (copy from assets/erd-template.mmd)
├── field-definitions.yaml     (copy from assets/field-definitions-template.yaml)
└── capability-map.md          (copy from assets/capability-map-template.md)
```

`capability-map.md` is the odd one out: it's not filled in phase-by-phase like
the others, it's *generated/refreshed* from the other artifacts (see
"Generating the capability map" below). Everything else is raw material;
this is the polished output non-technical stakeholders actually read.

Fill in the initiative name and today's date in `state.md` when creating it.

## Confidence tagging (applies everywhere)

Every entity, attribute, and relationship gets exactly one tag. This is not
optional and it is the mechanism that keeps the model honest:

- **Confirmed** — verified against an actual schema, API contract, or system
  you or the user has directly inspected.
- **Reported** — a stakeholder said it in conversation; not yet verified
  against a system.
- **Assumed** — inferred from context; nobody has confirmed or denied it.

Tags travel with the data across all artifacts (glossary, tracker, ERD,
YAML). When a tag is upgraded (e.g. Reported → Confirmed after someone checks
a schema), update it everywhere it appears, not just in one file.

## Cascading updates to downstream artifacts

Most phases only update the artifact they're scoped to — Phase 0/1/2 mainly
touch `glossary.md`, `entity-tracker.md`, and `erd.mmd`. Nothing about
editing those files automatically fixes `field-definitions.yaml` or
`capability-map.md`, which are only produced/refreshed in Phase 5 and by the
capability-map generation step, respectively. If new or different
information surfaces about an entity/attribute that already has a
feasibility rating or a capability-map row, that downstream data is now
stale and will not correct itself just because the tracker changed.

Whenever this happens:

1. Say so immediately and add the item to a "Needs re-verification" list in
   `state.md` — don't silently patch the rating and don't wait for someone
   to notice later.
2. In `field-definitions.yaml`, don't overwrite the existing rating outright.
   Add a `needs_reverification: true` (or equivalent) flag next to it along
   with a short reason (e.g. "Phase 2 event storming revealed this is
   created differently than assumed") so the last confirmed rating isn't
   lost, only marked suspect.
3. Treat a flagged item the same as any other unresolved Phase 5 item — it
   gets re-verified against the real system before the rating is trusted
   again, not silently re-guessed.

This is what keeps `capability-map.md`'s "last verified" dates honest: a
flagged item should surface there for exactly this reason rather than
quietly carrying a stale rating forward.

## The phases

Work through these roughly in order, but it's fine to loop back — e.g. a
Phase 2 event-storming session often surfaces new terms that send you back to
Phase 1 interviews. Track current phase in `state.md`.

### Phase 0 — Passive term harvesting

Goal: build a raw glossary of domain nouns and verbs before modeling anything.

- Ask the user what meetings, docs, or transcripts they have access to (or
  paste in) where domain terms come up naturally — UX/PO syncs, requirements
  docs, Slack threads.
- Extract every recurring noun and verb used without definition, plus the
  sentence it appeared in and who said it. Do not attempt to structure or
  relate these yet.
- Append each to `glossary.md` under "Raw terms," tagged **Reported** by
  default (or **Assumed** if the user is paraphrasing from memory rather than
  quoting).
- Exit condition: the user feels they have a representative list, or new
  meetings stop producing new terms.

### Phase 1 — Three-question entity interview

Goal: turn glossary terms into candidate entities via structured questions
the user can literally take into a conversation with a stakeholder.

For each term that recurs or seems central, walk the user through asking (or
role-play answering, if the user is telling you what a stakeholder already
said):

1. **Where does this get created?** (What system/event brings one into
   existence?)
2. **What uniquely identifies one?** (Business key — doesn't need to be a DB
   primary key yet.)
3. **Does anyone else in the org use this same word for something
   different?** — this is the highest-value question; it surfaces bounded
   context mismatches (e.g. "Plan" meaning different things to a
   sales-facing system vs. a recordkeeping engine) before they cause damage
   downstream.

Record answers in `entity-tracker.md`, one row/section per entity, with a
confidence tag on each answer independently — it's common for the identity
question to be Confirmed while the "creation point" is still Reported.

If question 3 surfaces a same-word/different-meaning conflict, capture both
meanings as separate entities with a disambiguating qualifier (e.g.
`Plan (Sales)` vs `Plan (Recordkeeping)`) — never silently merge them.

### Phase 2 — Facilitated event storming

Goal: get entities and events directly from a narrative walkthrough with
mixed technical/non-technical stakeholders, which also happens to be the
fastest way to get UX, PO, and technical people speaking the same vocabulary
in one room.

- Ask the user for a real business scenario relevant to the current
  initiative (e.g. "advisor onboards a plan sponsor").
- Walk it chronologically. At each step, ask: what event just happened, what
  entity did it act on or create, what changed?
- Feed new entities/relationships into `entity-tracker.md` and `erd.mmd`.
  Default new items from this phase to **Reported** unless the narrator is
  directly quoting a system behavior they've seen, not just described.
- If a discovery in this phase touches an entity/attribute that already has
  a feasibility rating in `field-definitions.yaml` or a row in
  `capability-map.md`, follow "Cascading updates to downstream artifacts"
  above rather than leaving that rating stale.
- If the user is running this live with stakeholders in the room, offer to
  produce a lightweight recap they can share back to the group afterward.

### Phase 3 — Confidence audit

Goal: sanity-check that tags are honest, not optimistic.

Go through `entity-tracker.md` and `erd.mmd` and ask the user, per item:
"How do you actually know this — did you see the schema, or did someone tell
you?" Downgrade anything that's been tagged Confirmed without a real system
check. This phase is intentionally adversarial toward the user's own model;
its job is to prevent silent drift into fiction.

### Phase 4 — Draft narrow, then widen

Goal: keep scope bounded to what's actually useful now.

- Confirm with the user which entities are in-scope for the *current*
  initiative. Mark everything else in `entity-tracker.md` as "parked" rather
  than deleting it — it's likely useful for the next initiative.
- Only build out `erd.mmd` and `field-definitions.yaml` fully for in-scope
  entities. Parked entities stay as stub rows in the tracker.
- When a new initiative starts and touches a parked entity, that's the
  signal to widen scope — reopen this session rather than starting fresh.

### Phase 5 — Close the loop against real systems

Goal: convert Reported/Assumed items to Confirmed, and produce the
feasibility-rated YAML that plugs into the field-definition schema.

- For each Reported entity, ask the user whether they can get read access or
  30 minutes with a system owner (DBA, platform architect, recordkeeping
  owner) to verify it. If yes, walk through: does the field exist, is it
  nullable/required, what's the actual source system?
- For each in-scope entity/attribute, assign a feasibility rating in
  `field-definitions.yaml`:
  - 🟢 **Green** — sourced, available, known shape. Safe for UX to design
    against freely.
  - 🟡 **Yellow** — exists but needs integration work. Capture a rough
    T-shirt size (days/weeks) in the `effort` field.
  - 🔴 **Red** — doesn't exist or unknown. This should block further
    hi-fi design investment on that data point until a time-boxed spike
    resolves it — say so explicitly to the user.
- This file is meant to merge into or sit alongside the user's existing
  central field-definition schema, so match its shape (nullability,
  required-ness, versioning) rather than inventing a parallel format — ask
  the user to paste their existing schema conventions the first time this
  phase runs if you haven't seen them yet.

## Generating the capability map

Refresh `capability-map.md` whenever: Phase 5 assigns or changes a
feasibility rating, the user explicitly asks to see the capability map, a
session is ending and in-scope entities have unsaved changes, or an in-scope
item has been flagged `needs_reverification` since its rating was last set.

This file is for PO/UX/BA readers with no data background, so:

- One row per **capability** (a plain-language thing someone might design a
  flow around — "advisor's assigned plans," not `advisor_plan_xref.plan_id`),
  not one row per raw field. Roll related fields up into a single capability
  row where they'd be consumed together; split them out only if they have
  different feasibility ratings.
- Pull the plain-language name from `entity-tracker.md`, the rating/effort/
  confidence/source from `field-definitions.yaml`.
- Only include in-scope entities (skip anything parked in Phase 4).
- Every row needs a "last verified" date. Stale ratings are worse than
  missing ones — if a rating hasn't been touched in a while, flag it to the
  user rather than presenting it as current.
- Sort red items to the top, or otherwise make them impossible to miss —
  they're the ones that should stop a design conversation, not the green
  ones.

## Ending a session

Before the conversation ends (or whenever the user wants to stop), always:

1. Update `state.md` with current phase, a one-line "resume here" note, and
   any immediate next action (e.g. "ask the platform architect about the
   Advisor/Plan sync job").
2. Make sure every artifact file reflects everything discussed — don't leave
   captured information only in the chat.
3. Give the user a short summary of what was captured this session and what
   the next concrete step is, not a re-listing of the whole model.

## Presenting artifacts

When the user asks to see current state, or a phase completes, present the
relevant file(s) rather than re-deriving them from memory — read them back
from disk so what's shown matches what's actually saved. For the ERD, render
`erd.mmd` as a Mermaid diagram inline. Use present_files (or equivalent) if
the user wants the actual files, not just an in-chat view.
