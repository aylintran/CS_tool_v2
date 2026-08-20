---
name: storefront-metafield-pipeline
description: How storefront features deliver data to the Preact widget (metafield, not app proxy)
metadata:
  type: project
---

Storefront features in this app deliver data WITHOUT an app proxy.

Pipeline: admin Save → shop metafield `color_swatch.<key>` (JSON, via `writeSwatchMetafield`
in app/services/swatch-style/swatch-metafield.server.ts, keyed off `GET_SHOP_INFO` shop id) →
theme Liquid block `extensions/shopify-theme-extension/blocks/theme-code-block.liquid` injects
into `window.AppSettings.<key>` → Preact widget reads it fail-safe.

Live inventory (sold-out/deleted) is resolved at render time in Liquid via
`all_products[handle].available`, not baked into the metafield. A handle missing from the
injected availability map = deleted product.

**Why:** Same proven pattern as the "options" feature; avoids app-proxy latency/auth.
**How to apply:** When reviewing storefront features, expect config-in-metafield + live-data-in-Liquid;
metafield writes are best-effort (try/catch, never block admin mutation). The shared `DisplayType`
enum (formerly GroupStyle) uses underscore values (`drop_down_menu`) identical on DB and widget —
no remap needed despite some stale plan docs claiming a mismatch.
