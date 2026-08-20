---
name: feature-shopify-billing
description: Implement Shopify app monetization — subscription plans, usage charges, and trials — using Shopify-native billing (off-platform billing is prohibited and blocks App Store distribution). Covers choosing Shopify App Pricing (dashboard-configured, preferred for new apps) vs the legacy Billing API (code-based via @shopify/shopify-app-react-router billing object), plan config (lineItems with amount/currencyCode/interval Every30Days|Annual|Usage), billing.require()/request()/check()/cancel(), the appSubscriptionCreate mutation, confirmationUrl redirect flow, trialDays, usage caps, replacementBehavior, and isTest mode for dev stores. Use this skill EVERY TIME a user asks to "add billing", "charge merchants", "create a subscription plan", "add a paywall", "gate features by plan", "usage-based pricing", "free trial", "thêm thanh toán", "tạo gói subscription", "tính phí merchant", "chặn tính năng theo gói", or whenever an app needs to make money. Default stack is React Router 7 + @shopify/shopify-app-react-router. Out of scope: GDPR/lifecycle webhooks (use feature-shopify-webhooks).
---

# Feature Shopify Billing (v1)

Guideline for monetizing a Shopify app the only allowed way — through Shopify's billing.

## Core principle

> **Off-platform billing is prohibited and blocks App Store distribution. New apps should default to Shopify App Pricing (configured in the Partner/Dev Dashboard at submission). Use the code-based Billing API only when App Pricing can't express the model. A merchant approves a charge via Shopify's hosted confirmationUrl. Always pass `isTest: true` on dev stores or you attempt real charges.**

## Decide first: App Pricing vs Billing API

- **Shopify App Pricing** (was "Managed Pricing") — DEFAULT/preferred. Configured in dashboard, NOT code. Shopify hosts the plan-selection UI + billing card. Supports subscriptions + usage (App Events API meters). → No code beyond gating features by active plan (read subscription status).
- **Billing API** (LEGACY, still works) — code-based. Use only for models App Pricing can't cover or existing apps.

> If the user just wants standard tiered subscriptions → recommend App Pricing (dashboard) and only write the feature-gating code. Below covers the Billing API path when needed.

## Verify via MCP first (REQUIRED)

Before coding billing, confirm current method signatures / mutation fields / enum names via `shopify-dev-mcp`: `learn_shopify_api(api: "admin")` → `search_docs_chunks(conversationId, "<query>")`. Billing terminology + APIs change — don't guess.

## Tech assumptions (this repo)

- **Lib**: `@shopify/shopify-app-react-router` — `billing` object from `authenticate.admin(request)`.
- **Config**: `billing` key in the `shopifyApp({...})` config (this repo's `shopify.server.ts`).
- **API version**: pin to repo's configured version.

## Procedure (Billing API path)

### Step 1 — Define plans in the shopifyApp config

```ts
import { shopifyApp, BillingInterval } from "@shopify/shopify-app-react-router/server";

export const MONTHLY_PLAN = "Monthly subscription";
export const ANNUAL_PLAN = "Annual subscription";

const shopify = shopifyApp({
  billing: {
    [MONTHLY_PLAN]: {
      lineItems: [{ amount: 5, currencyCode: "USD", interval: BillingInterval.Every30Days }],
      trialDays: 7,
    },
    [ANNUAL_PLAN]: {
      lineItems: [{ amount: 50, currencyCode: "USD", interval: BillingInterval.Annual }],
    },
  },
});
```
`BillingInterval`: `Every30Days` | `Annual` | `Usage`. Use `lineItems[]` (v3+) — NOT flat `amount`/`interval`.

### Step 2 — Gate features (loader/action)

```ts
const { billing } = await authenticate.admin(request);

// Hard gate: redirect to approval if no active plan
await billing.require({
  plans: [MONTHLY_PLAN, ANNUAL_PLAN],
  isTest: process.env.NODE_ENV !== "production",
  onFailure: async () => billing.request({ plan: MONTHLY_PLAN }),
});

// OR non-redirecting status check
const { hasActivePayment, appSubscriptions } = await billing.check({
  plans: [MONTHLY_PLAN], isTest: true,
});
```

### Step 3 — Request a charge (redirect flow)

`billing.request({ plan, isTest, returnUrl?, trialDays?, lineItems? })` → redirects merchant to Shopify's `confirmationUrl` → merchant approves → subscription active (after trial). `trialDays`/`lineItems` override config.

### Step 4 — Cancel + usage

- `billing.cancel({ subscriptionId, isTest, prorate })`.
- Usage: `billing.createUsageRecord(...)`, `billing.updateUsageCappedAmount({ subscriptionLineItemId, cappedAmount: { amount, currencyCode } })`.

### Step 5 — (raw GraphQL if needed) `appSubscriptionCreate`

Inputs: `name`, `returnUrl`, `lineItems[]` (`appRecurringPricingDetails{ price, interval, discount }` or `appUsagePricingDetails{ cappedAmount, terms }`), `trialDays`, `test`, `replacementBehavior`. Returns `confirmationUrl` → redirect merchant.

## Output

- `billing` config in `shopify.server.ts`.
- Feature-gating in loaders/actions (`billing.require` / `billing.check`).
- Approval + return-url handling; cancel flow where relevant.

## GOTCHAS

- New apps → prefer **Shopify App Pricing** (dashboard), not code Billing API.
- Use `lineItems[]`; old flat `amount`/`currencyCode`/`interval` shape errors in v3+.
- Forgetting `isTest: true` on dev stores = real charge attempts.
- One active subscription per merchant; **annual + usage is unsupported**.
- Uninstall auto-cancels with NO proration; access continues until period end.
- When overriding a plan's line items, interval must match the configured interval.
- Off-platform/external billing = instant App Store rejection.
