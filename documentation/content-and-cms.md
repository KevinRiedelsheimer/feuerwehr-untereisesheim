# Content & CMS

## Data Files

All structured content is stored as YAML in `src/_data/`. Eleventy loads every file in this directory and exposes it to templates under the filename (without extension) as a global variable.

---

### `settings.yaml`

Site-wide metadata.

```yaml
name: Feuerwehr Untereisesheim
author: Feuerwehr Untereisesheim
url: "https://feuerwehr-untereisesheim.de"
```

| Field | Template usage | Purpose |
|---|---|---|
| `name` | `{{ settings.name }}` | Footer copyright line, page titles |
| `author` | `{{ settings.author }}` | Meta |
| `url` | `{{ settings.url }}` | Canonical URL |

---

### `navigation.yaml`

Drives both the desktop and mobile navigation menus in `partials/navbar.html` and the footer link list.

```yaml
items:
  - text: Aktuelles
    url: "/#aktuelles"
  - text: Termine
    url: "/#termine"
  - text: Mannschaft
    url: "/#mannschaft"
  - text: Fahrzeuge
    url: "/#fahrzeuge"
  - text: Brandschutz
    url: "/#brandschutz"
  - text: Kontakt
    url: "/#kontakt"
```

`url` values use `/#anchor` for same-page sections and `/page` for subpages. Smooth scrolling is handled by `scroll-behavior: smooth` on the `<html>` element in `tailwind.css`.

> **CMS note:** The Navigation collection in `admin/config.yml` has `allow_add: false`, preventing editors from adding new nav items. This is intentional — nav structure changes require a developer.

---

### `events.yaml`

Upcoming events rendered in the Termine section.

```yaml
events:
  - title: Maibaumaufstellung
    day: "24"
    month: "Apr"
    time: "18:00 Uhr"
    type: Veranstaltung
    description: Gemeinsame Maibaumaufstellung mit der Ortsgemeinde
```

| Field | Type | Notes |
|---|---|---|
| `title` | string | Event name |
| `day` | string | Day of month (quoted to preserve leading zeros) |
| `month` | string | 3-letter German abbreviation (Jan, Feb, Mär, Apr, Mai, Jun, Jul, Aug, Sep, Okt, Nov, Dez) |
| `time` | string | Time string, e.g. `"19:30 Uhr"` |
| `type` | string | Displayed as a badge: `Übung`, `Veranstaltung`, etc. |
| `description` | string | One-line description |

Date parts are stored split (not as an ISO date) because the template renders them separately in the date badge without needing a date filter.

---

### `vehicles.yaml`

Vehicle fleet rendered in the Fahrzeuge section.

```yaml
vehicles:
  - name: HLF 20
    type: Hilfeleistungs-Löschgruppenfahrzeug
    description: Unser leistungsstärkstes Fahrzeug…
    year: 2018
```

| Field | Type | Notes |
|---|---|---|
| `name` | string | Short designation (e.g. `HLF 20`) |
| `type` | string | Full type name displayed as subheading |
| `description` | string | One or two sentence description |
| `year` | integer | Year of commissioning — displayed as "In Dienst seit YYYY" |

---

### `mannschaft.yaml`

Team members organised into sections with Alpine.js tab switching.

```yaml
sections:
  - id: aktive
    title: Aktive Mannschaft
    members:
      - name: Vorname Nachname
        role: Kommandant
        image: ""
```

| Field | Type | Notes |
|---|---|---|
| `sections[].id` | string | Used as the Alpine.js `activeTab` identifier. Must be a valid JS identifier (no spaces, no special chars). |
| `sections[].title` | string | Tab button label and count badge |
| `members[].name` | string | Full name |
| `members[].role` | string | Function/rank, displayed in fire-red |
| `members[].image` | string | Path relative to `public_folder` (e.g. `/static/img/name.jpg`). Empty string shows placeholder. |

**Image handling:** Images are uploaded via CMS to `src/static/img/`. Eleventy copies this directory to `_site/static/img/` as a passthrough. The CMS `public_folder` is set to `/static/img`, so image paths in YAML resolve correctly in both development and production.

---

### `quicklinks.yaml`

Legacy data file from the NEAT starter template. Still managed by the CMS but not rendered on the homepage. Can be repurposed or removed.

---

## Blog Posts

Blog posts live in `src/posts/` as Markdown files. `posts.json` in the same directory applies shared front matter to all posts:

```json
{
  "layout": "posts",
  "tags": ["post"]
}
```

The `tags: ["post"]` makes each post part of the `collections.post` collection, which the homepage Aktuelles section and the blog index iterate over.

**Per-post front matter:**
```yaml
---
title: Post title
description: Short excerpt shown on card
author: Name
date: 2025-04-01
tags:
  - Einsatz
  - Übung
---
```

The homepage Aktuelles section shows the **3 most recent posts** (sorted by `date` descending via `| reverse` + `loop.index0 < 3`).

---

## Decap CMS Configuration

The CMS is configured in `src/admin/config.yml`. The built copy at `_site/admin/config.yml` is a passthrough — always edit the source.

### Backend

```yaml
backend:
  name: git-gateway
  branch: master
```

`git-gateway` is Netlify's managed Git proxy. It authenticates users via Netlify Identity and commits changes back to the repository on their behalf. The CMS writes directly to the files listed in each collection's `file:` or `folder:` field.

### Collections overview

| Collection | Type | Managed file(s) |
|---|---|---|
| Blog | folder | `src/posts/*.md` |
| Settings › Navigation | file | `src/_data/navigation.yaml` |
| Settings › Quick Links | file | `src/_data/quicklinks.yaml` |
| Settings › Meta Settings | file | `src/_data/settings.yaml` |
| Mannschaft | file | `src/_data/mannschaft.yaml` |

### Adding a new CMS-managed data file

1. Create the YAML file in `src/_data/`.
2. Add a `files` entry under an existing collection or create a new collection in `src/admin/config.yml`:

```yaml
- label: "Neue Sammlung"
  name: "neue_sammlung"
  editor:
    preview: false
  files:
    - label: "Daten"
      name: "daten"
      file: "src/_data/neue_daten.yaml"
      fields:
        - { label: Titel, name: title, widget: string }
        - { label: Beschreibung, name: description, widget: text }
```

3. The Decap CMS admin will reflect the change on next page load (no rebuild required for config changes — the CMS reads `config.yml` at runtime).

### Available widgets

Commonly used Decap CMS widgets for this project:

| Widget | Usage |
|---|---|
| `string` | Single-line text |
| `text` | Multi-line plain text |
| `markdown` | Rich text editor (blog post body) |
| `datetime` | Date + time picker |
| `number` | Integer or float |
| `image` | File upload to `media_folder` |
| `select` | Dropdown with fixed `options` |
| `list` | Repeating group of fields |
| `boolean` | Toggle |

Full reference: https://decapcms.org/docs/widgets/

---

## Media / Images

**Upload path:** `src/static/img/`
**Public path:** `/static/img/`

Images uploaded via the CMS are stored in `src/static/img/` and committed to the repository. Eleventy copies the entire directory to `_site/static/img/` unchanged (passthrough copy — no optimisation).

**For future improvement:** Add an image optimisation step (e.g. `@11ty/eleventy-img`) to generate responsive image sets and WebP variants.
