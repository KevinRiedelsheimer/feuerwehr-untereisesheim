# Architecture

## Build Pipeline Overview

```
src/**/*.html  ─┐
src/**/*.md    ─┤──► Eleventy 1.x ──► _site/**/*.html   (minified)
src/_data/*.yaml─┘
                         │
src/static/css/          │
  tailwind.css  ────► Tailwind CLI ──► _site/static/css/style.css
                         │
node_modules/            │
  alpinejs/     ─────────┴──► _site/static/js/alpine.js  (passthrough copy)
  prismjs/      ──────────────► _site/static/css/prism-tomorrow.css
```

Eleventy and Tailwind are **independent processes** that run in parallel during development (`npm start` uses `npm-run-all --parallel`). They share no direct coupling — Eleventy writes HTML, Tailwind generates CSS that the HTML links to.

---

## Eleventy Configuration (`.eleventy.js`)

Key decisions in the Eleventy config:

### Input directory
```js
return { dir: { input: "src" } };
```
All source files are under `src/`. The output directory defaults to `_site`.

### HTML files are processed as Nunjucks
```js
htmlTemplateEngine: "njk"
```
Files with `.html` extensions are compiled as Nunjucks templates. This means `.html` and `.njk` are interchangeable — the project uses `.html` throughout.

### YAML data extension
```js
eleventyConfig.addDataExtension("yaml", (contents) => yaml.load(contents));
```
Enables `_data/*.yaml` files. Without this, Eleventy only natively reads `.json` and `.js` data files.

### Passthrough copies
| Source | Destination | Notes |
|---|---|---|
| `src/admin/config.yml` | `_site/admin/config.yml` | CMS config |
| `node_modules/alpinejs/dist/cdn.min.js` | `_site/static/js/alpine.js` | Alpine.js served locally |
| `node_modules/prismjs/themes/prism-tomorrow.css` | `_site/static/css/prism-tomorrow.css` | Syntax highlighting |
| `src/static/img/` | `_site/static/img/` | Media uploads |
| `src/favicon.ico` | `_site/favicon.ico` | Favicon |

### HTML minification
All output HTML is minified via `html-minifier` (collapseWhitespace, removeComments, useShortDoctype). This means the built HTML is a single line — do not grep built files for debugging, use the source files.

### Custom filters
| Filter | Usage | Implementation |
|---|---|---|
| `readableDate` | `{{ date \| readableDate }}` | Formats a JS Date via Luxon: `"dd LLL yyyy"` (e.g. `"12 Apr 2025"`) |

---

## Template Inheritance

```
src/_includes/default.html          ← base layout
│  - <html lang="de">
│  - Google Fonts (Inter)
│  - emergency bar
│  - {% include partials/navbar.html %}
│  - {{ content | safe }}           ← page body injected here
│  - {% include partials/footer.html %}
│  - scroll-to-top button (Alpine.js)
│  - /static/js/alpine.js
│
├── src/index.html                  (layout: default, path: home)
│       All homepage sections rendered inline:
│       Hero → Stats → Aktuelles → Termine → Mannschaft →
│       Fahrzeuge → Brandschutz → Kontakt
│
├── src/blog/index.html             (layout: blog)
│       src/_includes/blog.html     (layout: default)
│
└── src/posts/*.md                  (layout: posts — via posts.json)
        src/_includes/posts.html    (layout: default)
```

Front matter in every template specifies its layout:
```yaml
---
layout: default
title: Page Title
---
```

The `path: home` front matter variable is used in `default.html` to conditionally load the Netlify Identity widget (required for CMS authentication on the homepage only).

---

## Data Cascade

Eleventy's global data directory (`src/_data/`) makes each YAML file available to all templates under the filename as the variable name:

| File | Template variable | Example access |
|---|---|---|
| `settings.yaml` | `settings` | `{{ settings.name }}` |
| `navigation.yaml` | `navigation` | `{% for item in navigation.items %}` |
| `events.yaml` | `events` | `{% for event in events.events %}` |
| `vehicles.yaml` | `vehicles` | `{% for vehicle in vehicles.vehicles %}` |
| `mannschaft.yaml` | `mannschaft` | `{% for section in mannschaft.sections %}` |
| `quicklinks.yaml` | `quicklinks` | `{{ quicklinks.links }}` |

Note the double-key pattern: the filename **and** the top-level YAML key share the same name (e.g. `events.yaml` contains `events: [...]`, accessed as `events.events`). This is intentional — it keeps the template variable names unambiguous.

---

## CSS Generation

### Tailwind CSS v3 (JIT mode)

Tailwind v3 uses JIT (Just-in-Time) compilation by default. It scans content files for class names and only emits CSS for classes that are actually used.

**Content paths** (`tailwind.config.js`):
```js
content: ["./src/**/*.html"]
```

Only source files are scanned — not `_site/`. This avoids a race condition where Tailwind would scan the built output before Eleventy had finished writing it.

**Entry point** (`src/static/css/tailwind.css`):
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html { scroll-behavior: smooth; }
  body { font-family: "Inter", system-ui, sans-serif; }
}
```

**Output:** `_site/static/css/style.css` — the only CSS file loaded by the site.

### Arbitrary values

The design uses Tailwind arbitrary values extensively for the brand colors rather than config extensions:
- `bg-[#CC1F1A]` — primary fire red
- `bg-[#1A1A2E]` — dark charcoal (hero, navbar, footer, brandschutz)
- `bg-[#252542]` — lighter charcoal (stats bar)

These are valid Tailwind v3 JIT syntax and are picked up by the content scanner as long as the class strings appear verbatim in source files. **Never construct these dynamically** (e.g. `` `bg-[${color}]` ``) — the scanner won't find them.

### PostCSS

`postcss.config.js` runs Tailwind and Autoprefixer:
```js
plugins: { tailwindcss: {}, autoprefixer: {} }
```

`postcss-cli` is present as a dev dependency but the active dev script now uses the `tailwindcss` CLI directly (see [development.md](./development.md)).

---

## Alpine.js Runtime

Alpine.js v3 is loaded from a local copy (`/static/js/alpine.js`, copied from `node_modules` at build time). It is **not** loaded from a CDN, ensuring offline builds and eliminating external dependency at runtime.

Alpine.js handles three UI behaviours:

| Component | Location | Behaviour |
|---|---|---|
| Mobile nav menu | `partials/navbar.html` | `x-data="{ isOpen: false }"` — toggles the mobile drawer |
| Mannschaft tabs | `src/index.html` | `x-data="{ activeTab: 'aktive' }"` — switches visible section |
| Scroll-to-top button | `src/_includes/default.html` | `@scroll.window` — shows button after 300 px scroll |

All Alpine directives use v3 syntax. Notably, `@click.outside` replaces the v2 `@click.away` for closing menus — the navbar uses `@keydown.escape` instead.
