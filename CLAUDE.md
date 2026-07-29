# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Shopify theme based on **Dawn v15.5.0** (Shopify's reference theme), customized for the **Rainbowsmile** brand (oral/gut health products). No build system — all JavaScript and CSS are plain files served directly from Shopify's CDN.

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

### Two Section Patterns

The codebase has two distinct section patterns — don't mix them up:

**1. Dawn sections** (e.g., `main-product.liquid`, `featured-collection.liquid`): load external CSS assets at the top via `{{ 'component-foo.css' | asset_url | stylesheet_tag }}`, use `color-{{ section.settings.color_scheme }}` CSS classes, and draw settings from `config/settings_schema.json` color schemes.

**2. Custom Rainbowsmile sections** (e.g., `hero-main.liquid`, `sticky-atc.liquid`, `comparison-table.liquid`, `trust-bar.liquid`, etc.): embed all CSS directly in a `{%- style -%}` block at the top of the file, scoped to `#shopify-section-{{ section.id }}` and a local BEM class like `.hero-{{ section.id }}`. Colors are exposed as CSS custom properties (e.g., `--hro-coral`, `--satc-bg`) set from `section.settings.*` color pickers defined in the section's own `{% schema %}`.

### Custom Sections (Rainbowsmile-specific)

These sections do not exist in Dawn and were built for this brand:

| Section | Purpose |
|---------|---------|
| `hero-main.liquid` | Homepage hero with background image, overlay, and coral accent curve |
| `sticky-atc.liquid` | Fixed bottom add-to-cart bar, slides in on scroll |
| `comparison-table.liquid` | Product comparison table with check/cross icons |
| `trust-bar.liquid` | Icon + text trust badges strip |
| `testimonials.liquid` | Customer review cards |
| `reviews-slider.liquid` | Scrolling review slider |
| `ingredients-band.liquid` | Horizontal ingredients highlight band |
| `how-it-works.liquid` | Step-by-step process section |
| `how-to-use.liquid` / `how-to-use-drawer.liquid` | Usage instructions with optional drawer |
| `science-hook.liquid` | Clinical/scientific credibility section |
| `oral-gut-care.liquid` / `oral-gut-axis.liquid` | Brand education sections |
| `community-*.liquid` | Community hub sections (hero, stats, feed, AMA, education, forum) |
| `faq-accordion.liquid` / `faq-banner.liquid` / `faq-groups.liquid` | FAQ sections |
| `phrase-marquee.liquid` | Scrolling text marquee |
| `founder.liquid` | Founder story section |
| `rs-mission.liquid` / `rs-contact.liquid` | Brand mission and contact sections |
| `biome-hacker.liquid` | Brand concept section |
| `audio-story.liquid` | Audio content embed |
| `free-book.liquid` / `book-hero.liquid` | Lead magnet book sections |

### Custom Page Templates

Beyond Dawn's defaults, these page-specific templates exist:

- `templates/page.about.json` — About page
- `templates/page.contact.json` — Contact page
- `templates/page.faqs.json` — FAQs page
- `templates/page.community.json` — Community hub page

### JavaScript Patterns

All JS uses native **Custom Elements** (Web Components) — no framework, no bundler. Key files loaded globally via `layout/theme.liquid`:

- **`constants.js`** — Shared constants: `PUB_SUB_EVENTS` event names, `ON_CHANGE_DEBOUNCE_TIMER`
- **`pubsub.js`** — Lightweight pub/sub (`subscribe(eventName, callback)` / `publish(eventName, data)`). Used for cross-component communication (e.g., cart updates triggering drawer refresh)
- **`global.js`** — Utility classes: `SectionId` (parses Shopify's `template--XXXX__sectionname` IDs), `HTMLUpdateUtility.viewTransition()` (double-buffer DOM swap for section re-renders)
- **`standard-actions-override.js`** — Integrates Shopify's `StandardEvents` storefront analytics

Section-specific JS (e.g., `product-info.js`, `cart.js`) is loaded lazily within the section's own Liquid file, not in `theme.liquid`.

### CSS Pattern

**Dawn components**: CSS is split per component (e.g., `component-card.css`, `component-price.css`). Sections inline their own `{{ 'component-foo.css' | asset_url | stylesheet_tag }}` calls at the top.

**Custom sections**: All CSS lives in an inline `{%- style -%}` block — no separate asset file. CSS custom properties are set per-section using `section.settings.*` values, enabling full Theme Editor control without touching asset files.

### Cart System

Cart uses Shopify's AJAX Cart API (`/cart/update.js`, `/cart/add.js`, `/cart.js`). The pub/sub event `PUB_SUB_EVENTS.cartUpdate` coordinates updates between `cart-items`, `cart-drawer`, and `cart-notification` custom elements.

### Localization

All user-facing strings must use translation keys. Add new strings to `locales/en.default.json` first; the `.schema.json` variants hold Theme Editor label translations.

### Theme Settings

`config/settings_schema.json` defines the controls available in Shopify's Theme Editor (colors, fonts, etc.). Reference settings in Liquid as `{{ settings.setting_id }}`. Color schemes are defined as a `color_scheme_group` and applied per-section via `color-{{ section.settings.color_scheme }}` CSS classes. Custom sections bypass this system and define their own color pickers directly in their `{% schema %}`.
