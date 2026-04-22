# Development Guide

## Prerequisites

- Node.js ≥ 14 (LTS recommended)
- npm ≥ 7

---

## Installation

```bash
git clone <repo-url>
cd feuerwehr-untereisesheim
npm install
```

---

## npm Scripts

| Script | Command | Description |
|---|---|---|
| `npm start` | `npm-run-all --parallel css eleventy browsersync` | Starts all three dev processes in parallel |
| `npm run eleventy` | `eleventy --watch` | Watches `src/` and rebuilds HTML on change |
| `npm run css` | `tailwindcss -i src/static/css/tailwind.css -o _site/static/css/style.css --watch` | Watches `src/**/*.html` and rebuilds CSS on new classes |
| `npm run browsersync` | `browser-sync start --server _site --files _site --port 8080` | Serves `_site/` and live-reloads the browser on any file change |
| `npm run build` | `eleventy && tailwindcss -i ... -o ...` | Production build (sequential, no watch) |
| `npm run debug` | `set DEBUG=* & eleventy` | Verbose Eleventy output (Windows syntax — use `DEBUG=* npx eleventy` on macOS/Linux) |

---

## Development Server

`npm start` starts three concurrent processes:

1. **Eleventy watch** — rebuilds HTML templates whenever any source file in `src/` changes
2. **Tailwind CSS watch** — rebuilds CSS whenever a class in `src/**/*.html` appears or disappears
3. **BrowserSync** — serves `_site/` and reloads the browser whenever anything in `_site/` changes

BrowserSync defaults to port 8080 and auto-increments (8081, 8082…) if the port is taken.

---

## Known Issue: CSS Watcher

> **This is the most important operational caveat for this project.**

The Tailwind CSS `--watch` mode and Eleventy's `--watch` mode start in parallel. During the brief startup window, Tailwind scans `src/**/*.html` for class names. If Eleventy hasn't yet written all files to `_site/` — or if you add a net-new Tailwind utility class to a template — Tailwind may complete its scan before the file is fully updated, missing the new class entirely.

**Symptoms:** Styles visually absent for a specific element; inspecting the element in DevTools shows the class is present in the HTML but has no corresponding rule in `style.css`.

**Fix:** Run a one-shot rebuild manually, then save any source file to trigger a BrowserSync reload:

```bash
npx tailwindcss -i src/static/css/tailwind.css -o _site/static/css/style.css
```

This is only needed when introducing **new** utility classes. Modifying existing classes that are already in the CSS does not require a manual rebuild.

**When this happens:**
- After adding a new section or page that uses previously unused Tailwind classes
- After the first `npm install` on a fresh clone (no `_site/` exists yet)

**Permanent fix (future improvement):** Upgrade to Tailwind CSS v3.3+ and use the `--watch` flag with a polling interval, or switch to a Vite-based build that has better file-watch integration.

---

## Adding a New Homepage Section

All site content lives on a single page (`src/index.html`). To add a new section:

1. Add an anchor `<section id="your-section-id">` in `src/index.html` at the desired position in the page flow.
2. If the section uses data from a YAML file, create `src/_data/your-section.yaml` and reference it in the template as `{{ your-section.key }}`.
3. Add a nav entry in `src/_data/navigation.yaml`:
   ```yaml
   - text: Abschnitt
     url: "/#your-section-id"
   ```
4. If you introduced new Tailwind utility classes, run the manual CSS rebuild (see above).

**Section pattern used throughout the project:**
```html
<section id="section-id" class="py-20 bg-white">
  <div class="container mx-auto px-6">
    <div class="mb-12">
      <p class="text-[#CC1F1A] font-semibold text-sm uppercase tracking-widest mb-2">Kategorie</p>
      <h2 class="text-3xl lg:text-4xl font-black text-[#1A1A2E]">Überschrift</h2>
    </div>
    <!-- section content -->
  </div>
</section>
```

Alternating section backgrounds follow the pattern: `bg-white` → `bg-gray-50` → `bg-white` → `bg-[#1A1A2E]` (dark).

---

## Adding a Standalone Subpage

The site currently has one subpage: `/blog/`. To add another:

1. Create `src/your-page.html` with front matter:
   ```yaml
   ---
   layout: default
   title: Page Title
   description: Meta description for SEO
   ---
   ```
2. Eleventy will output it to `_site/your-page/index.html` → accessible at `/your-page/`.
3. Add a nav entry pointing to `/your-page` (no trailing slash needed in the YAML; Eleventy adds `index.html`).
4. If the page has a header banner, use the pattern from the old `mannschaft.html`:
   ```html
   <section class="bg-[#1A1A2E] py-20 relative overflow-hidden">
     <div class="container mx-auto px-6 relative">
       <p class="text-[#CC1F1A] font-semibold text-sm uppercase tracking-widest mb-3">Kategorie</p>
       <h1 class="text-4xl lg:text-5xl font-black text-white mb-4">Seitentitel</h1>
     </div>
   </section>
   ```

---

## Adding a New Data Field to an Existing Section

1. Add the field to the YAML file under `src/_data/`.
2. Reference it in the template with `{{ item.new_field }}`.
3. Add the field to the CMS collection in `src/admin/config.yml` so editors can manage it:
   ```yaml
   - { label: "Neues Feld", name: new_field, widget: string }
   ```

---

## Alpine.js Patterns

### Toggle (mobile nav, tabs)
```html
<div x-data="{ open: false }">
  <button @click="open = !open">Toggle</button>
  <div x-show="open" x-transition>Content</div>
</div>
```

### Scroll-triggered visibility
```html
<div x-data="{ show: false }" @scroll.window="show = window.scrollY > 300">
  <div x-show="show">Visible after 300px scroll</div>
</div>
```

### Tab switcher (Mannschaft section)
```html
<div x-data="{ activeTab: 'first' }">
  <button @click="activeTab = 'first'" :class="activeTab === 'first' ? 'active-styles' : 'inactive-styles'">Tab 1</button>
  <div x-show="activeTab === 'first'">Content 1</div>
  <div x-show="activeTab === 'second'">Content 2</div>
</div>
```

**Important:** Alpine.js is loaded at the **end of `<body>`** in `default.html`. All `x-data` elements must be present in the DOM before the script tag. This is the correct pattern — do not move the script to `<head>` without adding `defer`.

---

## Local CMS Development

To run Decap CMS locally (without connecting to Netlify), use the local proxy server:

```bash
npx decap-server
```

Run this in a separate terminal alongside `npm start`. The CMS at `http://localhost:8081/admin/` will read and write files directly from your local filesystem instead of committing to Git.

`local_backend: true` in `src/admin/config.yml` enables this mode. Remove or comment it out before deploying if you want to prevent unauthenticated local access on staging.

---

## Syntax Highlighting

Blog posts support fenced code blocks with syntax highlighting via Prism.js. To enable it in a layout, add `prism: true` to the front matter:
```yaml
---
prism: true
---
```

The `posts.html` layout sets this automatically for all posts. The CSS is loaded conditionally in `default.html`:
```html
{% if prism == true %}
  <link rel="stylesheet" href="/static/css/prism-tomorrow.css">
{% endif %}
```
