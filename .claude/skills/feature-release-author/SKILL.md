---
name: feature-release-author
description: Write a Release Plan for a feature, placed at docs/phases/<phase-slug>/features/<slug>/05-release-plan.md with feature flag strategy, rollout plan (canary %, gradual ramp), kill switch / rollback runbook, monitoring dashboards, alert thresholds, post-launch checklist, and on-call playbook. Use this skill EVERY TIME a user (Tech Lead, Release Manager, SRE, DevOps) asks to "write a release plan", "plan the rollout", "create a runbook", "define rollback strategy", "monitoring + alerts for feature", "feature flag plan", "lập kế hoạch release", "viết runbook", "kế hoạch rollback", or whenever a feature has FE/BE implementation done and needs a documented rollout strategy before shipping to production. Output is single file. Out of scope (intentionally): code-level changes (those go in tech design), infra changes outside feature scope.
---

# Feature Release Plan Author

## 1. When to use

Activate this skill when:

- FE + BE implementation for the feature is **complete** (PR merged or behind-flag).
- Test Plan exit criteria from `04-test-plan.md` are green (or near-green).
- The feature is queued for production rollout and needs a documented strategy.
- A user asks to "write a release plan", "plan the rollout", "create a runbook",
  "define rollback strategy", "lập kế hoạch release", "viết runbook", etc.

**Do NOT** activate when:
- Implementation is not yet complete — release plan describes how to ship existing code, not a coding task.
- The change is a hotfix with no flag (use a lightweight runbook only, not this 7-section doc).

## 2. Required output

Single file:

```
docs/phases/<phase-slug>/features/<slug>/05-release-plan.md
```

`<slug>` matches the feature slug used by the SRS / Tech Design / Test Plan in the same folder.

For multi-release features (e.g. Phase 2.1, 2.2 of <feature-slug>), suffix with phase:
```
docs/phases/<phase-slug>/features/<slug>/05-release-plan-phase-2-1.md
docs/phases/<phase-slug>/features/<slug>/05-release-plan-phase-2-2.md
```

## 3. Procedure

1. **Resolve `<phase-slug>`**: if not given, run `grep -l "<slug>" docs/phases/*/00-prd.md` — the matching phase folder is `<phase-slug>`. Ask the user if 0 or >1 matches. All upstream docs below live under `docs/phases/<phase-slug>/features/<slug>/`.
2. **Read upstream docs** in this order:
   - `01-srs.md` — Section 7 (runtime behavior), Section 8 (QA scenarios) — to know what user-visible changes to monitor.
   - `03-tech-design.md` — Section 8 (performance budget) for thresholds, Section 10 (migration plan) for ordering, Section 11 (API endpoints) for monitoring targets.
   - `04-test-plan.md` — Section 9 (exit criteria) — confirm green before flipping flag.
3. **Confirm prerequisites**:
   - All High-priority TC-IDs in Test Plan passed.
   - Tech Design Section 10 migration steps applied to staging.
   - Feature flag created in flag system (LaunchDarkly / config table / env var).
4. **Write the 7-section plan** (see Section 5 below).
5. **Cross-link**:
   - Reference Tech Design Section 10 migration plan from Section 5 (rollback).
   - Reference Tech Design Section 8 perf budget from Section 4 (monitoring).
   - Reference Test Plan Section 9 exit criteria from Section 1 (release summary).

## 4. Required frontmatter

```yaml
---
feature_slug: <slug>
phase_slug: <phase-slug>             # e.g. phase-2-example-b
doc_type: release-plan
version: 1.0
status: draft                         # draft | review | approved | shipped
last_updated: YYYY-MM-DD
owner: "@release-manager @tech-lead"
depends_on:
  - 01-srs.md
  - 03-tech-design.md
  - 04-test-plan.md
---
```

## 5. Required content structure (7 sections)

### Section 1. Release summary
- Version (semver of feature, e.g. v1.0.0).
- Target release date.
- Target audience (shop tiers, region, % rollout).
- Scope: 1-paragraph plain English of what is shipping.
- Link to Test Plan exit criteria status.

### Section 2. Feature flag strategy
- Flag name (snake_case, e.g. `example_feature_v2_enabled`).
- Flag system (LaunchDarkly / `app_config` table / env var).
- Default state (off in production, on in staging).
- % rollout schedule with concrete dates (or "T+N days").
- Targeting rules (by shop_id, plan tier, region).
- Cleanup plan: when to remove the flag (target sprint).

### Section 3. Rollout plan
Phases — each MUST have entry criteria, exit criteria, estimated duration, owner:

| Phase | Audience | Duration | Entry criteria | Exit criteria | Owner |
|---|---|---|---|---|---|
| Internal staging | dev/QA shops | 2 days | Tests green | No P0/P1 bugs | QA |
| Canary | 5% of merchants | 24h | Staging clean | Error rate < 0.5% | SRE |
| Ramp 25% | 25% | 24h | Canary clean | Error rate < 0.5% | SRE |
| Ramp 50% | 50% | 24h | 25% clean | Error rate < 0.5% | SRE |
| GA 100% | All | — | 50% clean for 24h | — | Release Manager |

State explicit advance criteria — concrete metric thresholds, not "looks good".

### Section 4. Monitoring & alerts
- Dashboard URL (Grafana / DataDog / New Relic).
- Key metrics — table with target, threshold, alert action:

| Metric | Target | Warn threshold | Critical threshold | Alert channel |
|---|---|---|---|---|
| API p95 latency `/api/v1/<feature-slug>/*` | <300ms | >500ms 5min | >1s 5min | #release-alerts (Slack), PagerDuty |
| Error rate | <0.5% | >1% 5min | >5% 1min | PagerDuty |
| DB CPU | <60% | >75% 10min | >90% 2min | PagerDuty |
| Feature-specific KPI (e.g. AOV uplift) | +X% | -5% vs control | -10% vs control | #product-metrics |

- Alert rule definitions linked / inlined.
- Log queries (saved searches) for debugging.

### Section 5. Rollback runbook
- **Trigger criteria** — exact conditions that mandate rollback (e.g. critical alert fires for >5min, P0 bug confirmed, data-corruption detected).
- **Step-by-step rollback** — copy-pastable commands:
  ```bash
  # 1. Kill flag (instant)
  ld-cli flag set example_feature_v2_enabled --rollout 0

  # 2. Verify traffic drops on dashboard (within 60s)
  open https://grafana.../example-feature

  # 3. If schema migration needs revert (Tech Design §10 step N):
  pnpm prisma migrate resolve --rolled-back <migration_name>

  # 4. Notify
  # post in #incidents using template below
  ```
- **Data integrity check** — SQL queries to confirm no orphaned rows / corrupted state.
- **Comms template** — pre-written Slack/email message for incident channel + customer-facing if needed.

### Section 6. Post-launch checklist
- Smoke test list (concrete URLs / API calls / merchant flows).
- Key flow validation — manual QA sign-off in production within 1h of GA.
- Business metric tracking — what to watch for week 1 (conversion rate, refund rate, support tickets).
- Code cleanup — flag removal task scheduled for sprint N+2.
- Docs sync — invoke `feature-docs-sync` skill.

### Section 7. Open questions
List unresolved questions with owner + due date. Block GA until resolved or accepted.

## 6. Writing rules

- **Concrete commands, not vague text.** "Run `ld-cli flag set X --rollout 0`" not "disable the flag".
- **Every metric has a threshold and an action.** "Monitor latency" is useless. "Alert PagerDuty if p95 > 1s for 5min" is actionable.
- **Rollback steps must be runnable copy-paste.** Test them on staging first.
- **On-call coverage** — explicitly name primary + backup for first 48h post-GA.
- **Time-bound everything** — durations, dates, sprint numbers.

## 7. Anti-patterns (DO NOT do these)

- ❌ Rollback "revert PR and redeploy" without feature-flag kill-switch — too slow for production incidents.
- ❌ Vague metrics ("monitor for issues") — say what metric, what threshold, what action.
- ❌ No on-call assignment for first 48h post-launch — incidents need named owners.
- ❌ Skipping canary phase for stateful changes (DB schema, data migrations).
- ❌ Forgetting flag cleanup task — dead flags rot the codebase.
- ❌ Copy-pasting last release's plan without re-reading Tech Design §10 migration delta.

## 8. After creation

1. Notify `@release-manager` and `@on-call` for review.
2. Verify Tech Design Section 10 migration plan aligns with Section 5 rollback steps.
3. Confirm Test Plan Section 9 exit criteria are all green before flipping the flag.
4. Add the file to PR description for visibility.
5. After GA, update `status: shipped` and trigger `feature-docs-sync`.

## 9. Example triggers

EN:
- "Write a release plan for the <feature-slug> feature"
- "Plan the rollout for phase 2.1"
- "Create a runbook for shipping the new checkout flow"
- "Define the rollback strategy for the BXGY feature"
- "Set up monitoring + alerts for the example launch"
- "Feature flag plan for example v2"

VN:
- "Lập kế hoạch release cho feature <feature-slug>"
- "Viết runbook cho việc ship phase 2.1"
- "Kế hoạch rollback cho feature BXGY"
- "Thiết lập monitoring và alerts cho launch example feature"
- "Feature flag plan cho example v2"
