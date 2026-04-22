# Feuerwehr Untereisesheim — Technical Documentation

Technical reference for engineers maintaining or extending the website. For content editing instructions, use the CMS at `/admin`.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Quick Start](#quick-start)
3. [Directory Structure](#directory-structure)
4. [Further Reading](#further-reading)

---

## Tech Stack

| Technology | Version | Role |
|---|---|---|
| [Eleventy (11ty)](https://www.11ty.dev/) | ^1.0.0 | Static site generator |
| [Tailwind CSS](https://tailwindcss.com/) | ^3.0.13 | Utility-first CSS framework |
| [Alpine.js](https://alpinejs.dev/) | ^3.7.1 | Lightweight reactive JS |
| [Decap CMS](https://decapcms.org/) | ^3.0.0 (CDN) | Git-backed headless CMS |
| [Nunjucks](https://mozilla.github.io/nunjucks/) | (via Eleventy) | HTML templating language |
| [Luxon](https://moment.github.io/luxon/) | ^2.3.0 | Date formatting |
| [BrowserSync](https://browsersync.io/) | ^2.27.7 | Dev server with live reload |
| [html-minifier](https://github.com/kangax/html-minifier) | ^4.0.0 | HTML minification at build time |

All source files live in `src/`. Eleventy outputs the built site to `_site/`, which is the Netlify publish directory.

---

## Quick Start

**Prerequisites:** Node.js ≥ 14, npm.

```bash
npm install
npm start
```

The site is served at **http://localhost:8081** (BrowserSync auto-assigns the port if 8080 is taken).

> **First-run CSS note:** On the very first start after adding new Tailwind utility classes, the watcher may miss them. Run the one-shot rebuild once, then restart:
> ```bash
> npx tailwindcss -i src/static/css/tailwind.css -o _site/static/css/style.css
> ```
> See [development.md](./development.md#known-issue-css-watcher) for the full explanation.

---

## Directory Structure

```
feuerwehr-untereisesheim/
│
├── src/                          # All source files (Eleventy input)
│   ├── index.html                # Homepage (single-page layout, all sections)
│   ├── blog/
│   │   └── index.html            # Blog index page
│   ├── posts/                    # Markdown blog posts + collection config
│   │   ├── posts.json            # Front matter defaults for all posts
│   │   └── *.md                  # Individual posts
│   ├── admin/
│   │   ├── index.html            # Decap CMS entry point (loads CDN script)
│   │   └── config.yml            # CMS collection + field definitions
│   ├── _includes/                # Eleventy layouts and partials
│   │   ├── default.html          # Base layout (emergency bar, nav, footer, scroll button)
│   │   ├── blog.html             # Blog index layout
│   │   ├── posts.html            # Individual post layout
│   │   └── partials/
│   │       ├── navbar.html       # Sticky top navigation (Alpine.js mobile menu)
│   │       ├── footer.html       # Site footer (dark, 3-column)
│   │       └── content.html      # Legacy quick-links partial (unused on homepage)
│   ├── _data/                    # Global data files (auto-loaded by Eleventy)
│   │   ├── settings.yaml         # Site name, author, URL
│   │   ├── navigation.yaml       # Nav link items
│   │   ├── events.yaml           # Upcoming events (Termine section)
│   │   ├── vehicles.yaml         # Fire vehicles (Fahrzeuge section)
│   │   ├── mannschaft.yaml       # Team members by section (Mannschaft section)
│   │   └── quicklinks.yaml       # Legacy quick links (CMS-managed, unused on homepage)
│   └── static/
│       ├── css/
│       │   └── tailwind.css      # Tailwind CSS entry point (@tailwind directives)
│       └── img/                  # Media uploads (passthrough to _site)
│
├── _site/                        # Build output — DO NOT edit directly
│
├── documentation/                # This folder
│   ├── README.md                 # Overview (this file)
│   ├── architecture.md           # Build pipeline and system design
│   ├── development.md            # Dev workflow, scripts, known issues, how-tos
│   ├── content-and-cms.md        # Data schemas, CMS configuration
│   └── deployment.md             # Netlify deployment and CMS auth
│
├── .eleventy.js                  # Eleventy configuration (filters, plugins, passthrough)
├── tailwind.config.js            # Tailwind theme + content paths
├── postcss.config.js             # PostCSS plugins (tailwindcss + autoprefixer)
├── netlify.toml                  # Netlify build settings
└── package.json                  # npm scripts and dependencies
```

---

## Further Reading

| Document | Contents |
|---|---|
| [architecture.md](./architecture.md) | Build pipeline, template inheritance, data cascade, CSS generation |
| [development.md](./development.md) | npm scripts, dev server, CSS watcher quirk, adding sections/pages |
| [content-and-cms.md](./content-and-cms.md) | YAML data schemas, CMS collections, media handling |
| [deployment.md](./deployment.md) | Netlify config, Decap CMS auth, environment setup |
| [github-pages.md](./github-pages.md) | GitHub Actions workflow, custom domain setup, CMS on GitHub Pages |
