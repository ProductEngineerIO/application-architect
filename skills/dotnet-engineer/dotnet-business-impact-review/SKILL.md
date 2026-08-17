---
name: dotnet-business-impact-review
description: Review a git commit or diff and produce a plain-language, non-technical summary of what business rules, behavior, or processes changed — suitable for sharing directly with non-technical stakeholders (product managers, business analysts, compliance). Use when the user wants to explain a code change in business terms rather than technical/architectural terms, including inferring business rule changes from code (new validation constraints, invariants, thresholds) and flagging changes that may need business or compliance sign-off.
---

# Business Impact Review

You are reviewing a commit or diff to explain, in plain language a non-technical stakeholder can read directly, what changed about how the business actually operates — not how the code is architected. The output should be shareable as-is: no class names leading the narrative, no jargon standing in for its business meaning.

This is a workflow skill, not a topic skill — it doesn't have Evaluate/Review/Write/Validate contexts. It always does one thing: take a diff and produce a shareable business-impact summary of it.

## Step 1: Get the diff

If it isn't already clear (e.g. the user already pasted code, or already named a commit/hash), ask:

> How would you like to give me the commit to review?
> A) Paste the diff directly
> B) Pick from your last 10 commits
> C) Give me a commit hash or range

- If **B**: run exactly `git log -10 --oneline` (add `-- <path>` only if the user already narrowed it to a project/service). Do not run any other git commands yet — just show the resulting list with numbers and stop to let them pick. Only after they pick a specific commit, run `git show <hash>` or `git diff <hash>^ <hash>` to get that one diff.
- If **C**: take the hash/range they give and run `git show <hash>` (single commit) or `git diff <hash1> <hash2>` (range) to get the diff.
- If **A**: wait for them to paste it.
- If there's no git repo available (e.g. `git log` fails) or they're not in a repo context, skip straight to asking them to paste the diff.

Also check the commit message (if reviewing via git) and any code comments in the diff — these are legitimate context for inferring intent, and should be used, but see Step 3 on distinguishing confident inference from speculation.

## Step 2: Find business-meaningful changes

Look past the technical mechanics for things that represent a change to business rules, behavior, or process. This includes:

- **New or changed business rules/invariants** — validation constraints, eligibility rules, computed values, thresholds, limits. This is the core skill here: infer the business rule from the code even when it isn't spelled out. For example, a new invariant that a `MarketingCampaign`'s start date cannot be set in the past isn't just a validation check — it's a business rule that campaigns can no longer be backdated, and a non-technical reader should be told exactly that, not shown the code that enforces it.
- **Behavior changes visible to users or downstream processes** — anything that changes what a user, customer, or another business process would actually experience or receive.
- **Domain concept changes** — a new concept being introduced, an existing one being renamed/redefined/removed. Use the same ubiquitous-language discipline as `dotnet-ddd` — describe things the way the business already talks about them, not the way the code names them, when the two differ.
- **Hidden business logic in technical-looking code** — actively look for magic numbers, hardcoded thresholds, or validation logic embedded in code that looks purely infrastructural (a config value, a constant, a seemingly-arbitrary conditional). These are easy to miss and often carry real business meaning (e.g. a hardcoded `30` that's actually "customers have 30 days to dispute a charge"). Flag these explicitly as things a technical review would likely skip past.

## Step 3: Be honest about confidence

Distinguish clearly between:
- Business rules you're **confident** about because they're explicit in code, comments, or the commit message
- Business rules you're **inferring** from code structure alone (like the campaign start-date example) — label these as inferred, and say what in the code led you there, so the reader can verify with the right person rather than take it as fact
- Anything genuinely ambiguous — say so rather than guessing

Never assert a business rule as fact when it's actually inferred from ambiguous code.

## Step 4: Write the summary in plain language

- Lead with what changed for the business, not what changed in the code. ("Marketing campaigns can no longer be scheduled with a start date in the past" — not "`MarketingCampaign.Validate()` now checks `StartDate >= DateTime.Today`.")
- You can reference the responsible code element in parentheses or a footnote for traceability, but it should never be the lead of a sentence.
- Avoid technical jargon (don't say "validation," "invariant," "null check" as the primary framing — say what it means for the business: "the system will now reject...", "customers will no longer be able to...").
- Organize by business capability/process affected, not by file or class.

## Step 5: Risk framing

For each business-meaningful change, explicitly flag whether it looks like something that should get business or compliance sign-off before shipping, and briefly say why. For example: "This changes how the early-withdrawal fee is calculated — worth confirming with product/compliance before this merges, since it affects what customers are charged." Don't flag everything as risky by default — reserve this for changes that plausibly need a human business decision, not routine implementation details.

## Navigation

At any natural pause point, offer the user a way to move on. They can say:
- "back to menu" — returns to the `/dotnet-engineer` topic menu
- "also do a technical review" — hands off to `dotnet-commit-review` for the same diff
- "done" / "that's all" — ends here

If the user says any of these, act on it immediately rather than continuing to add more analysis unprompted.
