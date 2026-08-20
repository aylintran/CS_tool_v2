---
name: feature-design-spec-author
description: Write a lightweight Design Spec for a feature, placed at docs/phases/<phase-slug>/features/<slug>/02-design-spec.md with three sections only — Screen inventory, Component spec (only the 2-3 most important components), and Responsive behavior. The doc bridges Figma (visual source of truth) and FE implementation by capturing component states, props mapping to SRS Section 5 fields, async behavior, and concrete edge cases that Figma cannot express. Use this skill EVERY TIME a user (Designer, UX Engineer, Design Lead) asks to "write a design spec", "document Figma", "convert Figma to spec", "describe UI states", "spec for FE Dev", "spec component states/edge cases", "viết design spec", "tài liệu hoá Figma", "mô tả states UI", "spec cho FE Dev", "describe component behavior from Figma", or whenever an SRS is approved AND Figma is ready and the team needs a written contract for FE before coding. Skill produces a single file. Out of scope (intentionally): accessibility, animation, design tokens, dynamic content sections — those were dropped in v2.4 lightweight format.
---

# Feature Design Spec Author (v2.4)

This skill is for Designers / UX Engineers turning a Figma file plus an approved SRS into a short, behaviour-focused Design Spec that FE Devs can consume directly.

## Core principle

> **Figma is the source of truth for visuals. The Design Spec captures behaviour, states, props mapping, and edge cases that Figma cannot express. Keep it short.**

## When to use this skill

- The feature's SRS (`docs/phases/<phase-slug>/features/<slug>/01-srs.md`) is `approved`.
- The Figma file/frames for the feature exist and have stable node IDs.
- FE Dev needs a written contract for component states + edge cases before coding.
- Tech Lead wants a design contract to reference from Tech Design Section 4 (file structure).

## Required output

Single file: `docs/phases/<phase-slug>/features/<feature-slug>/02-design-spec.md`

Located between `01-srs.md` and `03-tech-design.md` in the feature folder.

## Procedure (REQUIRED ORDER)

### Step 1 — Read SRS
**Resolve `<phase-slug>` first**: if not given, run `grep -l "<slug>" docs/phases/*/00-prd.md` — the matching phase folder is `<phase-slug>`. Ask the user if 0 or >1 matches.

Read `docs/phases/<phase-slug>/features/<slug>/01-srs.md` carefully:
- **Section 3** — business rules (drive what triggers each component state).
- **Section 5** — data model & field semantics (every component prop must map to a field here).
- **Section 11** — user stories / Gherkin AC (each AC must surface as an observable state in some component).

### Step 2 — Read PRD
Read `docs/phases/<phase-slug>/00-prd.md` for phase scope, personas, KPIs — used to prioritise which 2-3 components are "most important".

### Step 3 — Inventory screens via Figma MCP
- Run `get_metadata` on the parent frame referenced in the SRS.
- Build a flat list of screens with stable IDs `S-01`, `S-02`, ... — one per meaningful state of a screen (e.g. `S-01 Default`, `S-02 Empty`, `S-03 Error`).

### Step 4 — Pull design context for components
For each candidate "most important" component (max 3):
- Run `get_screenshot` on the node.
- Run `get_design_context` on the node.
- Note Figma path (e.g. `Components/Cart/UpsellBlock`), variants, naming.

### Step 5 — Write Section 1 (Screen inventory)
List every screen / state with: section, id, figma link/nodeId, 1-2 sentence description.

### Step 6 — Write Section 2 (Component spec)
Pick the **2-3 most important components only**. Document each with:
- Figma path/naming.
- Purpose.
- States (default, loading, error, success, disabled, empty, ...).
- Props — type + which SRS Section 5 field it maps to.
- Behavior — interaction sequence, async flows.
- Edge cases — concrete (e.g. URL > 60 chars → ellipsis at middle, keep 8 prefix + 6 suffix).
- SRS reference — exact Section 3 / Section 5 references.

### Step 7 — Write Section 3 (Responsive)
Table for Mobile / Tablet / Desktop + mobile-specific notes (e.g. Web Share API with Copy fallback).

## Required frontmatter

```yaml
---
feature_slug: <slug>
phase_slug: <phase-slug>
doc_type: design-spec
version: 1.0
status: draft
owner: "@<designer-username>"
reviewers: ["@po", "@ba", "@tech-lead", "@fe-lead", "@qa"]
created_at: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
links:
  jira_epic: <PROJ-XXX>          # optional — omit if no Jira (Git branch `feature/<slug>` is mandatory)
  figma: <URL>
  related_docs:
    - ./01-srs.md
depends_on:
  - ./01-srs.md
consumed_by:
  - ./03-tech-design.md
---
```

`depends_on` is ONLY `./01-srs.md` — user stories are merged into SRS Section 11 in v2+.
`consumed_by` is ONLY `./03-tech-design.md` — test plan was removed in v2.0.

## Required content structure

```markdown
# Design Spec — <Feature Name>

## 1. Screen inventory

- **section**: <feature-slug>
  **id**: S-01 - Wishlist List / Default
  **figma link/nodeId**: https://figma.com/...?node-id=1:1234
  **description**: Default state of wishlist list with at least 1 item.

- **section**: <feature-slug>
  **id**: S-02 - Wishlist List / Empty
  **figma link/nodeId**: https://figma.com/...?node-id=1:1240
  **description**: Empty state — user has no wishlist items yet.

(... one entry per screen / meaningful state)

## 2. Component spec

### 2.1 `WishlistCard`

- **Figma**: `Components/Wishlist/WishlistCard` (variants: Default / Loading / Error)
- **Purpose**: Render one wishlist item; let user remove or add to cart.
- **States**:
  - `default` — product loaded, image visible.
  - `loading` — skeleton while product fetch in flight.
  - `error` — product fetch failed; show retry.
  - `removing` — optimistic strikethrough while delete API in flight.
- **Props** (mapping to SRS Section 5):
  - `productId: string` → SRS §5 `wishlist_item.product_id`
  - `addedAt: Date` → SRS §5 `wishlist_item.added_at`
  - `priceCents: number` → SRS §5 `wishlist_item.price_cents`
- **Behavior**:
  1. On mount, fetch product via TanStack Query.
  2. Click "Remove" → optimistic strikethrough, fire DELETE, on error revert + toast.
- **Edge cases**:
  - Product title > 60 chars → ellipsis at middle, keep 8 prefix + 6 suffix.
  - Image 404 → fallback to `placeholder.svg` (Figma asset `Misc/Placeholder`).
  - Slow network (> 2s) → show skeleton; never empty card.
- **SRS reference**: §3 BR-04 (remove must be optimistic), §5 `wishlist_item`.

(... 1-2 more components, max 3 total)

## 3. Responsive

| Breakpoint | Behavior |
|---|---|
| Mobile (<768px) | 1 column; FAB for "Add" anchored bottom-right. |
| Tablet (768-1024px) | 2 columns; sticky filter bar. |
| Desktop (>1024px) | 3 columns; filter sidebar pinned left. |

Mobile-specific:
- Share button uses Web Share API; fallback to Copy-link toast on browsers without `navigator.share`.
- Long-press on card opens context menu (mobile only).
```

## Writing rules

- **Component spec** documents only the **2-3 most important components**, NOT every component. Reasoning: Figma already covers visuals; the spec adds behaviour/state/edge cases for the components most likely to bite FE.
- Every prop MUST map to an SRS Section 5 field. If you need a prop without a backing field, STOP and ask the BA — do not invent fields.
- Every state MUST reference the SRS business rule (Section 3) that triggers it.
- Edge cases MUST be concrete. Bad: "handle long content gracefully". Good: "URL > 60 chars → ellipsis at middle, keep 8 prefix + 6 suffix".
- Match the user's language. Default English; switch to Vietnamese when the user writes in Vietnamese.
- **Jira optional, Git mandatory**: `links.jira_epic` may be omitted if the team doesn't use Jira. Git branch (`feature/<slug>` for code, `docs/<slug>` for doc-only) is mandatory.

## Anti-patterns (DO NOT)

- ❌ Documenting EVERY component — scope is the 2-3 most important; the rest live in Figma.
- ❌ Inventing a prop without a matching SRS Section 5 field.
- ❌ Vague edge cases ("handle errors gracefully") — be concrete or skip it.
- ❌ Adding accessibility / animation / design-tokens / dynamic-content sections — out of scope in v2.4 lightweight format.
- ❌ Writing the design spec when SRS is not yet `approved`.
- ❌ Using `depends_on` other than `./01-srs.md` or `consumed_by` other than `./03-tech-design.md`.

## After creation

- Set `status: in-review`, request review from `@tech-lead` and `@fe-lead`.
- Once approved, notify the Tech Lead — they read the design spec to drive the component breakdown in Tech Design Section 4 (file structure).
- Notify the FE Lead — they reference Section 2 component specs while implementing.

## Example triggers

- "Write a design spec for the wishlist feature"
- "Document the Figma states for <feature-slug>"
- "Convert Figma to spec for FE Dev"
- "Describe component states and edge cases for cart <feature-slug>"
- "Spec the component states / edge cases for cart <feature-slug>"
- "Document the Figma frames as a spec for FE Dev"
- "Turn Figma into a written design spec"
- "Describe UI states for the wishlist screens"
