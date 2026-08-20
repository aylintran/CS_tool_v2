---
name: feature-docs-sync
description: Sync project-level documentation (system-architecture.md, codebase-summary.md, project-roadmap.md, code-standards.md) after a feature ships to production. Updates affected sections to reflect new modules, schemas, endpoints, conventions. No new file produced — modifies existing docs in `/docs/`. Use this skill EVERY TIME a user (Tech Lead, Doc Maintainer) asks to "sync project docs", "update architecture doc after feature ships", "cập nhật system-architecture", "update roadmap", "đồng bộ docs sau release", or after a feature reaches `released` status and any of system-architecture / codebase-summary / project-roadmap / code-standards needs reflecting the change.
---

# Feature Docs Sync

## 1. When to use

Activate this skill when:

- A feature reaches `released` / `shipped` status (flag at 100%, post-launch checklist clean).
- Project-level docs in `docs/` are out of date relative to the shipped feature.
- A user asks to "sync project docs", "update architecture doc after feature ships",
  "cập nhật system-architecture", "update roadmap", "đồng bộ docs sau release", etc.

**Do NOT** activate when:
- Feature is still behind a flag at <100% — defer until GA.
- For feature-folder docs (01-srs, 02-design-spec, 03-tech-design, 04-test-plan, 05-release-plan) — those are owned by their authoring skills.

## 2. No file output — only EDITS

This skill produces **no new file**. It edits the following project-level docs:

| File | What to update |
|---|---|
| `docs/system-architecture.md` | New modules / services / integration points / cross-service flows added by the feature. |
| `docs/codebase-summary.md` | New directories under `app/modules/<slug>/`, `app/services/`, `app/api/v1/<slug>/`, `prisma/schema/`. New top-level files of note. |
| `docs/project-roadmap.md` | Mark the feature as **shipped YYYY-MM-DD**. Add follow-up items (flag cleanup, tech debt, next iteration). |
| `docs/code-standards.md` | **Only if** the feature introduced a new pattern/convention worth standardising (new lint rule, new file structure, new util pattern). Skip otherwise. |

## 3. Procedure

1. **Read the shipped feature's docs** (resolve `<phase-slug>` first via `grep -l "<slug>" docs/phases/*/00-prd.md`):
   - `docs/phases/<phase-slug>/features/<slug>/01-srs.md` — what user-facing capability shipped.
   - `docs/phases/<phase-slug>/features/<slug>/03-tech-design.md` — Section 2 (modules), Section 3 (schema), Section 4 (file structure), Section 11 (endpoints).
   - `docs/phases/<phase-slug>/features/<slug>/05-release-plan.md` — confirm `status: shipped` + ship date.

2. **Diff against current project-level docs** — for each of the 4 target files, find the section(s) that mention modules/schemas/endpoints in the same area as the new feature.

3. **Identify minimal sections to update**:
   - System Architecture: usually 1 module diagram + 1-2 paragraphs.
   - Codebase Summary: usually 1 row in the directory table + 1 entry per new top-level area.
   - Roadmap: 1 line under "Shipped" + maybe new follow-ups under "Backlog".
   - Code Standards: skip unless a new pattern is established.

4. **Make minimal surgical edits** — append to existing tables/sections, don't rewrite. Preserve existing wording for unchanged areas.

5. **Add shipped marker** in `docs/project-roadmap.md`:
   ```markdown
   - [x] **<feature-slug>** — shipped 2026-MM-DD. Phase: <phase-slug>. Owner: <tech-lead>.
   ```

6. **Bump `last_updated`** in frontmatter of each file you edited.

## 4. What to write where

### system-architecture.md
- New module: add to component diagram (Mermaid or ASCII).
- New external integration (Shopify webhook, 3rd-party API): add to integration list.
- New data flow: add a 2-3 sentence description, not a re-explanation.

### codebase-summary.md
- New directory: 1 row in the directory table with 1-line purpose.
- New convention pattern: 1 bullet under conventions section if relevant.

### project-roadmap.md
- Move feature from "In Progress" → "Shipped" with date.
- Add follow-up items only if known (e.g. "Phase 2.2 cart <feature-slug>", "Flag cleanup S+2").

### code-standards.md
- Edit only if the feature established a new repeatable pattern (e.g. "use TanStack Query for all admin data fetching" — if not previously standardised).

## 5. Anti-patterns

- ❌ Rewriting the entire project-level doc when only 1 section needs update.
- ❌ Skipping `project-roadmap.md` shipped-marker — this is the single source of truth for "what's live".
- ❌ Adding marketing language ("revolutionary new feature!") — keep docs technical and terse.
- ❌ Editing `code-standards.md` for one-off code — only update when convention is established and reusable.
- ❌ Forgetting to bump `last_updated` frontmatter on edited files.
- ❌ Updating project-level docs for a feature still behind a flag <100%.

## 6. After edits

1. Run a final `grep` of new module/endpoint names against project docs to confirm at least one mention each.
2. Notify `@tech-lead` and `@docs-maintainer` for review.
3. Open a PR with `docs:` conventional commit prefix — keep it scoped to docs only.

## 7. Example triggers

EN:
- "Sync project docs after <feature-slug> shipped"
- "Update system-architecture for the new BXGY service"
- "Reflect the v2 API endpoints in codebase-summary"
- "Mark phase 2.1 shipped in the roadmap"
- "Roll project docs forward after the release"

VN:
- "Đồng bộ docs sau release <feature-slug>"
- "Cập nhật system-architecture cho service BXGY"
- "Update roadmap đánh dấu phase 2.1 đã ship"
- "Sync project-level docs sau khi feature live"
