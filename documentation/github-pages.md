# GitHub Pages Deployment

## Overview

The site is deployed to GitHub Pages via a GitHub Actions workflow that builds the static site on every push to `main` and publishes the output directory.

**Workflow file:** `.github/workflows/deploy.yml`

---

## One-Time Setup (GitHub UI)

Before the workflow can deploy, GitHub Pages must be configured to use GitHub Actions as the deployment source. This is a one-time step per repository.

1. Go to the repository on GitHub
2. Navigate to **Settings → Pages**
3. Under **Build and deployment → Source**, select **GitHub Actions**
4. Save

The next push to `main` (or a manual trigger) will deploy the site.

---

## Workflow Breakdown

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

- Triggers automatically on every push to `main`
- `workflow_dispatch` allows manual runs from the GitHub Actions UI without a code push

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

Minimal required permissions. `pages: write` allows the workflow to publish to GitHub Pages. `id-token: write` is required by the OIDC-based `actions/deploy-pages` action.

```yaml
concurrency:
  group: "pages"
  cancel-in-progress: false
```

Prevents two deploys from running simultaneously. `cancel-in-progress: false` means a queued deploy waits for the current one to finish rather than being cancelled — important for a production site.

### Jobs

**`build` job:**

| Step | Action | Notes |
|---|---|---|
| Checkout | `actions/checkout@v4` | Full repo checkout |
| Setup Node.js | `actions/setup-node@v4` | Node 20 LTS, npm cache enabled |
| Install | `npm ci` | Reproducible install from `package-lock.json` |
| Build | `npm run build` | Runs Eleventy then Tailwind CLI (sequential) |
| Upload artifact | `actions/upload-pages-artifact@v3` | Packages `_site/` as a Pages deployment artifact |

**`deploy` job:**

Runs after `build` (`needs: build`). Calls `actions/deploy-pages@v4` which takes the uploaded artifact and publishes it. The `environment` block sets the deployment URL in the GitHub UI.

---

## Build Command Reference

The production build command (`npm run build`) runs two steps sequentially:

```bash
cross-env NODE_ENV=production eleventy
cross-env NODE_ENV=production tailwindcss -i src/static/css/tailwind.css -o _site/static/css/style.css
```

**Order matters:** Tailwind must run after Eleventy because Tailwind scans `src/**/*.html` for class names. Since the content path is `./src/**/*.html` (source files, not `_site/`), the ordering doesn't actually affect CSS generation — but Eleventy must run first so `_site/` exists for the artifact upload.

---

## Custom Domain

To serve the site under a custom domain (e.g. `feuerwehr-untereisesheim.de`):

### 1. Add a CNAME file

Create `src/static/CNAME` containing only the domain:

```
feuerwehr-untereisesheim.de
```

Then add a passthrough copy in `.eleventy.js` so Eleventy copies it to the build output root:

```js
eleventyConfig.addPassthroughCopy("./src/static/CNAME");
```

GitHub Pages reads the `CNAME` file from the root of the deployed site to configure the custom domain.

### 2. Configure DNS

Add a `CNAME` DNS record pointing to `<username>.github.io` at your DNS provider. For apex domains (no `www`), use four `A` records pointing to GitHub's IPs instead:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

### 3. Enable HTTPS

After DNS propagates (up to 24h), go to **Settings → Pages → Custom domain** and tick **Enforce HTTPS**. GitHub provisions a Let's Encrypt certificate automatically.

---

## Default GitHub Pages URL

Without a custom domain, the site is served at:

```
https://<username>.github.io/<repository-name>/
```

**Path prefix issue:** All internal asset paths in the built HTML are absolute (e.g. `/static/css/style.css`). On a subdirectory URL like `/feuerwehr-untereisesheim/`, these paths resolve to `https://<username>.github.io/static/css/style.css` — which does not exist — and the site will be unstyled.

**Fix (implemented):** Eleventy's `pathPrefix` is set conditionally in `.eleventy.js` based on the `GITHUB_ACTIONS` environment variable, which GitHub automatically sets to `"true"` in every workflow run:

```js
// .eleventy.js
return {
  dir: { input: "src" },
  pathPrefix: process.env.GITHUB_ACTIONS === "true" ? "/feuerwehr-untereisesheim/" : "/",
};
```

All internal links and asset paths in templates use Eleventy's `| url` filter (e.g. `{{ '/static/css/style.css' | url }}`), which prepends the `pathPrefix` automatically. Local development is unaffected — `pathPrefix` stays `/` and paths resolve as normal.

**When switching to a custom domain:** Remove the `pathPrefix` conditional and set it to `"/"` permanently, or remove the line entirely (defaults to `"/"`). Then remove the `CNAME` file if you add one (see above) — the path prefix is only needed for the subdirectory URL.

---

## CMS on GitHub Pages

The Decap CMS admin panel (`/admin/`) will load on GitHub Pages but **cannot authenticate** because the `git-gateway` backend is a Netlify-specific service.

To enable CMS editing on GitHub Pages, switch the backend in `src/admin/config.yml`:

```yaml
backend:
  name: github
  repo: <owner>/<repository>
  branch: main
```

The `github` backend authenticates directly via GitHub OAuth. This requires an OAuth application:

1. Create a GitHub OAuth App at **GitHub → Settings → Developer settings → OAuth Apps**
   - Homepage URL: `https://feuerwehr-untereisesheim.de`
   - Authorization callback URL: `https://api.netlify.com/auth/done` (Netlify still acts as the OAuth callback proxy even without hosting)
2. Store the Client ID and Secret in Netlify (free tier, no hosting required) and link to the repository
3. Update `src/admin/config.yml` with `base_url: https://api.netlify.com`

Alternatively, a self-hosted OAuth proxy such as [decap-cms-github-oauth-provider](https://github.com/vencax/netlify-cms-github-oauth-provider) can be deployed to a free service like Fly.io or Railway.

---

## Deployment Status

After setup, each push to `main` creates a new entry under **GitHub → Actions**. A green checkmark indicates a successful deploy. The deployed URL is shown in the `deploy` job output and in **Settings → Pages**.

To manually trigger a deploy without a code change:

1. Go to **GitHub → Actions → Deploy to GitHub Pages**
2. Click **Run workflow → Run workflow**
