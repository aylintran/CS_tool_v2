---
name: feature-frontend-impl
description: Implement Frontend code for a feature based on the phase PRD, SRS, Design Spec (component states/edge cases) at `02-design-spec.md`, and Tech Design (whose Section 2 + Section 4 list the route files and Tech Design Section 11 lists endpoints — lightweight contract — as a 1-row-per-endpoint summary table that the FE consumes) plus the Figma file (via MCP). Default stack is React Router 7 + TypeScript + Shopify Polaris + App Bridge + TanStack Query for the admin app, and Preact for the storefront extension. Code lives under app/modules/<slug>/, file structure follows the Tech Design, Zod schemas match the SRS data model, and unit tests are written alongside. Use this skill EVERY TIME a user (FE Dev) asks to "code/implement/build feature X", "write a React component for ...", "implement the UI for ...", "convert Figma into code", "create a form per SRS", "viết React component cho ...", "triển khai UI cho ...", or any FE task in a feature whose docs are complete. The skill enforces reading docs in the right order, never inferring business logic, and adhering to the repo's naming conventions.
---

# Feature Frontend Implementation (v2)

This skill is a guideline for FE Devs (or AI tools like Claude Code/Codex) implementing a feature whose docs are complete.

## Core principle

> **Docs are the single source of truth. Do not infer business logic. Do not rename fields. When docs conflict, stop, ask the BA, do not choose for them.**

## Tech assumptions (Shopify app default stack)

- **Admin app**: React Router 7 (SSR) + TypeScript + Shopify Polaris + App Bridge + TanStack Query.
- **Forms**: Zod + react-hook-form + Polaris (`Form`, `Card`, `TextField`, `ChoiceList`, `Select`, `Checkbox`, `BlockStack`, `InlineStack`).
- **Save bar**: `BaseFormStateProvider` (or the repo's equivalent) wired to App Bridge save bar.
- **Resource pickers**: `shopify.resourcePicker({ type: "product" | "collection" })` from App Bridge.
- **Storefront extension**: **Preact** (lightweight, separate compiled bundle) — reads the shop metafield written by the admin (no API roundtrip).
- **File location** (admin): `app/modules/<slug>/` (matches the current repo). Mention `app/features/<slug>/` only if the user's project explicitly uses that convention.
- **Storefront extension location**: `extensions/<slug>-storefront/`.

## When to use this skill

- The feature has `01-srs.md`, `02-design-spec.md`, and `03-tech-design.md`, and the parent phase has `00-prd.md`.
- User asks to code a component/screen/flow.
- Refactor/extend existing FE code per new docs.

## Procedure (REQUIRED ORDER)

### Step 1: Read the docs in priority order

**First, resolve `<phase-slug>`**: if not given, run `grep -l "<slug>" docs/phases/*/00-prd.md` — the matching phase folder is `<phase-slug>`. Ask the user if 0 or >1 matches.

```
1. docs/phases/<phase-slug>/00-prd.md           ← phase scope, KPIs, constraints
2. docs/phases/<phase-slug>/features/<slug>/01-srs.md           ← business rules, data model, validation,
                                              user stories (Section 11 Gherkin AC)
3. docs/phases/<phase-slug>/features/<slug>/02-design-spec.md   ← screen inventory, component states/props
                                              mapping SRS §5, behavior, edge cases, responsive
4. docs/phases/<phase-slug>/features/<slug>/03-tech-design.md   ← file structure, libs, patterns;
                                              endpoints listed in Section 2 (Module breakdown) + Section 4 (File structure)
                                              and — canonically — in Section 11 (API Contract: 1-row-per-endpoint summary table)
5. docs/phases/<phase-slug>/features/<slug>/04-test-plan.md     ← MUST READ Section 4 BEFORE writing unit tests —
                                              match TC-IDs in test names; ensure unit tests
                                              cover all High-priority TC-IDs.
```

> **Reading map**:
> - Design Spec Section 2 (Component spec) → component states + props mapping; guides component implementation (states, behavior, edge cases).
> - SRS Section 5 → field semantics → drives Zod schemas.
> - Tech Design Section 4 → file structure (where each component lives).
> - Tech Design Section 11 (API Contract summary table) → canonical list of endpoints the FE talks to. Use it as the master TODO of TanStack Query hooks to write.
> - Section 2 / Section 4 → context (which module + route file path).
> - Request/response shapes → NOT in Section 11. Source of truth = Zod schemas in `app/schemas/<slug>/` (referenced from Section 4). Import + reuse those types in the FE; do NOT duplicate.
> - **Test Plan Section 4** → list of TC-IDs (functional cases). Every High-priority TC-ID must have a matching unit/integration test; reference the TC-ID in the `it(...)` / `describe(...)` name.

### Step 2: Read Figma via MCP
- Use the frame names referenced in the SRS or Tech Design.
- Extract layout, spacing, color, typography from Figma — DO NOT eyeball them.
- Map Figma tokens to Polaris design tokens where possible (do not invent custom CSS).
- If a Figma frame is missing or its name does not match → stop, report to the Designer.

### Step 3: Read the existing code base
- `glob` to explore the actual repo structure (`app/modules/*`, `app/services/*.server.ts`, `app/schemas/*`, `extensions/*`).
- Read 1-2 existing modules to learn patterns: TanStack Query hooks, API client, Polaris layout, save bar wiring, tests.
- Read `package.json` for exact lib versions (Polaris, App Bridge, react-hook-form, zod, TanStack Query).
- Read `.eslintrc`, `.prettierrc`, `tsconfig.json`.

### Step 4: Plan a TODO list
Before writing the first line of code, list the files to create/modify and reconcile against Tech Design Section 3 (file structure):

```
- [ ] app/modules/<slug>/models/<feature>.types.ts        ← types from SRS Section 4
- [ ] app/schemas/<slug>/<feature>.schema.ts              ← Zod schema from SRS Section 5
- [ ] app/modules/<slug>/hooks/use-<feature>.ts           ← TanStack Query hooks
- [ ] app/modules/<slug>/pages/<page>.tsx                 ← React Router 7 route
- [ ] app/modules/<slug>/components/<component>.tsx       ← Polaris components
- [ ] app/modules/<slug>/components/<component>.test.tsx
- [ ] extensions/<slug>-storefront/src/index.tsx         ← Preact entry (if applicable)
```

### Step 5: Code

#### Type definitions first
- Generate TypeScript types from SRS Section 4 (enums) and Section 5 (data model).
- Field names must **match the SRS exactly** (e.g. `discount_value`, not `discountValue`).
- If the repo uses camelCase at runtime, create a separate mapper — DO NOT rename fields in the DTO.

#### Validation schema (Zod)
- Live in `app/schemas/<slug>/`.
- Match SRS Section 5 100%: required fields, defaults, ranges, conditional rules (`superRefine`).
- For MongoDB projects, the Zod schema also enforces what would have been Postgres CHECK constraints (per SRS Section 8.B).
- Test the schema: unit tests covering every rule in SRS Section 5.

#### API client (TanStack Query)
- Endpoints come from Tech Design **Section 11 (API Contract summary table)** — canonical 1-row-per-endpoint list. Section 2 / Section 4 give the route file context.
- For React Router 7 projects, endpoints follow `/api/v1/<resource>/*`.
- Use TanStack Query (`useQuery`, `useMutation`) with proper `queryKey` per shop.
- Error handling per Tech Design Section 6.
- Type-safe: input/output both typed.

#### Components (Polaris)
- One component per file (`PascalCase.tsx`).
- Use Polaris primitives (`Page`, `Card`, `BlockStack`, `InlineStack`, `Form`, `TextField`, `Select`, `ChoiceList`, `Checkbox`, `Button`, `Banner`).
- Forms: react-hook-form + zodResolver, wrapped by `BaseFormStateProvider` for save bar wiring.
- Product/collection pickers: `shopify.resourcePicker({ type: "product" | "collection" })`.
- Every state mentioned in SRS user stories (Section 12) and runtime contract (Section 7) must have corresponding code.
- Loading/Error/Empty states MUST NOT be skipped — use Polaris `SkeletonPage`, `Banner`, `EmptyState`.
- Edge cases (long text, large numbers, missing image) handled per the Figma + SRS.
- Accessibility: aria-label, role, focus management (Polaris components mostly handle this — verify).

#### Storefront delivery (metafield + Preact)
- Admin writes the campaign config to a shop-level metafield (namespace `$app:<slug>`) via the BE service.
- Preact extension reads that metafield directly — no API roundtrip.
- Keep the Preact bundle small (target ≤15KB gzip per Tech Design Section 7).
- Avoid pulling Polaris into the storefront extension (admin-only).

#### Tests (alongside the code)
- Vitest + Testing Library.
- Each component has tests for:
  - Rendering all states
  - Edge cases
  - User interactions (click, type, submit)
  - Validation errors (Zod messages match SRS)
  - Accessibility (axe scan if available)
- Each business rule in SRS Section 3 → at least one test.
- Each Gherkin AC in SRS Section 12 → at least one test.
- Coverage ≥ 80% for complex logic.

### Step 6: Self-review before opening MR
Checklist:
- [ ] Every field name matches the SRS
- [ ] Every enum value matches the SRS
- [ ] Every state in user stories has code
- [ ] File structure matches the tech design (`app/modules/<slug>/`)
- [ ] Polaris components used (no custom CSS where Polaris exists)
- [ ] No leftover console.log/debugger
- [ ] Tests cover every business rule and AC
- [ ] Full a11y attributes
- [ ] Translation keys (i18n) added if the repo uses i18n
- [ ] Storefront extension stays under bundle budget

### Step 7: Commit & MR
- Conventional commits: `feat(example-feature): add CampaignForm component`
- Branch (MANDATORY): `feature/<feature-slug>` or `feature/<feature-slug>-<sub>`
- MR title: `[<STORY-ID>] feat: <summary>` — STORY-ID may be `US-01`, `STORY-01`, or a Jira key (`PROJ-XXX`). Jira is optional; the Git branch is not.
- MR description includes:
  - Story link (Jira if used; else doc anchor in `01-srs.md` Section 11)
  - Links to docs you read
  - Screenshot/recording for any UI change
  - Test coverage report

## MR description block

```markdown
## Docs consumed
- [x] docs/phases/<phase-slug>/00-prd.md (v1.0)
- [x] docs/phases/<phase-slug>/features/<slug>/01-srs.md (v1.0)
- [x] docs/phases/<phase-slug>/features/<slug>/03-tech-design.md (v1.0)
- [x] Figma frame: <slug>/S-02 - Create / Default

## Story: <STORY-ID>
<!-- e.g. US-01, STORY-01, or PROJ-101 if Jira -->

## Doc deviations (if any)
- (none) or list points where docs are wrong → issue opened to update docs
```

## Rules when conflicts arise

| Situation | Action |
|---|---|
| SRS missing a rule | Open a tracking ticket (Jira sub-task if used; else a Git issue or note in `01-srs.md` Open assumptions) "SRS update needed", STOP coding that part |
| Tech Design deviates from SRS | SRS wins; notify Tech Lead to update the Tech Design |
| Figma differs from SRS | Ask the Designer/BA; prefer the SRS |
| Tech Design Section 2 / Section 4 missing an endpoint or field | Notify the Tech Lead, do not add it yourself |
| Existing code base lacks a suitable pattern | Propose a new pattern in the MR description and wait for Tech Lead approval |

## Anti-patterns (DO NOT)

- ❌ Code without reading the full set of docs (PRD + SRS + Design Spec + Tech Design)
- ❌ Implementing component states without checking Design Spec Section 2 — could miss edge cases
- ❌ Rename a field from `snake_case` (SRS) to `camelCase` in the DTO
- ❌ Infer a business rule that is not in the SRS
- ❌ Skip loading/error/empty states
- ❌ Create files outside `app/modules/<slug>/` (admin) or `extensions/<slug>-*/` (storefront) without discussion
- ❌ Add a new dependency without updating the Tech Design
- ❌ Skip tests "because the code is simple"
- ❌ Hardcode text (use i18n if the repo has it)
- ❌ Inline styles (use Polaris tokens / theme)
- ❌ Pull Polaris into the storefront Preact bundle
- ❌ Disable an ESLint rule without an explanatory comment
- ❌ Push code to main; everything goes through MR

## When the feature has Figma MCP

```
Standard sequence:
1. Read SRS Section 12 user stories to know which Figma frame to fetch
2. Use Figma MCP to get the frame
3. Cross-check the frame against the SRS — if different, ask
4. Extract design tokens/layout from Figma → map to Polaris tokens
5. Code the component
6. Visual diff (if a tool exists) against Figma
```

## After MR merge

1. Update `03-tech-design.md` if structure changed (bump version).
2. Notify QA to test on staging.

## Example triggers

- "Code the CampaignForm component per the example-feature docs"
- "Implement the wishlist FE feature"
- "Build the admin UI for subscription billing"
- "Convert Figma S-02 into a React component"
- "Implement the storefront cart widget per the SRS"
- "Code the CampaignForm component following the example-feature docs"
