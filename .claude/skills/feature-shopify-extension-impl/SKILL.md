---
name: feature-shopify-extension-impl
description: Implement and deploy Shopify app extensions — theme app extensions (storefront blocks/embeds), checkout UI extensions, admin action/block extensions, and Shopify Functions (discounts, cart/checkout validation) — including shopify.extension.toml config, target/extension-point selection, file structure under extensions/<name>/, the Preact storefront-widget build pipeline used in this repo, and the shopify app dev / deploy / versioning workflow. Use this skill EVERY TIME a user asks to "build a theme app extension", "add a storefront block", "create a checkout UI extension", "add an admin action/block", "write a Shopify Function", "discount function", "cart validation function", "deploy the extension", "viết extension", "thêm block storefront", "tạo checkout extension", "function giảm giá", "deploy extension", or whenever code must run inside the merchant's storefront, checkout, admin, or as a Function. Default stack: Preact for storefront widget (this repo) + @shopify/ui-extensions for checkout/admin + Rust/Wasm for Functions. Builds run from repo root (pnpm w:build / shopify app deploy).
---

# Feature Shopify Extension Implementation (v1)

Guideline for building the 4 extension families + deploying them the Shopify way.

## Core principle

> **Extensions are sandboxed surfaces, each declared by a `shopify.extension.toml` with a `type` + `target`. Code lives under `extensions/<name>/`. In THIS repo, storefront code is authored in `packages/storefront-widget/src/` and built (Vite IIFE) into `extensions/shopify-theme-extension/assets/app-extension.js` — NEVER hand-edit the built asset. Extensions ship only via `shopify app deploy`, which snapshots an immutable app version.**

## Verify via MCP first (REQUIRED)

Before coding any extension, confirm target/extension-point names, TOML keys, and component APIs via `shopify-dev-mcp`: `learn_shopify_api(api: "functions" | "polaris-checkout-extensions" | "polaris-admin-extensions" | "use-shopify-cli")` → `search_docs_chunks(conversationId, "<query>")`. Targets + APIs change per version — don't guess.

## Pick the right surface first

| Surface | `type` | Example target | When |
|---|---|---|---|
| Theme app extension | (theme structure) | app block / app embed (liquid) | Inject UI into storefront pages/theme |
| Checkout UI extension | `ui_extension` | `purchase.checkout.block.render` | UI in checkout (banners, fields, upsell) |
| Admin action/block | `ui_extension` | `admin.product-details.action.render` | UI inside Shopify admin pages |
| Shopify Function | `function` | `cart.lines.discounts.generate.run` | Server-side logic (discounts, validation) — pure WASM |

## A. Theme app extension (this repo: `extensions/shopify-theme-extension/`)

```
extensions/<name>/
├── assets/      # built JS/CSS → CDN. In this repo: app-extension.js is OUTPUT (don't edit)
├── blocks/      # app-block.liquid (in-section) + app-embed-block.liquid (page-level)
├── snippets/    # reusable liquid
├── locales/     # en.default.json + en.default.schema.json
└── shopify.extension.toml
```
- Each block has ONE `{% schema %}` (valid JSON, no comments/trailing commas).
- App blocks added to theme sections as `{ "type": "@app" }`. No access to `content_for_*`; parent section exposes only `id`.
- **This repo's pipeline**: edit `packages/storefront-widget/src/` → `pnpm w:build` writes the IIFE bundle to the theme extension asset. `pnpm w:dev` for standalone dev. See `packages/storefront-widget/README.md`.

## B. Checkout UI extension (`type = "ui_extension"`)

Current (2025) path = **Preact + web components**:
```ts
import '@shopify/ui-extensions/preact';
import { render } from 'preact';
export default async () => render(<Extension />, document.body);
// components: <s-banner>, <s-stack>, <s-text-field>; global `shopify` (shopify.i18n, shopify.applyCartLinesChange, shopify.instructions)
```
Legacy React still valid: `reactExtension('purchase.checkout.block.render', () => <Extension/>)` from `@shopify/ui-extensions-react/checkout`.
- Common targets: `purchase.checkout.block.render`, `...delivery-address.render-before`, `...shipping-option-list.render-before/-after`, `...payment-option-item.details.render`.
- `[extensions.capabilities]`: `api_access`, `block_progress`, `network_access`, `collect_buyer_consent`. `[extensions.settings.fields]` for merchant config.

## C. Admin action/block (`type = "ui_extension"`)

- Action: `admin.product-details.action.render` (also `admin.product.item.action.render`). Block: `admin.product.item.block.render`. Config: `admin.product-details.configuration.render`.
- **Exactly ONE target per admin action/block extension.**

## D. Shopify Functions (`type = "function"`)

```toml
api_version = "2025-07"
[[extensions]]
type = "function"
handle = "my-discount-function"
[[extensions.targeting]]
target = "cart.lines.discounts.generate.run"
input_query = "src/run.graphql"   # MUST NOT be named input.graphql
export = "run"
[extensions.build]
command = "cargo build --target=wasm32-unknown-unknown --release"
path = "target/wasm32-unknown-unknown/release/discount.wasm"
watch = ["src/**/*.rs"]
```
- Files: `src/<fn>.graphql` (operation name MUST be `Input`), `src/<fn>.rs`, `src/main.rs` (Rust: `#[typegen("./schema.graphql")]`), `schema.graphql` (pull via `shopify app function schema`).
- Functions are **PURE**: no network/filesystem/RNG/clock. All data via input query (camelCase; unions need `__typename`).
- Output type matches the run target's result (e.g. `CartLinesDiscountsGenerateRunResult`).
- Discount targets (current): `cart.lines.discounts.generate.run`, `cart.delivery-options.discounts.generate.run` (+ `.fetch`).
- Test: `shopify app function run --input=input.json --export=run`.

## Scaffold / dev / deploy

- Scaffold: `shopify app generate extension --template <api> --flavor <rust|typescript|vanilla-js>`.
- Dev: `shopify app dev` (tunnels + hot-reload; **restart after TOML edits**).
- Deploy: `shopify app deploy` — bundles all extensions + config into a new immutable **app version**, releases to merchants. Rollback = re-release a prior version. This repo: `pnpm deploy` = `w:build` then `shopify app deploy`.
- Validate config: `shopify app config validate --json`.

## GOTCHAS

- Don't hand-edit `extensions/shopify-theme-extension/assets/app-extension.js` — it's build OUTPUT (overwritten by `pnpm w:build`).
- Function input query file must NOT be `input.graphql`; Rust operation name must be `Input`.
- Rust functions: no external crates beyond `shopify_function`; unwrap Optional GraphQL fields; `Decimal::from` takes floats.
- Admin action/block = exactly one target each.
- `api_version` change in TOML requires redeploy; per-`[[extensions]]` overrides root.
- TOML/extension changes sync on `deploy`, not always reflected until dev-server restart.
- Web-components Preact is the current checkout path; old React API works but is being migrated.
