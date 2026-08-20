---
name: feature-incident-postmortem
description: Write a blameless incident postmortem at `docs/_incidents/<YYYY-MM-DD>-<slug>.md` covering Summary, Timeline, Impact, Root cause (5-whys), Contributing factors, Resolution, Detection, Action items (with owner + due date), and Lessons learned. Use this skill EVERY TIME a user (SRE, Tech Lead, On-call) asks to "write a postmortem", "document an incident", "do a 5-why analysis", "viết postmortem", "phân tích sự cố", "RCA report", or after any production incident (SEV-1/2/3) with customer impact, data loss, or downtime > 5 minutes. Output is a single file. Action items become tracked tasks; recurring root causes trigger an ADR. Out of scope: writing the fix code (separate skill / `fix` workflow). Postmortems are blameless — describe systems and processes, never individuals.
---

# Incident Postmortem (v1.0)

Blameless. Focus on system/process, never people.

## When to use

- Production incident with customer impact (SEV-1, SEV-2).
- SEV-3 if recurring (3rd time same root cause → mandatory).
- Data loss or corruption (any size).
- Downtime > 5 minutes on user-facing endpoint.
- Security incident (treat as SEV-1 by default).

## Inputs (gather before writing)

1. **Incident ID** — `INC-<YYYY-MM-DD>-NN` (NN = nth incident that day).
2. **Severity** — SEV-1 / SEV-2 / SEV-3 (define in `docs/_conventions/` if not yet).
3. **Detection source** — alert, customer report, internal find.
4. **Timeline raw data** — Slack/PagerDuty/log timestamps. Convert to Asia/Saigon, sort chronological.
5. **Linked artefacts** — Sentry issue, Grafana snapshot, runbook used, commit/PR that introduced the bug.

## Output

Path: `docs/_incidents/<YYYY-MM-DD>-<slug>.md` (slug = kebab-case, e.g. `2026-05-06-checkout-timeout`).

## Procedure

### Step 1 — Confirm incident is closed

Postmortem is written **after** mitigation, not during firefighting. If incident is still active, refuse and tell the user to use the `fix` skill / on-call playbook first.

### Step 2 — Gather timeline

Build chronological list. Each row: `HH:MM | actor | event`. Include detection, escalation, mitigation attempts (failed + succeeded), full resolution. Use **objective wording** ("alert fired" not "monitoring failed to fire earlier").

### Step 3 — Run 5-whys for root cause

Start from "user impact" → ask why 5 times. Record each layer. The deepest "why" answerable without speculation is the root cause. If you hit "human error" by why-3, keep going — humans don't error in a vacuum, find the system condition that allowed it.

### Step 4 — List contributing factors

Things that didn't cause but worsened: missing alert, stale runbook, oncall not paged, doc out of date, test not catching, recent change adjacent to failure.

### Step 5 — Action items

Each action item:
- **Owner** (single person, not team).
- **Due date** (absolute, ≤30 days).
- **Type**: `prevent` (root cause), `detect` (faster alert), `respond` (faster mitigation), `doc` (runbook/SRS update).
- **Linked artefact**: Jira ticket / GitHub issue / PR.

Bias toward `prevent` + `detect`; `respond` only if root cause is genuinely unfixable.

### Step 6 — Write the file using this template

```markdown
---
incident_id: INC-<YYYY-MM-DD>-NN
title: <one-line title>
severity: SEV-<1|2|3>
status: closed
detected_at: <YYYY-MM-DD HH:MM Asia/Saigon>
mitigated_at: <YYYY-MM-DD HH:MM>
resolved_at: <YYYY-MM-DD HH:MM>
duration_minutes: <N>
affected_features: [<feature-slug>, ...]
authors: [<name1>]
related_adrs: []          # filled if a follow-up ADR is opened
related_commits: [<sha>]
---

# Postmortem: <title>

## 1. Summary
2-3 sentences. What happened, who was affected, how it ended. No jargon — a non-engineer should understand.

## 2. Impact
- Users affected: <N or %>
- Revenue / SLA impact: <$ or "none measured">
- Data loss / corruption: <yes/no — describe>
- Duration: <minutes>

## 3. Timeline (Asia/Saigon)
| Time | Actor | Event |
|---|---|---|
| HH:MM | system | alert X fired |
| HH:MM | oncall | acked, started investigation |
| ... | | |
| HH:MM | oncall | mitigation deployed (commit <sha>) |
| HH:MM | system | metric Y back to baseline |

## 4. Root cause (5-whys)
1. Why did users see X? → ...
2. Why ...? → ...
3. Why ...? → ...
4. Why ...? → ...
5. Why ...? → **<root cause stated as a system/process condition>**

## 5. Contributing factors
- <factor 1>
- <factor 2>

## 6. What went well
- <positive — fast detect, clean rollback, etc.>

## 7. What went wrong
- <gaps — late alert, missing runbook, etc.>

## 8. Action items
| ID | Type | Description | Owner | Due | Tracker |
|---|---|---|---|---|---|
| AI-1 | prevent | <action> | <name> | <YYYY-MM-DD> | <link> |
| AI-2 | detect | <action> | <name> | <YYYY-MM-DD> | <link> |

## 9. Lessons learned
1-3 bullets. Things the team will do differently. Generalisable, not specific to this incident.

## 10. References
- Sentry: <link>
- Grafana snapshot: <link>
- PR/commits: <link>
- Affected docs: <docs/phases/<phase-slug>/features/<slug>/01-srs.md, ...>
```

### Step 7 — Cross-link

- For each `affected_feature`, append a 1-line entry in that feature's `05-release-plan.md` Section "Incident log" (create section if missing): `- INC-<id> (YYYY-MM-DD): <title>. SEV-<N>. Postmortem: <link>.`
- If root cause reveals a missing/wrong rule in SRS or Tech Design, propose a doc patch (delegate to `feature-srs-author` / `feature-tech-design-author`).
- If root cause is a cross-cutting decision (e.g. "we should never use sync HTTP in webhook handlers"), suggest opening an ADR via `feature-adr-author`.

### Step 8 — Schedule the review

Postmortem must be reviewed in a meeting within 7 days. Output a reminder line in the final report: `Review meeting: schedule by <YYYY-MM-DD>. Required attendees: <oncall, tech lead, PO if customer impact>.`

## Anti-patterns

- ❌ Naming individuals as causes ("Alice deployed bad code") — name the system gap ("deploy lacked staging soak time").
- ❌ "Human error" as root cause — keep asking why.
- ❌ Action items without owner or due date.
- ❌ Action items > 30 days out — too vague to be real; break down or drop.
- ❌ Writing during the incident — wait until mitigated.
- ❌ Over-action ("rewrite the entire service") — match action scope to actual root cause.
