---
name: feature-bugfix-doc-sync
description: Weekly catch-all skill that scans recent `fix:` commits (default last 7 days) and proposes doc patches for bugs that touched business rules, schema, endpoints, state transitions, or validation but did not bump the affected SRS / Tech Design / Design Spec version. Output is a single report at `plans/reports/bugfix-doc-sync-<YYYY-MM-DD>.md` listing each commit → likely affected doc section + suggested patch. Use this skill EVERY TIME a user (Tech Lead, Doc Maintainer) asks to "sync bugfix docs", "weekly doc hygiene", "rà soát bugfix docs", "đồng bộ docs sau bugfix", "find drifted docs", or as a scheduled weekly cron via the `schedule` skill. Catch-all for bugs that bypassed the `fix` skill's per-commit doc-impact check. Out of scope: writing the actual doc patches (delegate to `feature-srs-author` / `feature-tech-design-author` / `feature-design-spec-author`).
---

# Bugfix Doc Sync (v1.0)

This skill is the safety net for the per-commit doc-impact check in `~/.claude/skills/fix/`. Bugs fixed manually (without `/fix`) or via squash-merge often skip the doc bump — this catches them weekly.

## When to use

- Weekly cron (recommend Monday 09:00 Asia/Saigon via the `schedule` skill).
- Before a phase status review.
- When user suspects "docs feel stale".
- Ad-hoc: pass a custom `--since` range (e.g. `--since=1.month`).

## Inputs

- `--since` (default `1.week`) — git log time window.
- `--feature` (optional) — restrict to one feature slug.
- `--branch` (default current branch) — git log target.

## Procedure

### Step 1 — Collect candidate commits

```bash
git log --since=<since> --pretty=format:'%h%x09%ad%x09%s' --date=short \
  --grep='^fix:' --grep='^bug:' --grep='^hotfix:' \
  -- 'app/**' 'prisma/**'
```

Filter out: revert commits, dependabot, lockfile-only, doc-only.

### Step 2 — Per commit, classify changed files

Use the same mapping table as `~/.claude/skills/fix/SKILL.md` Step 5.2:

| File pattern | Likely affected doc section |
|---|---|
| `prisma/schema*` / migrations | Tech Design §3 + §10 |
| `app/api/**`, route handlers | Tech Design §11 |
| `app/services/**` | SRS §3 + Tech Design §7 |
| State machine code | SRS §6 |
| Zod / validation refines | SRS §5 |
| `app/modules/**` (FE) | Design Spec §2 |

For each commit, list affected feature(s). Resolve feature slug from path (e.g. `app/services/<feature-slug>/*` → `example-feature-b` if filename hints, else infer from PR title or ask user).

### Step 3 — Check whether the affected doc was bumped

For each `(commit, feature, doc)` triple:

```bash
git log --since=<commit-date> --until=<+3 days> --oneline \
  -- docs/phases/<phase-slug>/features/<feature-slug>/<doc>.md
```

If 0 commits touched the doc within 3 days of the fix → **drift candidate**.

### Step 4 — Read the bug commit + the current doc section

For each drift candidate:
- Read the commit diff (`git show <sha>`).
- Read the doc section that should have changed.
- Compose a 1-paragraph "what the doc currently says vs. what the fix implies".

### Step 5 — Generate the report

Path: `plans/reports/bugfix-doc-sync-<YYYY-MM-DD>.md`.

```markdown
# Bugfix Doc Sync — <YYYY-MM-DD>
Range: <since> → today | Branch: <branch>

## Summary
- Fix commits scanned: N
- Drift candidates: N
- Already in sync: N (skipped)

## Drift candidates

### 1. <commit-sha> · <commit-subject>
- Date: YYYY-MM-DD
- Feature: <feature-slug> (<phase-slug>)
- Files changed: <list>
- Likely affected doc: `docs/phases/<phase-slug>/features/<slug>/<doc>.md` §<section>
- Current doc says: <quote>
- Fix implies: <1-line summary of new rule/behavior>
- Suggested patch:
  ```diff
  - <old line>
  + <new line>
  ```
- Suggested version bump: `<X.Y.Z>` → `<X.Y.Z+1>`
- Recommended skill: `feature-srs-author` | `feature-tech-design-author` | `feature-design-spec-author`

### 2. ...

## Already-in-sync (audit trail)
| Commit | Feature | Doc bumped |
|---|---|---|
| abc123 | ... | ✓ v1.0.3 |

## Recurring root causes
If 3+ candidates point to the same root file/section, flag for ADR consideration.
```

### Step 6 — Final report to user

- One-line verdict: `<N> drift candidates found. <M> ready for cleanup.`
- Path to the report file.
- Suggest: "run the appropriate `feature-*-author` skill on each candidate, or batch via the `cook` skill".
- If a recurring root cause emerged → suggest `feature-adr-author` to lock the lesson.

## Anti-patterns

- ❌ Auto-write doc patches — this skill **proposes only**. Human reviews + delegates to author skills.
- ❌ Treat absence of doc bump as definitive drift — sometimes the fix doesn't change semantics (e.g. typo fix in a regex). Flag, don't accuse.
- ❌ Skip "Already-in-sync" table — useful audit trail proving the per-commit check is working.
- ❌ Run on a branch with WIP commits — only run on `main` / `develop` / merged history.
