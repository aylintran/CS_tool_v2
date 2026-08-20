---
name: feature-backend-impl
description: Implement Backend code for a feature based on the phase PRD, SRS, and Tech Design (which contains the Prisma schema as Section 3, module breakdown in Section 2, file structure in Section 4 listing route files, and Tech Design Section 11 lists endpoints — lightweight contract — as a 1-row-per-endpoint summary table) — including Prisma migration, entities, DTO/Zod validation, services, repositories, runtime resolver, and unit/integration tests. Default ORM is Prisma (works with PostgreSQL or MongoDB). Code lives under app/services/ + app/api/v1/<slug>/ (React Router 7) or src/modules/<slug>/ (NestJS) per the Tech Design. Schema matches Tech Design Section 3 (Prisma), field semantics + validation rules match SRS Section 5, runtime behavior matches SRS Section 7. Use this skill EVERY TIME a user (BE Dev) asks to "implement backend for feature X", "write API endpoints", "create migrations", "code the service layer", "build the resolver", "implement backend cho feature X", "viết API endpoints", or any BE task in a feature whose docs are complete. The skill enforces no field renames, no skipping of constraints (CHECK for Postgres / Zod refines for Mongo), no missed business rules, and proper handling of soft-fail cases (empty result is not a 500).
---

# Feature Backend Implementation (v2)

This skill is a guideline for BE Devs (or Claude Code/Codex) implementing the BE of a feature whose docs are complete.

## Core principle

> **The SRS is the business contract (Section 5 = field semantics + validation rules). The Tech Design is the technical contract (Section 3 = Prisma schema; Section 2 + Section 4 = module breakdown + file structure listing endpoints). Copy the Prisma schema verbatim from Tech Design Section 3 (CHECK constraints for Postgres, Zod refines for Mongo). Field names are sacred. Empty/no-result is not a 500 error.**

## Tech assumptions (Shopify app default stack)

- **ORM**: **Prisma** — supports PostgreSQL and MongoDB.
  - Postgres: `npx prisma migrate dev --name <slug>` (produces SQL migrations).
  - MongoDB: `npx prisma db push` (no migration history; document schema diffs in PR).
- **Validation**: **Zod** — schemas live in `app/schemas/<slug>/` (RR7) or `src/modules/<slug>/dto/` (NestJS).
- **Two framework options** (pick the one the repo uses):
  - **React Router 7 monolith**: services in `app/services/<slug>-service.server.ts`; API endpoints in `app/api/v1/<slug>/*.ts` as route handlers.
  - **NestJS backend**: standard module pattern `src/modules/<slug>/{api/controllers, api/dto, domain/services, domain/entities, infrastructure/repositories}`.
- **Shopify integration**: `@shopify/shopify-api` (GraphQL Admin API). Storefront delivery via shop-level metafield (namespace `$app:<slug>`); cart discount via Shopify Function (WASM) where applicable.

## When to use this skill

- The feature has `01-srs.md` and `03-tech-design.md` (whose Section 3 contains the Prisma schema; Section 2 + Section 4 describe the modules and the route files), and the parent phase has `00-prd.md`.
- User asks to code BE: API, service, migration, resolver.
- Refactor/extend BE per a new SRS.

## Procedure (REQUIRED ORDER)

### Step 1: Read docs in order

**First, resolve `<phase-slug>`**: if not given, run `grep -l "<slug>" docs/phases/*/00-prd.md` — the matching phase folder is `<phase-slug>`. Ask the user if 0 or >1 matches.

```
1. docs/phases/<phase-slug>/00-prd.md         ← phase scope, KPIs, constraints
2. docs/phases/<phase-slug>/features/<slug>/01-srs.md         ← ALL sections, especially:
   - Section 3: business rules
   - Section 4: enums
   - Section 5: data model with field semantics + validation rules (BA's source of truth)
   - Section 6: state transitions
   - Section 7: runtime/resolver behavior
   - Section 9: implementation notes (cache, performance, idempotency)
   - Section 11: user stories (AC for integration tests)
3. docs/phases/<phase-slug>/features/<slug>/03-tech-design.md ← file structure, libs, patterns:
   - Section 2: Module breakdown (lists endpoints + route files)
   - Section 3: Prisma schema (Postgres DDL with CHECK, OR Mongo + Zod refines) — COPY VERBATIM
   - Section 4: File/folder structure (route file paths = endpoint paths)
   - Section 10: migration plan
   - Section 11: API Contract — 1-row-per-endpoint summary table (Method · Path · Auth · Description · Story). Canonical list of endpoints to implement.
4. docs/phases/<phase-slug>/features/<slug>/04-test-plan.md   ← MUST READ Section 4 BEFORE writing unit/integration tests —
   match TC-IDs in test names; ensure unit tests cover all High-priority TC-IDs.
```

> **Reading map**:
> - SRS Section 5 → field NAMES, conceptual TYPES, REQUIRED, DEFAULT, business RULES (Zod schema source).
> - Tech Design Section 3 → Prisma model + DB-level constraints (migration source).
> - Tech Design Section 11 → **canonical endpoint list** (lightweight summary table). Use this as the master TODO of routes to ship. Section 2 + Section 4 give context (which module + route file path); Section 11 gives the contract (method + auth + story link).
> - DTOs come from SRS Section 5 + Zod schemas in `app/schemas/<slug>/` (referenced from Section 4) — NEVER expect schemas inside Section 11.
> - **Test Plan Section 4** → list of TC-IDs (functional cases). Every High-priority TC-ID must have a matching unit/integration test; reference the TC-ID in the test name.

### Step 2: Read the existing code base
- Explore the actual repo structure (`glob`, `grep`).
- Confirm framework: React Router 7 (`app/services/*.server.ts`, `app/api/v1/`) or NestJS (`src/modules/`).
- Read 1-2 existing modules to learn patterns: route handler / controller, service, repository, error class, validation pipe.
- Read how the repo handles: auth (Shopify session), multi-tenancy (`shop_id`), logging, transactions, Prisma client wiring.

### Step 3: Plan a TODO list
Reconcile against Tech Design Section 4 (file structure). Pick **Template A (RR7)** or **Template B (NestJS)** — do not mix.

#### Template A — React Router 7
```
- [ ] prisma/schema/<slug>.prisma                            ← model from Tech Design Section 3
- [ ] (Postgres) prisma/migrations/<timestamp>_create_<slug>/ ← via prisma migrate dev
- [ ] app/schemas/<slug>/<entity>.schema.ts                  ← Zod schemas + enums
- [ ] app/services/<slug>-service.server.ts                  ← business logic
- [ ] app/api/v1/<slug>/create.ts
- [ ] app/api/v1/<slug>/update.ts
- [ ] app/api/v1/<slug>/get.ts
- [ ] app/api/v1/<slug>/list.ts
- [ ] app/api/v1/<slug>/change-status.ts
- [ ] app/services/<slug>-service.test.ts
```

#### Template B — NestJS
```
- [ ] prisma/schema.prisma                                   ← add models
- [ ] (Postgres) src/modules/<slug>/infrastructure/migrations/<timestamp>_create_<slug>/
- [ ] src/modules/<slug>/domain/enums/*.enum.ts
- [ ] src/modules/<slug>/domain/entities/<entity>.entity.ts
- [ ] src/modules/<slug>/api/dto/{create,update}-<entity>.dto.ts
- [ ] src/modules/<slug>/api/controllers/<entity>.controller.ts
- [ ] src/modules/<slug>/domain/services/<entity>.service.ts
- [ ] src/modules/<slug>/domain/services/resolver.service.ts (if a runtime resolver exists)
- [ ] src/modules/<slug>/infrastructure/repositories/<entity>.repository.ts
- [ ] src/modules/<slug>/tests/*.spec.ts
```

### Step 4: Code

#### Prisma schema + migration FIRST
- COPY the Prisma model from **Tech Design Section 3** — do not paraphrase. (Field semantics live in SRS Section 5; the Prisma model itself lives in Tech Design Section 3.)
- **Postgres**: keep every `@db.Check(...)` and the equivalent SQL CHECK constraints from the migration. They are the final validation at the DB layer. Run `npx prisma migrate dev --name create_<slug>` and review the generated SQL.
- **MongoDB**: no CHECK — every constraint that would have been a CHECK lives in the Zod schema (per Tech Design Section 3.B). Run `npx prisma db push` and document the diff in the PR.
- Keep all indexes from Tech Design Section 3.
- For Postgres: hand-write a reversible `down.sql` (Prisma does not produce one). Test up → down → up locally.
- For Mongo: document the rollback steps in the PR description (e.g. drop collection, remove fields).

#### Enums
- Each enum in SRS Section 4 = one file/const + Zod `z.enum(...)`.
- Values match SRS exactly (lowercase with underscores: `cross_sell`, not `crossSell`).

#### Entity / Prisma model
- Field names match SRS Section 5 (`snake_case`, mapped via `@map` if Prisma client uses camelCase).
- Types match: `numeric(12,2)` → `Decimal` (Postgres) / `Float` or string (Mongo), `timestamptz` → `DateTime`.
- Default values match the SRS.

#### DTO + Validation (Zod)
- Schemas in `app/schemas/<slug>/` (RR7) or `src/modules/<slug>/dto/` (NestJS).
- DTO field name = SRS field name.
- Implement EVERY validation rule in SRS Section 5:
  - Required: per the Required column
  - Range/format: per the Rule column
  - Conditional (e.g. "required when X = Y"): use `superRefine`
  - Default: per the Default column
- Whitelist input — Zod `.strict()` rejects unknown fields.
- For Mongo projects: Zod also enforces what would have been Postgres CHECK constraints (per Tech Design Section 3.B).

#### Service layer
- **RR7**: TypeScript class or factory in `app/services/<slug>-service.server.ts` (the `.server.ts` suffix prevents bundling into client code).
- **NestJS**: `@Injectable()` class in `src/modules/<slug>/domain/services/`.
- Implement EVERY business rule from SRS Section 3.
- Implement state transitions from SRS Section 6 — reject disallowed transitions with a clear error code.
- For each rule, add a comment pointing back to the SRS section: `// SRS 3.5: when offer_source = automated_recommendations, ...`
- Idempotency: if the SRS requires it, use an idempotency key.
- Shopify metafield writes: when the SRS requires storefront delivery, write the campaign config to a shop-level metafield (`$app:<slug>` namespace) via `@shopify/shopify-api` GraphQL Admin API.

#### Repository (or Prisma direct call)
- Multi-tenancy: EVERY query filters by `shop_id` — never forget.
- Soft delete: filter `deleted_at: null` unless soft-deleted rows are required.
- Do not expose raw Prisma client outside the layer when in NestJS; in RR7 it's acceptable to call Prisma from the service directly if that's the repo pattern.

#### Runtime resolver (if any)
Implement exactly per SRS Section 7:
- Input per Section 7.1
- Output per Section 7.2
- Filtering ORDER per Section 7.3 (1, 2, 3...)
- Soft-fail per Section 7.4, 7.5 (DO NOT throw 500 on empty/no-rec)
- Performance per SRS Section 9 (cache, p95)

> If the resolver runs **inside a Shopify Function (WASM)**, it lives in `extensions/<slug>-discount/` — code it in the language the function template requires (Rust/JS/TS depending on toolkit). The same SRS Section 7 still applies.

#### Error handling
- Error class per Tech Design Section 7.
- Error code prefix `BR_` for business rules, per Tech Design Section 7 error table.
- Correct HTTP statuses: 422 for business rule violations, 400 for validation, 404 for not found, 401 for unauth, 403 for forbidden.
- Log full context (shop_id, request_id, trace_id) but never log PII.

#### Tests (alongside)
- Unit tests:
  - Every service method, every business-rule branch
  - Every validation rule in SRS Section 5 → 1 positive + 1 negative
  - Every state transition in SRS Section 6
  - Every soft-fail case in SRS Section 7.4, 7.5
- Integration tests:
  - Every endpoint listed in Tech Design Section 2 / Section 4
  - Every error code
  - AuthN/AuthZ
  - Multi-tenancy (cross-shop access is rejected)
  - Every Gherkin AC in SRS Section 11

### Step 5: Self-review checklist

- [ ] Prisma schema matches Tech Design Section 3 100% (CHECK for Postgres, Zod refines for Mongo)
- [ ] Every field name matches SRS Section 5 (and Tech Design Section 3)
- [ ] Every enum value matches SRS Section 4.2
- [ ] Every rule in SRS Section 5 has Zod validation
- [ ] Every state transition in SRS Section 6 has code + tests
- [ ] Resolver SRS Section 7 has correct filtering order and soft-fail
- [ ] Every endpoint matches Tech Design Section 2 / Section 4
- [ ] Every error code is in Tech Design Section 7 error table
- [ ] Multi-tenancy filter is on every repo method
- [ ] Performance budget met (load test if necessary)
- [ ] Migration is reversible (Postgres) or rollback documented (Mongo)
- [ ] Feature flag is set up
- [ ] Shopify metafield write goes to the `$app:<slug>` namespace
- [ ] No PII is logged
- [ ] Test coverage ≥ 80%

### Step 6: Commit & MR
- Conventional commits.
- Standard branch + MR title (see `WORKFLOW-STANDARD.md` Section 5). Branch (`feature/<slug>`) is **MANDATORY**; Jira links in MR title/description are **OPTIONAL** (use SRS §11 stable Story IDs like `US-01` if no Jira).
- MR description block:

```markdown
## Docs consumed
- [x] docs/phases/<phase-slug>/00-prd.md (v1.0)
- [x] docs/phases/<phase-slug>/features/<slug>/01-srs.md (v1.0)
- [x] docs/phases/<phase-slug>/features/<slug>/03-tech-design.md (v1.0)

## Migration plan
- DB: Postgres / MongoDB
- Up: Prisma schema + indexes (see Tech Design Section 3)
- Down: reversible SQL (Postgres) or documented rollback (Mongo)
- Run on: <staging date>

## Performance check
- Resolver p95: <Xms> (target per SRS Section 9)
- Load test report: <link>

## Doc deviations (if any)
- (none) or list points where docs are wrong
```

## Rules when conflicts arise

| Situation | Action |
|---|---|
| SRS missing a validation rule | Open a tracking ticket (Jira sub-task if used; else Git issue / note in `01-srs.md` Open assumptions) "SRS update needed", STOP coding that part |
| Tech Design Section 3 missing a field that exists in SRS Section 5 | Open a tracking ticket (Jira sub-task if used; else Git issue) "Tech Design schema update needed", STOP migration |
| Endpoint listed in Tech Design Section 2 / Section 4 differs from SRS | SRS wins; notify Tech Lead to update the Tech Design |
| Tech Design proposes a pattern unlike the repo | Confirm with Tech Lead before coding |
| Performance budget is unfeasible | Report back to Tech Lead, propose alternatives |

## Anti-patterns (DO NOT)

- ❌ Code without reading the full set of docs (PRD + SRS + Tech Design)
- ❌ Writing schema from imagination — copy from Tech Design Section 3 verbatim
- ❌ Looking for DDL in SRS — it's in Tech Design Section 3 now
- ❌ Drop CHECK constraints (Postgres) or Zod refines (Mongo) because "the service validates"
- ❌ Mix Template A (RR7) and Template B (NestJS) in the same project
- ❌ Rename fields
- ❌ Throw 500 when the resolver returns empty (must soft-fail per SRS 7.4/7.5)
- ❌ Skip the `shop_id` filter in the repo
- ❌ Hard-delete when the schema has `deleted_at`
- ❌ Skip tests with "it's just CRUD"
- ❌ Add caching not specified in the Tech Design
- ❌ Add fields not in the SRS
- ❌ Log PII (raw cart contents, customer info)
- ❌ Non-reversible migrations (Postgres) without an approved exception
- ❌ Write metafields to a public namespace instead of `$app:<slug>`
- ❌ Look for an OpenAPI spec or `05-api-contract.md` — neither exists; endpoints are listed in Tech Design Section 11 (lightweight summary table) plus Section 2 / Section 4 (route files)
- ❌ Expect request/response schemas inside Tech Design Section 11 — they live in `app/schemas/<slug>/` (Zod, source of truth)

## After MR merge

1. Run the migration on staging first (Postgres) or push schema (Mongo).
2. Update Tech Design Section 2 / Section 4 if endpoints changed (bump version).
3. Notify QA + FE.
4. Monitor metrics for 24h after rollout: error rate, p95 latency, soft-fail rate.

## Example triggers

- "Implement the backend for example-feature per the SRS"
- "Code the admin API endpoints"
- "Write the migration for the subscription feature"
- "Implement the runtime resolver"
- "Build the service layer + repository for ..."
- "Implement the backend for example-feature following the SRS"
