---
name: liquid-all-products-20-handle-cap
description: Shopify Liquid all_products[handle] is capped at 20 unique handles/page — do not use it to build per-card data on collection pages
metadata:
  type: project
---

Shopify Liquid `all_products[handle]` resolves at most **20 unique handles per page** (platform limit, https://shopify.dev/docs/api/liquid/objects/all_products, Shopify/liquid issue #1283).

**Why:** The existing product-groups injection in `extensions/shopify-theme-extension/blocks/theme-code-block.liquid` (`productGroupsAvailability`/`productGroupsPrices`) uses `all_products[handle]` and works ONLY because a group has few items. A collection page has up to 50 cards.

**How to apply:** For per-card product/variant data on a COLLECTION template, iterate `collection.products` (current view, respects pagination/filters, gives full `.options_with_values`/`.variants`) instead of `all_products[handle]`. Liquid for-loops cap at 50 iterations/page, which matches typical card counts. Guard with `{% if collection %}` (app embed targets body → runs everywhere). Verify card #21+ has data as the regression check. See [[collection-page-option-swatch]] plan Phase A.
