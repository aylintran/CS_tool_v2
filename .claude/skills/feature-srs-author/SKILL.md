---
name: feature-srs-author
description: Write or update a Software Requirements Specification (SRS) for a feature, placed at docs/phases/<phase-slug>/features/<slug>/01-srs.md following the standard 11-section format with data model (field semantics + validation rules), state transitions, QA scenarios, and User Stories with Gherkin AC merged inline (Section 11). Use this skill EVERY TIME a user (BA, Solution Architect, Tech Lead) asks to "write an SRS", "create a software spec", "analyze requirements", "turn the PRD into an SRS", "write user stories", "create acceptance criteria", "viết SRS", "phân tích yêu cầu", or whenever a phase PRD exists and detailed technical documentation is needed for Dev/QA — even if they only say "write a spec" or "do business analysis". This skill ensures the SRS includes complete business rules, data model with field semantics + validation rules, testable scenarios, and user stories with Gherkin AC. Schema design (Prisma DDL, indexes, constraints) is OUT of scope — that belongs to the Tech Lead in Tech Design.
---

# Feature SRS Author (v2.1)

This skill helps a Business Analyst (BA) create `01-srs.md` for a feature, following the team's standard format (see `docs/_conventions/WORKFLOW-STANDARD.md`).

The SRS is the **gold standard for business semantics** — Dev and QA both rely on it. In v2 the SRS is the only BA-owned artefact: user stories are merged in as Section 11 (no separate `02-user-stories.md`).

> **v2.1 scope split (2026-05-05):** the BA describes WHAT each field is (name, conceptual type, required, default, business rule). The Tech Lead decides HOW to model it (Prisma schema, CHECK constraints / Zod refines, indexes, FKs, ON DELETE) inside Tech Design Section 3. There is no DB schema section in the SRS anymore.

## When to use this skill

- A phase PRD already exists at `docs/phases/<phase-slug>/00-prd.md` and detailed analysis for one sub-feature is needed.
- User provides raw business rules and needs them structured into a technical document.
- Update an existing SRS (add fields, change a business rule, bump version).
- Add/refine user stories and acceptance criteria inside an existing SRS.

## Required output

Single file: `docs/phases/<phase-slug>/features/<feature-slug>/01-srs.md`

> v2 NOTE: do **not** create `02-user-stories.md` — user stories now live in Section 11 of this SRS.

## Procedure

### Step 1: Read inputs
- **Resolve `<phase-slug>` first**: if the user did not provide it, run `grep -l "<feature-slug>" docs/phases/*/00-prd.md` to find which phase PRD lists this feature in its Section 8 sub-features table. If 0 matches → ask the user. If >1 matches → ask the user to disambiguate.
- READ FIRST: `docs/phases/<phase-slug>/00-prd.md` (the phase PRD that lists this feature in its Section 8 sub-features table). If it does not exist, stop and ask the PO to write the PRD first, or use the `feature-prd-author` skill.
- Confirm the feature slug from the PRD's `features` array / Section 8.
- Read `docs/_conventions/WORKFLOW-STANDARD.md` Section 3.2 to confirm the format.
- If a sample SRS exists in another feature folder, use it as a reference format.
- (No DB choice needed at this step — schema design lives in Tech Design Section 3, owned by the Tech Lead.)

### Step 2: Confirm with the user
- List clarification questions to resolve before writing the SRS (especially the PRD's "Open questions" relevant to this feature).
- DO NOT answer open questions yourself — push them back to the user/PO.

### Step 3: Generate the SRS in 11 sections (REQUIRED ORDER)

```
1.  Purpose
2.  Scope (2.1 In scope / 2.2 Out of scope)
3.  Key business rules (numbered 3.1, 3.2, ...)
4.  Domain model (4.1 entities / 4.2 enums)
5.  Data model & validation rules (table: Field/Type/Required/Default/Rule)
6.  State transitions (allowed / disallowed)
7.  Runtime/behavior contract (input, output, ordering, fallback)
8.  QA scenarios (numbered, minimum 15)
9.  Implementation notes for AI code generation
10. Final implementation assumptions to review
11. User Stories (Gherkin AC, one per story)
```

> The DB schema design (Prisma model, CHECK constraints, indexes, FKs, ON DELETE policies, migration plan) is **NOT** in the SRS. It lives in **Tech Design Section 3 "Database schema (Prisma)"**, owned by the Tech Lead.

### Step 4: Update the phase PRD if needed
- If new feature scope is uncovered, raise it with the PO so PRD Section 4 / Section 8 can be updated.

## Frontmatter for `01-srs.md`

```yaml
---
feature_slug: <slug>
phase_slug: <phase-slug>
doc_type: srs
version: 1.0
status: draft
owner: "@<ba-username>"
reviewers: ["@<po>", "@<tech-lead>", "@<qa-lead>"]
created_at: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
links:
  jira_epic: <from PRD>          # optional — omit if no Jira
  jira_stories: []               # optional — omit if no Jira
  related_docs:
    - ../../phases/<phase-slug>/00-prd.md
depends_on:
  - ../../phases/<phase-slug>/00-prd.md
consumed_by:
  - ./03-tech-design.md
---
```

## SRS writing rules

### Section 3 — Business rules
- Number as `3.1`, `3.2`, ..., with sub-sections `3.1.1` if needed.
- Every rule must be **testable** (a verifying test case can be written).
- Every default value must be stated explicitly.
- Every activation condition must be listed in full.

### Section 5 — Data model & validation rules (BA stops here)
- Use a Markdown table with columns: `Field | Type | Required | Default | Rule`.
- Field names use `snake_case`.
- **Conceptual** types only: `string`, `integer`, `numeric(p,s)`, `boolean`, `enum`, `array<...>`, `timestamptz`.
- "Required: conditional" must state the condition in the Rule column.
- Nested validation (e.g. "when A=true then B must be >= 1") must be explicit.

> **BA scope (Section 5)**: define field NAME, conceptual TYPE, REQUIRED, DEFAULT, and the business RULE.
>
> **NOT BA scope** (these belong to Tech Lead in Tech Design Section 3): DB engine, Prisma model, `@db.Check` syntax, indexes, foreign keys, `ON DELETE` policy, migration files. The BA describes WHAT the field is; the Tech Lead decides HOW to model it.

### Section 6 — State transitions
- List as `state_a -> state_b` with conditions (if any).
- Include a "Disallowed or discouraged" subsection to prevent AI/Dev from over-extending.

### Section 7 — Runtime/behavior
- Must include **input** (source data), **output** (response payload), **ordering** (processing order), **fallback** (when no data).
- Filtering order numbered 1, 2, 3...
- Define explicit conditions for "not a 500 error" (empty result, no recommendation, ...).

### Section 8 — QA scenarios
- Minimum 15 scenarios.
- Cover: happy path, validation errors, edge cases, state transitions, runtime fallback.
- Each scenario is a single short numbered sentence.

### Section 9 — Implementation notes
- Technical hints for Dev (cache strategy, performance budget, idempotency, ...).
- Clear performance targets (p95 < Xms).
- Note Shopify integration touchpoints (metafield namespace, GraphQL Admin API calls, Shopify Function inputs).
- DO NOT write DDL, Prisma models, or migration code here — that lives in Tech Design Section 3.

### Section 10 — Open assumptions
- REQUIRED. Anything the PRD did not state and the BA had to choose → list here for PO/FE confirmation.

### Section 11 — User Stories
Each story uses this template inline:

```markdown
## Story: <STORY-ID> — <Short title>
<!-- STORY-ID = stable identifier: `US-01`, `STORY-01`, or `PROJ-XXX` if Jira. Jira is optional; Git branch (`feature/<slug>`) is mandatory. -->


**As a** <persona>
**I want to** <action>
**So that** <value>

### Acceptance Criteria (Gherkin)
- **Given** <precondition>
  **When** <action>
  **And** <secondary action if any>
  **Then** <expected outcome>
  **And** <secondary outcome if any>

- **Given** <other case (negative test)>
  **When** ...
  **Then** <specific error message>

### Links
- Business rule: [SRS Section 3.x](#3x-...)

### Definition of Done
- [ ] Code merged
- [ ] Unit tests pass
- [ ] AC tests pass (QA)
- [ ] Docs updated if SRS shifted
```

#### Gherkin rules
- Each AC must be verifiable by automated test.
- Negative cases (validation errors) must specify the **exact error message**, not a generic "show error".
- Each rule in SRS Section 5 → at least one negative AC somewhere in Section 11.
- Each story links back to the relevant SRS section anchor.
- **Jira optional, Git mandatory**: Story IDs may be `US-01` / `STORY-01` (non-Jira) or `PROJ-XXX` (Jira). Frontmatter `links.jira_epic` / `jira_stories` may be omitted if no Jira. Git branch (`feature/<slug>`) is mandatory.

## Anti-patterns (DO NOT)

- ❌ Skip "Open assumptions" / "Out of scope" / "Disallowed transitions"
- ❌ Invent business rules not in the PRD
- ❌ Use "any" or unspecific "object" in the Type column
- ❌ Writing Prisma schema or DDL in SRS — that belongs in Tech Design Section 3
- ❌ Decide indexes, FK, ON DELETE policy in the SRS — Tech Lead's call
- ❌ AC like "system works correctly" (not testable)
- ❌ Rename fields between sections of the SRS
- ❌ Write the SRS before the phase PRD is approved
- ❌ Create a separate `02-user-stories.md` file — user stories belong in Section 11

## After creation

1. Tell the user the SRS file is created at `docs/phases/<phase-slug>/features/<slug>/01-srs.md`.
2. List the "Open assumptions" needing PO/FE confirmation.
3. Suggest next step: Tech Lead runs the `feature-tech-design-author` skill to read this SRS (especially Section 5 field semantics) and produce `03-tech-design.md` — including the Prisma schema design in Section 3.

## Example triggers

- "Phase PRD is approved, write the SRS for example-feature-d"
- "Analyze requirements for subscription billing into an SRS"
- "I need a technical spec for the wishlist feature"
- "Add user stories to the example-feature SRS"
- "Update example-feature SRS to add the discount-stacking field"
- "Phase PRD is approved — write the SRS for example-feature"
