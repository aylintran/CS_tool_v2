---
name: feature-prd-author
description: Write or update a phase-level Product Requirements Document (PRD) covering multiple features, placing it at docs/phases/<phase-slug>/00-prd.md with standard frontmatter and 8 required sections (the last section maps each sub-feature to its SRS + Tech Design). Use this skill EVERY TIME a user (PO, Product Manager, founder) asks to "write a PRD", "create a product spec", "capture product requirements", "turn raw notes into a PRD", "describe a new phase/feature", "viết PRD", "tạo product spec", or presents an idea/business requirement that has no formal documentation yet — even if they don't say the word "PRD". Includes turning customer meeting notes, support tickets, or OKRs into a standard phase PRD.
---

# Feature PRD Author (v2)

This skill helps a Product Owner (PO) create the `00-prd.md` file for a **phase** (which can contain one or many sub-features), following the team's standard workflow (see `docs/_conventions/WORKFLOW-STANDARD.md`).

> **Scope (v2):** 1 PRD = 1 phase = covers MULTIPLE sub-features. Each sub-feature listed in Section 8 will get its own SRS + Tech Design under `docs/phases/<phase-slug>/features/<feature-slug>/`.

## When to use this skill

- User provides raw notes/ideas and needs a structured phase PRD.
- User asks to write a PRD for a new phase or feature group.
- User updates an existing PRD (bump version, add scope, revise KPIs, add/remove sub-features).

## Required output

**Path**: `docs/phases/<phase-slug>/00-prd.md`

`<phase-slug>` is kebab-case in the form `phase-N-<short-name>` (e.g. `phase-1-example-a`, `phase-2-example-b`). If the user has not provided one, ask them to choose a slug before creating the file.

Each sub-feature inside the phase uses its own kebab-case slug (e.g. `example-feature-d`) — these go into Section 8 and the frontmatter `features` array.

## Procedure

1. **Confirm `phase_slug`** with the user if unclear. Must be kebab-case, prefixed `phase-N-`.
2. **Confirm the list of sub-features** in this phase. Each gets a kebab-case slug.
3. **Check whether `docs/phases/<phase-slug>/` already exists**. If not, create it.
4. **Ask the user for the minimum required information** if not in context:
   - The business problem being solved
   - Goals / expected KPIs
   - Primary user persona
   - Significant constraints (time, technical, legal)
   - The breakdown of sub-features (name + 1-line scope each)
5. **Generate the PRD using the template below**.
6. **Set frontmatter `status: draft`** unless the user specifies otherwise.
7. **List "Open questions"** at the end — every unresolved item lives there. DO NOT make assumptions to fill gaps.

## Required frontmatter

```yaml
---
phase_slug: <phase-slug>
features:
  - <feature-slug-1>
  - <feature-slug-2>
doc_type: prd
version: 1.0
status: draft
owner: "@<po-username>"
reviewers: ["@<ba-username>", "@<tech-lead-username>"]
created_at: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
links:
  jira_epic: <PROJ-XXX or TBD>   # optional — omit this key if no Jira
  related_docs: []
depends_on: []
consumed_by:
  - ../../features/<feature-slug-1>/01-srs.md
  - ../../features/<feature-slug-2>/01-srs.md
---
```

## Required content structure

Use exactly these 8 sections, in this order:

```markdown
# PRD: <Phase name>

## 1. Context & problem

- What business problem are we solving?
- Evidence (metrics, support tickets, user research, feedback)
- Why solve it now?

## 2. Goals

### Business goals

- (Measured by which KPIs? Current baseline? Target?)

### User goals

- (What does the user gain once this phase ships?)

## 3. User personas & use cases

### Primary persona

- Persona name / short description

### Top use cases

1. Use case 1 — scenario, expectation
2. Use case 2
3. Use case 3

## 4. Scope

### In scope

- Item 1
- Item 2

### Out of scope

- (VERY IMPORTANT: explicitly list what is NOT in this phase. BA, Designer, Dev rely on this to avoid scope creep.)

## 5. Constraints & assumptions(options)

### Constraints

- Technical (platform, API limits, ...)
- Legal / compliance
- Time / staffing

### Assumptions to verify

- Assumption 1 (who verifies, when)

## 6. Open questions

- [ ] Question 1 — waiting on whom
- [ ] Question 2 — waiting on whom

## 7. Sub-features

| Feature slug       | Short scope    | SRS path                                   | Tech Design path                                   |
| ------------------ | -------------- | ------------------------------------------ | -------------------------------------------------- |
| `<feature-slug-1>` | one-line scope | `docs/phases/<phase-slug>/features/<feature-slug-1>/01-srs.md` | `docs/phases/<phase-slug>/features/<feature-slug-1>/03-tech-design.md` |
| `<feature-slug-2>` | one-line scope | `docs/phases/<phase-slug>/features/<feature-slug-2>/01-srs.md` | `docs/phases/<phase-slug>/features/<feature-slug-2>/03-tech-design.md` |
```

## Writing rules

1. **Do not infer detailed business logic** — that is the BA's job in each sub-feature SRS. The PRD lives at "what" and "why", not "how".
2. **Every KPI must be measurable** — avoid "improve customer experience" without a metric.
3. **"Out of scope" must be specific** — if left blank, AI/Dev will fill the gap incorrectly.
4. **Open questions are the safe haven** — anything unclear belongs there instead of being guessed.
5. **Section 7 is the bridge** — every sub-feature in `frontmatter.features` MUST appear in Section 7 with its expected SRS + Tech Design path.
6. **Language**: match the user's input language (default English; switch to Vietnamese only if the user writes in Vietnamese).
7. **Jira optional, Git mandatory**: `links.jira_epic` may be omitted entirely if the team doesn't use Jira. Git branch convention (`docs/<phase-slug>` for the PRD work) is mandatory.

## After creation

1. Tell the user where the file was created (`docs/phases/<phase-slug>/00-prd.md`).
2. List the "Open questions" so the user can assign them.
3. Suggest the next step: BA runs the `feature-srs-author` skill **for each sub-feature listed in Section 7** to generate `docs/phases/<phase-slug>/features/<feature-slug>/01-srs.md`.
4. If the team uses Jira and the PRD is just approved (`status: approved`), remind the user to create the Jira Epic and update `links.jira_epic`. **Jira is optional** — skip this if the team doesn't use it. Git branch (`feature/<slug>` or `docs/<slug>`) is mandatory regardless.

## Example triggers

- "Plan a new phase covering example campaigns — write the PRD"
- "Here are the meeting notes with our enterprise client, turn them into a phase PRD"
- "Need to document requirements for the example feature phase"
- "Update the phase-1 PRD to add multi-currency"
- "I have this idea for a new phase: ... — capture it as a proper doc"
- "I want to start a new phase for <feature-slug> — write the PRD"

## Anti-patterns (DO NOT)

- ❌ Write technical design / API spec inside the PRD
- ❌ Write detailed user stories with Gherkin (that belongs in each sub-feature SRS Section 12)
- ❌ Place the file anywhere other than `docs/phases/<phase-slug>/00-prd.md`
- ❌ Leave "Out of scope" empty
- ❌ Invent KPIs the user has not confirmed
- ❌ List a feature in `frontmatter.features` but omit it from Section 8 (or vice versa)
