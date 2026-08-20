---
name: feature-shopify-appstore-checklist
description: Produce and verify a Shopify App Store submission readiness checklist for the app, covering the mandatory requirements that gate approval — OAuth/managed install, embedded app + latest App Bridge + session tokens (no cookies), GraphQL Admin API only (REST banned for new apps since Apr 2025), the 3 GDPR compliance webhooks, Shopify-native billing (no off-platform), TLS, performance budgets (storefront Lighthouse ≤10pt drop; admin Web Vitals LCP≤2.5s / CLS≤0.1 / INP≤200ms; checkout p95≤500ms), prohibited practices, and listing assets (icon, screenshots, privacy policy URL). Optionally assesses Built for Shopify (BFS) tier. Use this skill EVERY TIME a user asks to "is my app ready for the App Store", "submission checklist", "app review requirements", "pass app review", "prepare for App Store", "Built for Shopify", "check app store compliance", "kiểm tra điều kiện lên App Store", "chuẩn bị submit app", "yêu cầu review app", or before submitting an app for listing/review. Output is a single report at plans/reports/appstore-readiness-<date>.md with PASS/FAIL per requirement + fixes. Out of scope: implementing the fixes (delegate to feature-shopify-webhooks / feature-shopify-billing / feature-shopify-extension-impl).
---

# Feature Shopify App Store Checklist (v1)

Guideline to audit an app against Shopify App Store mandatory requirements before submission.

## Core principle

> **App Store review fails hard on a fixed set of mandatory requirements. This skill audits each, reports PASS/FAIL with evidence (file/config), and lists the exact fix + which skill implements it. It does NOT write fix code — it gates.**

## Output

Single report: `plans/reports/appstore-readiness-<YYYY-MM-DD>.md` — table of requirement → PASS/FAIL/N-A → evidence → fix owner skill.

## Procedure

### Step 1 — Audit MANDATORY requirements

| # | Requirement | How to verify (this repo) | Fix via |
|---|---|---|---|
| 1 | **OAuth / managed install** immediately, no manual domain entry | check auth flow uses `@shopify/shopify-app-react-router` `authenticate.admin` | — |
| 2 | **Embedded + latest App Bridge** via `<script src=".../app-bridge.js">` in `<head>`; **session tokens** only (NO third-party cookies / localStorage auth) | grep app root head; confirm `embedded = true` in `shopify.app.toml` | — |
| 3 | **GraphQL Admin API only** (REST banned for new public apps since 2025-04-01) | grep for REST `/admin/api/.../*.json` REST calls — must be none | feature-backend-impl |
| 4 | **3 GDPR compliance webhooks** subscribed + handled | check `shopify.app.toml` `compliance_topics` + `app/api/webhooks/compliance.ts` | **feature-shopify-webhooks** |
| 5 | **Shopify-native billing** (App Pricing or Billing API); NO off-platform billing | check billing config / dashboard pricing; grep for external payment links | **feature-shopify-billing** |
| 6 | **TLS/SSL** on all endpoints (webhook URIs HTTPS) | confirm prod URL + cert | infra |
| 7 | **No prohibited practices** | fake reviews/sales, external checkout bypass, marketplace, third-party POS, lending, freelancer marketplace | — |

### Step 2 — Audit PERFORMANCE budgets (concrete thresholds)

| Surface | Metric | Threshold | Notes |
|---|---|---|---|
| **Storefront** (mandatory) | Lighthouse perf score drop | **≤ 10 points** | weighted: Home 17% / Product 40% / Collection 43%; avg multiple runs |
| **Admin** (BFS tier) | LCP / CLS / INP | **≤2.5s / ≤0.1 / ≤200ms** | 75th pct, ≥100 samples/28d; needs latest App Bridge to collect |
| **Checkout** (if app affects checkout) | p95 latency / failure rate | **≤500ms / ≤0.1%** | ≥1000 req/28d |

### Step 3 — Audit LISTING assets

- Distribution = "Shopify App Store" selected.
- App icon, screenshots, clear description, **pricing**, benefits.
- **Privacy policy URL** present.

### Step 4 — (optional) Built for Shopify (BFS) tier

Admin Web Vitals (Step 2) + storefront ≤10pt + category-specific requirements + annual review. BFS = highest badge; latest App Bridge required to be assessable.

### Step 5 — Write the report

PASS/FAIL per row + evidence (file path / config line / grep result) + fix owner skill. Overall verdict: READY / NOT READY. List blockers first.

## GOTCHAS

- New apps cannot use REST Admin API (post 2025-04) — GraphQL only.
- Embedded apps must NOT use cookies/localStorage for auth — session tokens only.
- Web Vitals only collected if latest App Bridge present; without it the app can't be assessed for BFS.
- 10-pt Lighthouse limit is **storefront-only**; admin uses Web Vitals (different metrics).
- Lighthouse varies run-to-run + iframe rendering misreports — average multiple runs; Shopify uses separate-runtime Web Vitals.
- "Managed Pricing" is now "Shopify App Pricing"; Billing API = legacy but allowed.
- Authoritative live source (verify at submission): `shopify.dev/docs/apps/launch/app-store-review/app-store-ai-self-review-requirements`, `.../built-for-shopify/requirements`.
