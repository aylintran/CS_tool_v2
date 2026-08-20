---
name: feature-shopify-webhooks
description: Implement Shopify webhooks for a feature or app, including the 3 MANDATORY GDPR/privacy compliance webhooks (customers/data_request, customers/redact, shop/redact) required for App Store approval, plus lifecycle webhooks (app/uninstalled, app/scopes_update) and any business webhooks (orders/create, products/update, etc.). Covers shopify.app.toml subscription config (topics vs compliance_topics, api_version, uri), HMAC verification, the authenticate.webhook(request) handler pattern in @shopify/shopify-app-react-router, idempotency, and 200-within-48h response rule. Use this skill EVERY TIME a user asks to "add a webhook", "handle app uninstall", "implement GDPR webhooks", "compliance webhooks", "register webhook", "subscribe to orders/create", "verify webhook HMAC", "thêm webhook", "xử lý gỡ app", "webhook tuân thủ GDPR", "đăng ký webhook", or whenever an app needs to react to Shopify events or pass App Store privacy review. Default stack is React Router 7 + @shopify/shopify-app-react-router + Prisma (Postgres). Out of scope: billing webhooks handled by the billing object (use feature-shopify-billing).
---

# Feature Shopify Webhooks (v1)

Guideline for implementing Shopify webhooks correctly — especially the mandatory compliance webhooks that block App Store approval if missing.

## Core principle

> **Every public app MUST subscribe to + handle the 3 GDPR compliance topics. HMAC is verified on the RAW body before parsing. Handlers must be idempotent and return HTTP 200 within 48h. Webhook topic strings differ between TOML (`app/uninstalled`, lowercase/slash) and handler code (`APP_UNINSTALLED`, UPPER_SNAKE_CASE).**

## Verify via MCP first (REQUIRED)

Before writing webhook code, if unsure about a topic string, payload shape, or TOML key → check `shopify-dev-mcp`: `learn_shopify_api(api: "admin")` → `search_docs_chunks(conversationId, "<query>")`. Don't guess from memory.

## Tech assumptions (this repo)

- **Lib**: `@shopify/shopify-app-react-router` — `authenticate.webhook(request)` does HMAC verify + parse for you.
- **Config**: app-specific webhooks declared in `shopify.app.toml` (auto-synced on `shopify app deploy`, NOT on `dev`).
- **Handlers**: route files under `app/api/webhooks/` (this repo already has `uninstalled.ts`, `scopes-update.ts`).
- **DB**: Prisma Postgres — session cleanup on uninstall, compliance data export/delete.
- **API version**: pin to the version already in `shopify.app.toml` (`[webhooks] api_version`, e.g. `2025-07`).

## When to use this skill

- Adding/handling any Shopify webhook.
- Preparing an app for App Store submission (compliance webhooks are mandatory).
- Debugging webhook 401/HMAC failures or missing deliveries.

## Procedure (REQUIRED ORDER)

### Step 1 — Configure subscriptions in `shopify.app.toml`

```toml
[webhooks]
api_version = "2025-07"   # ONE version for ALL webhook events; cannot override per-subscription

# Lifecycle — normal `topics`
[[webhooks.subscriptions]]
topics = ["app/uninstalled"]
uri = "/api/webhooks/uninstalled"

[[webhooks.subscriptions]]
topics = ["app/scopes_update"]
uri = "/api/webhooks/scopes-update"

# MANDATORY GDPR — use `compliance_topics` (NOT `topics`)
[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/api/webhooks/compliance"
```

- `uri` = relative HTTPS path (or `pubsub://...` / EventBridge ARN).
- Optional: `filter = "..."`, `include_fields = [...]`, `name`.

### Step 2 — The 3 mandatory compliance webhooks

| Topic (TOML) | Handler topic | Must do | Payload key fields |
|---|---|---|---|
| `customers/data_request` | `CUSTOMERS_DATA_REQUEST` | Compile + return stored customer data to merchant (out-of-band ok) | `shop_id`, `shop_domain`, `customer{id,email,phone}`, `orders_requested[]`, `data_request{id}` |
| `customers/redact` | `CUSTOMERS_REDACT` | **Delete** the customer's stored data | `shop_id`, `shop_domain`, `customer{id,email,phone}`, `orders_to_redact[]` |
| `shop/redact` | `SHOP_REDACT` | **Delete** all shop data (sent ~48h after uninstall) | `shop_id`, `shop_domain` |

Respond **HTTP 200 within 48h**. Non-200/timeout → Shopify retries then removes subscription → blocks review.

### Step 3 — Handler pattern (`app/api/webhooks/compliance.ts`)

```ts
import type { ActionFunctionArgs } from "react-router";
import { authenticate } from "app/shopify.server"; // adjust to repo's shopify server export

export const action = async ({ request }: ActionFunctionArgs) => {
  // authenticate.webhook verifies X-Shopify-Hmac-Sha256 on the RAW body, then parses.
  const { topic, shop, payload } = await authenticate.webhook(request);

  switch (topic) {
    case "CUSTOMERS_DATA_REQUEST":
      // gather + deliver data for payload.customer
      break;
    case "CUSTOMERS_REDACT":
      // delete data for payload.customer (idempotent)
      break;
    case "SHOP_REDACT":
      // delete ALL data for `shop` (idempotent)
      break;
    default:
      throw new Response("Unhandled topic", { status: 404 });
  }
  throw new Response(); // 200
};
```

For `APP_UNINSTALLED`: guard `session` may be `undefined` (webhook can arrive after uninstall):
```ts
const { shop, session } = await authenticate.webhook(request);
if (session) await db.session.deleteMany({ where: { shop } });
```

### Step 4 — Idempotency + verification

- Webhooks can be delivered **more than once** → handlers MUST be idempotent (use `webhookId`, upsert/deleteMany, no "throw if exists").
- Never parse/mutate the body before HMAC — `authenticate.webhook` does verification first; only a risk if hand-rolling. If hand-rolling: base64 HMAC-SHA256 of raw body keyed by app client secret, constant-time compare against `X-Shopify-Hmac-Sha256`.

### Step 5 — Deploy + verify

- `shopify app deploy` (TOML webhooks only sync here, not on `dev`).
- Confirm registration; shop-specific subs (registered via GraphQL `webhookSubscriptionCreate`) show in `webhookSubscriptions` query — TOML ones do NOT.

## Output

- Updated `shopify.app.toml` `[webhooks]` block.
- Handler route files in `app/api/webhooks/` (`compliance.ts` at minimum + any business topics).
- Prisma deletes/exports wired for compliance.

## GOTCHAS

- Compliance uses `compliance_topics`, NOT `topics` — most common miss.
- `api_version` is global per `[webhooks]`; can't set per-subscription.
- Handler topics are UPPER_SNAKE_CASE; TOML topics are lower/slash.
- `session`/`admin` may be `undefined` on `APP_UNINSTALLED` — guard DB writes.
- Must return 200 even when handled-and-done; else retries → auto-removal.
- TOML webhooks sync on `deploy`, not `dev`.
- Handlers must be idempotent (duplicate delivery is normal).
