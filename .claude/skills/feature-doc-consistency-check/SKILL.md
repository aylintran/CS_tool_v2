---
name: feature-doc-consistency-check
description: Verify that SRS Section 5 (data model + validation rules) ↔ Tech Design Section 3 (Prisma schema) ↔ Zod schemas in `app/schemas/<slug>/` ↔ Tech Design Section 11 (API endpoints) ↔ actual route files in `app/api/v1/<slug>/` are all in sync for one feature. Reports field-level diffs (missing field, type mismatch, validation rule mismatch, endpoint defined in doc but no route file, route file with no doc entry). Use this skill EVERY TIME a user (Tech Lead, BA, QA) asks to "check doc consistency", "verify SRS matches schema", "audit feature docs", "do the docs match the code", "kiểm tra docs đồng bộ", "rà soát feature X", or before bumping a feature to `approved`/`released` status, or as part of weekly hygiene. Output is a single report at `plans/reports/doc-consistency-<feature-slug>-<date>.md` with PASS/FAIL summary + actionable diff list. Out of scope: writing the fixes (delegate to relevant feature-*-author skill).
---

# Feature Doc Consistency Check (v1.0)

This skill is a **read-only audit**. It does not modify docs or code. It produces a diff report so the Tech Lead / BA can decide what to fix.

## When to use

- Before flipping a feature's status to `approved` or `released`.
- After a non-trivial bug fix that touched schema / business rule / endpoint.
- Weekly hygiene (combine with `feature-bugfix-doc-sync`).
- User explicitly asks to verify docs ↔ code sync.

## Inputs

User must provide `<feature-slug>`. Resolve `<phase-slug>` via `grep -l "<feature-slug>" docs/phases/*/00-prd.md` (ask user if 0 or >1 matches).

## Sources of truth (in priority order)

1. **SRS §5** (`docs/phases/<phase-slug>/features/<slug>/01-srs.md`) — field semantics & validation rules (BA's source of truth for WHAT).
2. **Tech Design §3** (`03-tech-design.md`) — Prisma schema (Tech Lead's source of truth for HOW the data is modeled).
3. **Zod schemas** (`app/schemas/<slug>/*.ts`) — runtime source of truth for validation.
4. **Tech Design §11** — API endpoints summary table (1 row per endpoint).
5. **Route files** (`app/api/v1/<slug>/**` or `app/routes/api.*.ts`) — actual code.

If 1↔2 disagree → BA + Tech Lead must reconcile (usually SRS wins for semantics, Tech Design wins for storage details).
If 2↔3 disagree → bug. Code must match Tech Design after migration.
If 4↔5 disagree → bug. Either delete unused route or add missing doc row.

## Procedure

### Step 1 — Extract field inventory from SRS §5

Read the data model table. Record per field: `name | type | required | default | rule`.

### Step 2 — Extract field inventory from Tech Design §3

Parse the Prisma schema block. Record per field: `name | prisma_type | nullable | default | constraints (CHECK / @unique / @index)`.

### Step 3 — Extract field inventory from Zod schemas

Glob `app/schemas/<slug>/*.ts`, read each file, list field name → Zod chain (`.string().email()`, `.number().min(0)`, etc.). Use `grep`/`Read`, not regex parsing — Zod's DSL is too varied.

### Step 4 — Diff fields (3-way)

Build a table:

| Field | SRS §5 | Tech Design §3 | Zod | Status |
|---|---|---|---|---|
| `email` | string, required, email | `String @unique` | `.string().email()` | ✓ |
| `points` | number, required, min 0 | `Int @default(0)` | (missing) | ✗ Zod missing |

Status values: `✓` (all 3 match), `✗ <which mismatches>`, `⚠ partial` (2 of 3 match).

### Step 5 — Endpoint check

1. Read Tech Design §11 endpoints table — record each row's `method + path`.
2. List actual route files matching the feature module: `find app/api/v1/<slug> app/routes -name "*.ts*"` — extract method + path from filename / loader/action exports.
3. Diff:
   - In doc but no file → ⚠ unimplemented (or wrong doc).
   - In file but no doc row → ✗ undocumented endpoint.
   - Method/path mismatch → ✗.

### Step 6 — Validation rule spot-check

For 5 random fields with non-trivial rules in SRS §5 (e.g. `min: 0`, `max: 100`, regex, enum), confirm:
- Prisma has CHECK / `@db.VarChar(N)` / enum match.
- Zod has equivalent `.min()` / `.max()` / `.regex()` / `.enum()`.
Report missing as `✗ rule-not-enforced`.

### Step 7 — Generate report

Output to `plans/reports/doc-consistency-<feature-slug>-<YYYY-MM-DD>.md`:

```markdown
# Doc Consistency — <feature-slug> (<phase-slug>)
Date: <YYYY-MM-DD>
SRS version: <X.Y.Z> | Tech Design version: <X.Y.Z>

## Summary
- Fields checked: N
- ✓ Pass: N
- ✗ Fail: N (blocker)
- ⚠ Warn: N

## Field diffs
<table from Step 4>

## Endpoint diffs
<table from Step 5>

## Validation rule spot-check
<list from Step 6>

## Recommended actions (ordered)
1. <highest-priority fix> — owner: <BA|Tech Lead|FE|BE>, target doc/file
2. ...

## Open questions
- <if any>
```

### Step 8 — Final report to user

- One-line verdict: `PASS (0 fail, N warn)` or `FAIL — N blockers (see report)`.
- Path to the report file.
- Suggest next skill: `feature-srs-author` (if SRS lags), `feature-tech-design-author` (if schema lags), or open an ADR if the conflict is a design decision.

## Anti-patterns

- ❌ Modify docs or code in this skill — it's read-only.
- ❌ Auto-fix mismatches — always defer to the human owner.
- ❌ Treat warnings as blockers — only `✗` is a blocker.
- ❌ Skip Step 5 because "endpoints look fine" — silent endpoint drift is the most common rot.
