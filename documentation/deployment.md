# Deployment

## Hosting: Netlify

The site is deployed on Netlify via continuous deployment from the Git repository.

### `netlify.toml`

```toml
[build]
  publish = "_site"
  command = "npm run build"
```

| Setting | Value | Notes |
|---|---|---|
| `publish` | `_site` | Directory Netlify serves after build |
| `command` | `npm run build` | Runs Eleventy then Tailwind CSS (sequential) |

---

## Production Build

`npm run build` runs two commands sequentially:

```bash
cross-env NODE_ENV=production eleventy
cross-env NODE_ENV=production tailwindcss -i src/static/css/tailwind.css -o _site/static/css/style.css
```

**Why sequential?** Tailwind must run *after* Eleventy so that `_site/` exists and all HTML is written. In development (`npm start`) they run in parallel because the Tailwind watcher handles the race by rescanning on every file change. In production there is no watcher — the one-shot Tailwind build needs the HTML to already be present.

`NODE_ENV=production` is set but has no effect on the current build config (no environment-specific Tailwind or Eleventy settings are defined). It is retained for future use (e.g. enabling cssnano minification in `postcss.config.js`).

---

## Decap CMS Authentication

Decap CMS uses Netlify's `git-gateway` backend, which requires two Netlify features to be enabled:

### 1. Netlify Identity

Enable via: **Netlify dashboard → Site → Integrations → Identity → Enable**.

Invite users via the Identity tab. Users receive an email, set a password, and can then log in at `/admin/`.

### 2. Git Gateway

Enable via: **Netlify dashboard → Site → Integrations → Identity → Git Gateway → Enable**.

Git Gateway proxies CMS write operations (file updates, media uploads) through Netlify as authenticated Git commits, using the repository credentials associated with the Netlify account.

### Login flow

1. User navigates to `https://feuerwehr-untereisesheim.de/admin/`
2. Netlify Identity widget loads (from `https://identity.netlify.com/v1/netlify-identity-widget.js`)
3. User authenticates with email + password
4. CMS loads with write access to the connected repository
5. On save, Decap commits the changed file directly to the `master` branch, triggering a new Netlify build

### `local_backend` flag

`src/admin/config.yml` contains `local_backend: true`. This enables the local proxy server (`npx decap-server`) for development without Netlify authentication. It has **no effect in production** — Netlify's build environment does not run a local proxy server, so the CMS falls back to `git-gateway` automatically.

---

## Branch and Build Behaviour

The CMS is configured to commit to `branch: master`. Netlify's continuous deployment watches `master` and triggers a new build on every push.

| Action | Result |
|---|---|
| Push to `master` | Netlify build triggered → new deploy |
| CMS save | Decap commits to `master` → Netlify build triggered |
| PR to `master` | Netlify deploy preview (if enabled in Netlify settings) |

---

## Environment Variables

No environment variables are currently required. If secrets are added in future (e.g. analytics keys, API tokens), set them via **Netlify dashboard → Site → Environment variables** and reference them in build scripts via `process.env.VAR_NAME`.

---

## Dependency Updates

All dependencies are pinned with `^` (semver minor/patch updates allowed). To update:

```bash
npm outdated          # show available updates
npm update            # update within semver ranges
npx npm-check-updates # show all updates including major
```

Key packages to watch:
- `@11ty/eleventy` — Eleventy v2 and v3 have breaking changes vs v1 (front matter, config API)
- `tailwindcss` — v4 introduced a completely new CSS-first config format, incompatible with `tailwind.config.js`
- `alpinejs` — v3 is stable; v4 changes are being tracked upstream
- `decap-cms` — actively maintained; check release notes before upgrading the CDN version in `src/admin/index.html`

---

## Rollback

Netlify keeps a full deploy history. To roll back to a previous deploy:

**Netlify dashboard → Site → Deploys → [select deploy] → Publish deploy**

This does not revert the Git repository — only the live site. To also revert the source, use `git revert` and push to `master`.
