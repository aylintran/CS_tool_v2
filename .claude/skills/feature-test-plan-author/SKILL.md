---
name: feature-test-plan-author
description: Write a lightweight, Jira Xray/Zephyr import-ready Test Plan for a feature, placed at docs/phases/<phase-slug>/features/<slug>/04-test-plan.md with 10 sections (Scope, Test approach, Test environment, Test cases, Edge cases & boundary, State transition tests, Non-functional, Regression suite, Exit criteria, Bug severity guideline). Coverage is derived directly from approved upstream docs — SRS §3 (business rules), §5 (data model & validation), §6 (state transitions), §7 (runtime), §8 (QA scenarios), §11 (user stories with Gherkin AC) plus Design Spec §2 (component states) / §3 (responsive) plus Tech Design §3 (Prisma schema constraints), §7 (error handling), §8 (performance budget), §9 (security), §11 (API Contract endpoints summary). Use this skill EVERY TIME a user (QA, QA Lead, Test Engineer) asks to "write a test plan", "create QA test cases", "design regression suite", "QA spec for feature X", "viết test plan", "tạo test cases QA", "thiết kế regression suite", "spec QA cho feature X", "build the test matrix", or whenever SRS + Design Spec + Tech Design are all approved and QA needs to enumerate testable cases before/parallel-with FE/BE implementation. Output is convertible to Jira Xray CSV. Out of scope: detailed accessibility audits (a11y is tested via heuristics in this plan, not specced from Design Spec since v2.4 dropped the a11y section).
---

# Feature Test Plan Author (v2.5)

This skill is for QA / Test Engineers turning approved SRS + Design Spec + Tech Design into a focused, importable Test Plan.

## Core principle

> **Every test must trace back to a specific section in SRS / Design Spec / Tech Design. No "happy path only". No "system works correctly" tests. Concrete, observable, importable.**

## When to use this skill

- The feature's SRS (`docs/phases/<phase-slug>/features/<slug>/01-srs.md`) is `approved`.
- The feature's Design Spec (`docs/phases/<phase-slug>/features/<slug>/02-design-spec.md`) is `approved`.
- The feature's Tech Design (`docs/phases/<phase-slug>/features/<slug>/03-tech-design.md`) is `approved`.
- QA needs to enumerate testable cases before / parallel with FE/BE implementation.
- Test plan can be written in parallel with implementation; test execution waits for code.

## Required output

Single file: `docs/phases/<phase-slug>/features/<feature-slug>/04-test-plan.md`

(There is no `02-user-stories.md` or `05-api-contract.md` in v2 — user stories are in SRS §11, API contract in Tech Design §11.)

## Procedure (REQUIRED ORDER)

### Step 1 — Read input docs (in order)

0. **Resolve `<phase-slug>`**: if not given, run `grep -l "<slug>" docs/phases/*/00-prd.md` — the matching phase folder is `<phase-slug>`. Ask the user if 0 or >1 matches.
1. `docs/phases/<phase-slug>/features/<slug>/01-srs.md` — focus on §3 (business rules), §5 (data model & validation), §6 (state transitions), §7 (runtime), §8 (QA scenarios), §11 (user stories / Gherkin AC).
2. `docs/phases/<phase-slug>/features/<slug>/02-design-spec.md` — focus on §2 (component spec, states), §3 (responsive).
3. `docs/phases/<phase-slug>/features/<slug>/03-tech-design.md` — focus on §3 (Prisma schema constraints), §7 (error handling), §8 (performance budget), §9 (security), §11 (API Contract endpoints summary).
4. `docs/phases/<phase-slug>/00-prd.md` — out-of-scope check (don't write tests for items the PRD declares out of scope).

> Do NOT look for `02-user-stories.md` or `05-api-contract.md` — they were merged into SRS §11 and Tech Design §11 respectively in v2.

### Step 2 — Build a test inventory (counter)

Before writing the file, count what you owe the coverage matrix:

| Source | Item to count | Test count |
|---|---|---|
| SRS §3 | business rules | × 2 (1 positive + 1 negative) |
| SRS §5 | field validations | × 1 boundary each |
| SRS §6 | allowed transitions | × 1 each |
| SRS §6 | disallowed transitions | × 1 each |
| SRS §7 | runtime ordering steps | × 1 each |
| SRS §7 | soft-fail paths | × 1 each |
| SRS §8 | QA scenarios | × 1 each |
| SRS §11 | user story ACs | × 1 each |
| Design Spec §2 | component states | × 1 UI test each |
| Design Spec §3 | breakpoints | × 1 each (mobile / tablet / desktop) |
| Tech Design §3 | DB constraints (CHECK / Zod refines) | × 1 each |
| Tech Design §7 | error codes | × 1 each |
| Tech Design §8 | perf budgets | × 1 each |
| Tech Design §9 | authn/authz rules + multi-tenant boundaries | × 1 each |
| Tech Design §11 | endpoints | × 2 each (1 success 200 + 1 error path) |

Total = sum. Use this to sanity-check Section 4 (Test cases) doesn't drop coverage.

### Step 3 — Write 04-test-plan.md (10 sections)

Use the structure under "Required content structure" below. Each test row should have a stable ID (`TC-001`, `TC-002`, ...) so it can be imported as Jira Xray CSV later.

## Required frontmatter

```yaml
---
feature_slug: <slug>
phase_slug: <phase-slug>
doc_type: test-plan
version: 1.0
status: draft
owner: "@<qa-username>"
reviewers: ["@po", "@ba", "@tech-lead", "@fe-lead", "@be-lead"]
created_at: <YYYY-MM-DD>
last_updated: <YYYY-MM-DD>
links:
  jira_epic: <PROJ-XXX>          # optional — omit if no Jira (Git branch `feature/<slug>` is mandatory)
  related_docs:
    - ./01-srs.md
    - ./02-design-spec.md
    - ./03-tech-design.md
depends_on:
  - ./01-srs.md
  - ./02-design-spec.md
  - ./03-tech-design.md
---
```

`depends_on` lists only the v2 upstream docs. Do NOT add `02-user-stories.md` or `05-api-contract.md` — they don't exist.

## Required content structure

```markdown
# Test Plan — <Feature Name>

## 1. Scope

- **In scope**: list what is tested, mapped to PRD sub-features.
- **Out of scope**: copy from PRD §4 "Out of scope" — assert QA will NOT cover these.
- **Assumptions**: e.g. test data seeded via `pnpm seed:test`, Shopify dev store available.

## 2. Test approach

| Layer | Tool | Purpose |
|---|---|---|
| Unit | vitest | Pure functions, validators (Zod), business rule fns. |
| Integration | vitest + Prisma test DB | Service + repository roundtrips, DB constraints. |
| API | supertest / Playwright API | Endpoints from Tech Design §11 (200 + error). |
| E2E (UI) | Playwright | Critical user journeys from SRS §11 ACs. |
| Performance | k6 | Tech Design §8 budgets. |
| Security | manual + automated | Tech Design §9 authn/authz, multi-tenant. |
| Regression | tagged Playwright suite | Section 8 below. |

## 3. Test environment

- **App**: <repo>@<branch>, Node 20.x, pnpm 9.
- **DB**: Postgres 15, schema migrated to HEAD via `prisma migrate deploy`.
- **Shopify**: dev store `<handle>.myshopify.com`, app installed.
- **Seed**: `pnpm seed:test` (idempotent).
- **CI**: GitHub Actions `test.yml` runs unit + integration on PR.

## 4. Test cases (functional)

Format: import-ready table. Each row = one test (Jira Xray-compatible if used; else just a CSV row). The `Source` column references either the SRS §11 stable Story ID (e.g. `US-01`, `STORY-01`) or the Jira issue key (`PROJ-XXX`) — Jira is optional, Git branch (`feature/<slug>`) is mandatory.

| ID | Title | Source | Priority | Type | Preconditions | Steps | Expected |
|---|---|---|---|---|---|---|---|
| TC-001 | Create campaign with valid payload returns 201 | SRS §3 BR-01, TD §11 POST /campaigns | High | API | Auth shop A | POST `/campaigns` with valid body | 201, body matches Zod `CampaignSchema`, DB row exists |
| TC-002 | Create campaign without `name` returns 400 VALIDATION_ERROR | SRS §5 `campaign.name`, TD §7 | High | API | Auth shop A | POST `/campaigns` body without `name` | 400, error code `VALIDATION_ERROR`, no DB row |
| TC-003 | Activate DRAFT campaign transitions to ACTIVE | SRS §6 transition DRAFT→ACTIVE | High | Integration | Campaign in DRAFT | Call activate service | Status = ACTIVE, audit row written |
| TC-004 | Activate ARCHIVED campaign rejected (disallowed transition) | SRS §6 disallowed | High | Integration | Campaign in ARCHIVED | Call activate service | Throws `INVALID_TRANSITION` |
| ... | | | | | | | |

Group rows by Source section so coverage is auditable. Each business rule, each AC, each endpoint MUST appear.

## 5. Edge cases & boundary tests

From SRS §5 validation rules + Tech Design §3 constraints.

| ID | Field | Boundary | Expected |
|---|---|---|---|
| TC-101 | `campaign.name` length 0 | min boundary | 400 VALIDATION_ERROR |
| TC-102 | `campaign.name` length 1 | min+1 | 201 |
| TC-103 | `campaign.name` length 120 | max | 201 |
| TC-104 | `campaign.name` length 121 | max+1 | 400 |
| TC-105 | `discount.value_pct` = -1 | below min | 400 |
| TC-106 | `discount.value_pct` = 101 | above max | 400 |
| TC-107 | DB CHECK `discount_value_pct BETWEEN 0 AND 100` raw insert | constraint | Postgres `23514` |

## 6. State transition tests

From SRS §6. One row per allowed + one row per disallowed transition.

| ID | From | To | Allowed? | Trigger | Expected |
|---|---|---|---|---|---|
| TC-201 | DRAFT | ACTIVE | yes | activate() | Status flips, audit row |
| TC-202 | ACTIVE | PAUSED | yes | pause() | Status flips |
| TC-203 | PAUSED | ACTIVE | yes | resume() | Status flips |
| TC-204 | ACTIVE | DRAFT | NO | activate→draft attempt | `INVALID_TRANSITION` |
| TC-205 | ARCHIVED | * | NO | any | `INVALID_TRANSITION` |

## 7. Non-functional tests

### 7.1 Performance (Tech Design §8)
| ID | Endpoint | Budget | Tool | Expected |
|---|---|---|---|---|
| TC-301 | GET /campaigns p95 | < 300ms @ 50 RPS | k6 | meet |
| TC-302 | POST /campaigns p95 | < 500ms @ 20 RPS | k6 | meet |

### 7.2 Security (Tech Design §9)
| ID | Scenario | Expected |
|---|---|---|
| TC-401 | Unauthenticated request to /campaigns | 401 |
| TC-402 | Shop A token reads shop B campaign by id | 404 (not 403 — don't leak existence) |
| TC-403 | SQL injection in `?search=' OR 1=1--` | param escaped, normal results |
| TC-404 | XSS payload in `name` field | stored escaped, rendered as text |

### 7.3 Accessibility (heuristic; spec'd here, not from Design Spec)
| ID | Scenario | Expected |
|---|---|---|
| TC-501 | Tab through campaign form | All interactive elements reachable, visible focus ring |
| TC-502 | Screen reader label on primary CTA | aria-label or visible text present |
| TC-503 | Color contrast on error text | ≥ 4.5:1 |

> a11y was dropped from Design Spec in v2.4. QA still tests via WCAG AA heuristics; expected behaviour is the QA team's call, not derived from spec.

### 7.4 Compatibility / Responsive (Design Spec §3)
| ID | Breakpoint | Expected |
|---|---|---|
| TC-601 | Mobile <768px | layout per Design Spec §3 mobile row |
| TC-602 | Tablet 768-1024 | per Design Spec §3 tablet row |
| TC-603 | Desktop >1024 | per Design Spec §3 desktop row |
| TC-604 | Chrome / Safari / Firefox latest | parity |

## 8. Regression suite

Tag the following test IDs with `@regression` — must pass on every release:
- All TC-0xx Priority=High
- All TC-2xx (state transitions)
- All TC-401, TC-402 (multi-tenant boundary)
- Smoke: TC-001, TC-003

CI: `pnpm test --grep @regression` runs on `main` merges.

## 9. Exit criteria

Release blockers (ALL must hold):
- 100% Priority=High test cases passed.
- 0 open Severity-1 / Severity-2 bugs.
- Regression suite (Section 8) green for last 3 CI runs.
- Coverage matrix (Section 1 of this plan + Step 2 inventory) fully accounted for.

## 10. Bug severity guideline

| Severity | Definition | Example |
|---|---|---|
| S1 — Blocker | Data loss, security breach, app cannot start. | Cross-shop data leak; payment double-charge. |
| S2 — Critical | Core flow broken, no workaround. | Cannot create campaign at all. |
| S3 — Major | Important flow broken, workaround exists. | Filter does not persist on reload. |
| S4 — Minor | Cosmetic, edge case, low impact. | Tooltip wording typo. |
```

## Coverage matrix (the contract)

This skill MUST produce tests covering every row below. If any row has 0 tests, the plan is incomplete.

| Source | Item | Test count |
|---|---|---|
| SRS §3 | Each business rule | ≥ 2 (1 pos + 1 neg) |
| SRS §5 | Each field validation | ≥ 1 boundary |
| SRS §6 | Each allowed transition | ≥ 1 |
| SRS §6 | Each disallowed transition | ≥ 1 |
| SRS §7 | Runtime ordering | ≥ 1 per step |
| SRS §7 | Soft-fail | ≥ 1 each |
| SRS §8 | Each QA scenario | 1 |
| SRS §11 | Each user story AC | 1 |
| Design Spec §2 | Each component state | 1 UI test |
| Tech Design §11 | Each endpoint | ≥ 2 (200 + error) |
| Tech Design §7 | Each error code | 1 |

## Writing rules

- Every test ID is stable (`TC-NNN`) — never renumber once published.
- Every test references its source section (`SRS §3 BR-01`, `TD §11 POST /campaigns`, `Design Spec §2 WishlistCard:loading`).
- Expected results are observable: status code, DB row, error code, visible text. Never "system works correctly".
- Negative tests are mandatory — for every business rule, write the failing case too.
- Match the user's language. Default English; switch to Vietnamese when the user writes in Vietnamese.
- **Jira optional, Git mandatory**: `links.jira_epic` may be omitted; the `Source`/`Story` reference may use SRS §11 stable IDs (`US-01`, `STORY-01`) instead of Jira keys. Git branch (`feature/<slug>`) is mandatory.

## Anti-patterns (DO NOT)

- ❌ Test plan missing tests for any business rule in SRS §3 (every BR needs ≥ 2 tests).
- ❌ Expected results like "system works correctly" — not observable, not testable.
- ❌ Skipping negative / boundary / disallowed transition tests.
- ❌ Skipping a11y / security / perf sections (Section 7 must be filled even if minimal).
- ❌ Not linking back to the SRS / Design Spec / Tech Design section in the `Source` column.
- ❌ Skipping soft-fail tests (SRS §7 soft-fail paths must each have a test).
- ❌ Skipping multi-tenancy (cross-shop access must have explicit security tests — TC-402 pattern).
- ❌ Skipping the regression suite (Section 8).
- ❌ Renumber test IDs after publish — break Jira Xray traceability.
- ❌ Reference deleted docs (`02-user-stories.md`, `05-api-contract.md`).

## After creation

- Print the coverage matrix totals (computed in Step 2) and confirm Section 4-7 row counts match.
- List High-priority test IDs (Section 8 regression set) so the team can pre-tag in Jira.
- Mention that Section 4-7 tables are CSV-convertible — Jira Xray accepts: `ID, Title, Priority, Type, Preconditions, Steps, Expected` (drop the `Source` column on export, keep it inside the doc for traceability).
- Note: this is v2.5 — there is no test-plan README sync (the README skill was removed in v2.0).
- Set `status: in-review`, request review from `@tech-lead`, `@fe-lead`, `@be-lead`.

## Example triggers

- "Write a test plan for the <feature-slug> feature"
- "Create QA test cases for example-feature"
- "Design a regression suite for milestone campaigns"
- "Build the test matrix for example feature"
- "Write the QA spec for the example feature"
- "Enumerate test cases for example-feature"
- "Produce an importable Xray test plan for wishlist"
- "Plan QA coverage for the milestone campaign release"
