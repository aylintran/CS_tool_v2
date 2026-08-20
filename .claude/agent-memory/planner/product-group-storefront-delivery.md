---
name: product-group-storefront-delivery
description: Product-group storefront uses shop metafield + Liquid live availability, NOT the app proxy from the committed Tech Design
metadata:
  type: project
---

Product-group "combined listing" storefront rendering delivers config via the
`color_swatch.product_groups` **shop metafield** + Liquid `all_products[handle].available`
for LIVE sold-out — NOT the App Proxy described in the feature Tech Design
(`docs/phases/.../group-product-combined-listing/03-tech-design.md` §1/§3/§8/§11).

**Why:** User/PO decision (2026-06-25) — reuse the proven metafield→Liquid→Preact pipeline
already used by the "options" feature; avoid runtime API. Tech Design's `getStorefrontConfig`
bakes availability at admin-write time (stale), so it is superseded for delivery; only its
eligibility helper (`productPageIds`, SRS §3.4.1) is reused.

**How to apply:** When touching product-group storefront, do not add the app-proxy route.
Config write is best-effort after CRUD/reorder (mirrors `option-metafield.server.ts`).
Live availability is Liquid-only. Note: widget `DisplayType` uses underscores
(`drop_down_menu`) vs DB `GroupStyle` (`dropdown_menu`) — parser must remap. Collection
cards had NO mount before this work; product-group rendering is a sibling of `ProductSwatches`,
not a modification. Plan: `plans/260625-1613-product-group-storefront-render/`.
