# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Shopify theme based on **Dawn v15.5.0** (Shopify's reference theme). No build system — all JavaScript and CSS are plain files served directly from Shopify's CDN.

## Development Commands

Requires [Shopify CLI](https://shopify.dev/docs/themes/tools/cli) installed and authenticated.

```bash
# Start local dev server with hot reload (proxies your store)
shopify theme dev --store=your-store.myshopify.com

# Push theme to store
shopify theme push --store=your-store.myshopify.com

# Pull latest theme from store
shopify theme pull --store=your-store.myshopify.com

# Lint theme files
shopify theme check
```

## Architecture

### Directory Structure

| Directory | Purpose |
|-----------|---------|
| `layout/` | Root HTML shell (`theme.liquid` wraps every page; `password.liquid` for locked stores) |
| `templates/` | JSON files declaring which sections appear on each page type |
| `sections/` | Page-level Liquid components — each loads its own CSS/JS inline |
| `snippets/` | Reusable Liquid partials rendered via `{% render %}` |
| `assets/` | Plain JS (Custom Elements) and CSS files |
| `config/` | `settings_schema.json` defines Theme Editor controls; `settings_data.json` stores current values |
| `locales/` | Translation strings — `en.default.json` is the source of truth |

### JavaScript Patterns

All JS uses native **Custom Elements** (Web Components) — no framework, no bundler. Key files loaded globally via `layout/theme.liquid`:

- **`constants.js`** — Shared constants: `PUB_SUB_EVENTS` event names, `ON_CHANGE_DEBOUNCE_TIMER`
- **`pubsub.js`** — Lightweight pub/sub (`subscribe(eventName, callback)` / `publish(eventName, data)`). Used for cross-component communication (e.g., cart updates triggering drawer refresh)
- **`global.js`** — Utility classes: `SectionId` (parses Shopify's `template--XXXX__sectionname` IDs), `HTMLUpdateUtility.viewTransition()` (double-buffer DOM swap for section re-renders)
- **`standard-actions-override.js`** — Integrates Shopify's `StandardEvents` storefront analytics

Section-specific JS (e.g., `product-info.js`, `cart.js`) is loaded lazily within the section's own Liquid file, not in `theme.liquid`.

### CSS Pattern

CSS is split per component (e.g., `component-card.css`, `component-price.css`). Sections inline their own `{{ 'component-foo.css' | asset_url | stylesheet_tag }}` calls at the top of the Liquid file — CSS loads only when the section is rendered.

### Cart System

Cart uses Shopify's AJAX Cart API (`/cart/update.js`, `/cart/add.js`, `/cart.js`). The pub/sub event `PUB_SUB_EVENTS.cartUpdate` coordinates updates between `cart-items`, `cart-drawer`, and `cart-notification` custom elements.

### Localization

All user-facing strings must use translation keys. Add new strings to `locales/en.default.json` first; the `.schema.json` variants hold Theme Editor label translations.

### Theme Settings

`config/settings_schema.json` defines the controls available in Shopify's Theme Editor (colors, fonts, etc.). Reference settings in Liquid as `{{ settings.setting_id }}`. Color schemes are defined as a `color_scheme_group` and applied per-section via `color-{{ section.settings.color_scheme }}` CSS classes.
