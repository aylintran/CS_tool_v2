---
name: feature-adr-author
description: Write an Architecture Decision Record (ADR) using Michael Nygard format (Context · Decision · Status · Consequences) at `docs/_adr/<NNNN>-<slug>.md` with 4-digit monotonic numbering (never reused). Use this skill EVERY TIME a user (Tech Lead, Architect, Senior Engineer) asks to "write an ADR", "document a technical decision", "record an architecture decision", "ghi lại quyết định kiến trúc", "viết ADR", or whenever a cross-cutting decision needs immutable record (DB choice, framework swap, auth strategy, queue choice, library trade-off). ADRs are immutable once `Status: accepted` — supersede via a new ADR with `Status: Supersedes ADR-NNNN`. Tech Design references ADRs by ID. Out of scope: feature-level decisions that belong in Tech Design itself (only cross-feature decisions warrant an ADR).
---

# ADR Author (v1.0)

## When to use

- Cross-cutting decision affecting >1 feature (DB, queue, auth, deploy target, language).
- Decision is hard to reverse (cost > 1 week to undo).
- Decision overrides a default in `docs/code-standards.md`.
- Superseding a prior ADR.

**Do NOT use for**: feature-internal choices (those go in `03-tech-design.md` Section 2 / Section 9).

## Inputs (ask user if missing)

1. **Title** — imperative phrase, ≤80 chars (e.g. "Use Prisma over Drizzle for ORM").
2. **Context** — what forces (technical, business, team) made this decision necessary?
3. **Options considered** — at least 2; for each: pros, cons, why rejected/chosen.
4. **Decision** — the choice, in 1-3 sentences.
5. **Consequences** — what becomes easier; what becomes harder; what migration is implied.
6. **Status** — `proposed` (default for new), `accepted`, `superseded`, `deprecated`.
7. **Supersedes** — ADR-NNNN reference if applicable.

## Procedure

### Step 1 — Pick the next number

Run `ls docs/_adr/ | grep -E '^[0-9]{4}-' | sort | tail -1` to find the highest existing ID. Increment by 1, zero-pad to 4 digits. **Never reuse a number** even if the ADR was deleted.

### Step 2 — Confirm slug

Slug = kebab-case, ≤6 words, derived from the decision verb-object (e.g. `use-prisma-over-drizzle`, not `prisma-decision`).

### Step 3 — Write the file

Path: `docs/_adr/<NNNN>-<slug>.md`. Template:

```markdown
---
adr_id: ADR-<NNNN>
title: <imperative title>
status: proposed
date: <YYYY-MM-DD>
deciders: [<name1>, <name2>]
supersedes: null            # or ADR-NNNN
superseded_by: null         # filled in when this ADR is later replaced
related_features: [<feature-slug>, ...]   # optional cross-link
---

# ADR-<NNNN>: <title>

## Context

<Forces at play. 2-5 sentences. Why is a decision needed NOW? What constraints (perf, team, deadline, cost, compliance) shape the choice?>

## Options considered

### Option A — <name>
- Pros: ...
- Cons: ...

### Option B — <name>
- Pros: ...
- Cons: ...

(Add Option C, D, ... as needed.)

## Decision

We choose **<Option X>**.

<1-3 sentences explaining the rationale. Reference the dominant force from Context.>

## Consequences

### Positive
- <what becomes easier>

### Negative
- <what becomes harder, or what new burden is introduced>

### Neutral / migration
- <what code/docs need to change as a result>
- <which features are affected; cross-link to feature slugs>

## References

- <link to spike report, benchmark, RFC, or external article that informed the decision>
```

### Step 4 — Cross-link

If the decision affects existing features, edit those features' `03-tech-design.md` to add `See ADR-<NNNN>` in the relevant section (Section 2 or Section 9). Don't rewrite the Tech Design — just one reference line.

### Step 5 — Status workflow

- New ADR is `proposed`. The user (deciders) flips to `accepted` after review.
- ADRs in `proposed` can be edited freely.
- ADRs in `accepted` are **immutable**. To change a decision: write a new ADR with `Status: accepted, Supersedes: ADR-<old>` AND update the old ADR's `superseded_by` field (the ONE allowed mutation).

## Anti-patterns

- ❌ Editing an `accepted` ADR's body — supersede instead.
- ❌ Reusing a deleted ADR number.
- ❌ Writing an ADR for a feature-only decision (use Tech Design).
- ❌ Skipping "Options considered" — at least 2 options are mandatory; "we just picked X" without alternatives is not a decision, it's a default.
- ❌ Vague Consequences ("might cause issues") — be concrete or omit.
