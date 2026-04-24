# property-site-theme

Canonical theme repo for Catalyst Real Estate's property website portfolio.

## What this is

Shared source of truth for layout, styling, scripts, CI workflows, and editing conventions across every property website in the `Catalyst-Real-Estate` org. Each property (lynn-park, hampden, future) lives in its own repo and publishes to its own GitHub Pages site with its own custom domain, but inherits common design and tooling from this repo.

## Architecture

```
Catalyst-Real-Estate/
├── property-site-theme/      ← THIS REPO — shared layout, CSS, JS, workflows, CLAUDE.md
├── lynn-park-website/        ← property-specific content + CNAME → lynn-park.com
├── hampden-website/          ← property-specific content + CNAME → hampden.<domain>
└── <future>-website/         ← etc.
```

### Two propagation paths

**1. Creating a new property repo** — "Use this template" on this repo copies everything here into the new repo as a starting point. That handles the initial set.

**2. Updating an existing property repo when the theme changes** — template copies do NOT stay in sync automatically. For ongoing propagation, each property repo has a `.github/workflows/sync-from-theme.yml` workflow (inherited from this template) that runs on a daily cron AND whenever this theme repo fires a `repository_dispatch` event. It uses [`AndreasAugustin/actions-template-sync`](https://github.com/marketplace/actions/actions-template-sync) to open a PR with the theme's latest changes. The child repo's own `validate.yml` gates the merge.

This repo's `.github/workflows/notify-property-sites.yml` runs on push to `main` and fires the `repository_dispatch` to each registered child repo. Children are listed in `property-sites.json` at the root of this repo — edit that file when adding or removing a property.

### Authentication for cross-repo sync

The notify workflow needs a token that can fire `repository_dispatch` on each child repo. The sync workflow on each child needs read access to this theme repo. Both are handled by a single fine-grained PAT (or a GitHub App installation token) stored as `THEME_SYNC_TOKEN` in org or repo secrets.

Required scopes:
- Read access to `Catalyst-Real-Estate/property-site-theme` (for the sync step on children).
- `repository_dispatch` write on each property repo (for the notify step here).
- `contents: write` and `pull-requests: write` on each property repo (so the sync can push a branch and open a PR).

## What lives here vs. in each property repo

**Here (shared, synced):**
- `.github/workflows/validate.yml` — HTML validation (HTMLHint).
- `.github/workflows/pages.yml` — GitHub Pages deploy (runs only in child repos; guarded so it does not run here).
- `.github/workflows/sync-from-theme.yml` — theme-to-child sync (runs only in child repos; guarded so it does not run here).
- `.github/workflows/notify-property-sites.yml` — dispatches to children (runs only in this repo).
- `.gitignore`, `.nojekyll`, `.templatesyncignore`.
- `CLAUDE.md` — canonical editing guidance.
- `property-sites.json` — registry of property repos to notify.

**Per-property (not synced, ignored by `.templatesyncignore`):**
- `index.html` — property-specific content, copy, amenities, floor plans.
- `images/` — property photos, floor plans, site plans.
- `CNAME` — the property's custom domain.
- The property repo's own `README.md`.

### On partial extraction

The current lynn-park `index.html` inlines ALL CSS and JavaScript in a single ~1,640-line file. There is no client-side include mechanism on a plain GitHub Pages site (`.nojekyll` is set, no SSG), so true HTML partials require either a build step or a switch to an SSG (Eleventy/Astro/Hugo). That decision is deferred; see [lynn-park-website#1](https://github.com/Catalyst-Real-Estate/lynn-park-website/issues/1).

In the meantime, shared design lives in this repo as whole files that get synced as-is (workflows, conventions). Per-property `index.html` stays out of the sync via `.templatesyncignore`, so design changes intentionally do NOT auto-propagate into the property's `index.html` — a human or Claude edits each property's `index.html` manually when a structural HTML change is needed. Revisit this when we cross ~5–10 properties or adopt an SSG.

## Setting up a new property site

1. Click "Use this template" on this repo → create `<property>-website` under `Catalyst-Real-Estate`.
2. Set the new repo's `CNAME` to the property's custom domain.
3. Add the property's assets: `index.html`, `images/`.
4. Add the new repo name to `property-sites.json` in this theme repo (so it receives sync dispatches on future theme updates).
5. Enable GitHub Pages on the new repo (Settings → Pages → Source: GitHub Actions).
6. Make sure `THEME_SYNC_TOKEN` is available to the new repo (via org-level secret or manual copy).
7. Point DNS at GitHub Pages.

## See also

- Tracking issue for the architecture rollout: [Catalyst-Real-Estate/lynn-park-website#1](https://github.com/Catalyst-Real-Estate/lynn-park-website/issues/1).
- Recommendation source: Anthropic Console session `sesn_011CaMxnT8dFusWCQsexqp3P`.
