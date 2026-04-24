# CLAUDE.md

This file provides guidance to Claude (Claude Code, Claude Desktop, Claude CoWork) when working with code in this repository.

## Repository purpose

This is the **canonical theme repo** for Catalyst Real Estate's property website portfolio. It is NOT a property site itself — it does not publish a website. It owns the shared design, workflows, and conventions that are propagated into per-property site repos (`lynn-park-website`, `hampden-website`, future).

## When to edit THIS repo vs. a property repo

**Edit this repo** when the change is design or tooling scoped (affects ALL property sites):
- Layout, CSS, or JS used across every property.
- Shared CI workflows (HTML validation, Pages deploy).
- `CLAUDE.md` conventions.
- The canonical site template (once extracted).

**Edit a property repo** (e.g., `lynn-park-website`, `hampden-website`) when the change is content scoped (affects ONE property):
- Copy, unit availability, floor plans, photos.
- Property-specific amenities or features.
- The property's `CNAME`.
- Anything the property's owner or leasing team asks to tweak for that one site.

If a requested change is ambiguous — for example, "update the fonts on Lynn's site" — ask whether it should apply to ALL properties (edit theme) or only Lynn (edit lynn-park-website directly).

## How changes propagate

1. Changes merged to `main` in this repo fire a `repository_dispatch` event to every property repo listed in `property-sites.json` via `.github/workflows/notify-property-sites.yml`.
2. Each property repo's `.github/workflows/sync-from-theme.yml` picks up the dispatch (or its daily cron) and opens a sync PR using `actions-template-sync`.
3. Each property repo's `validate.yml` runs HTMLHint on the sync PR; if validation passes, a human (or Claude, with human approval) reviews and merges.

**Never** force-merge sync PRs that fail validation. Validation exists precisely so a bad theme change cannot silently break a live property site.

## Local validation

Before committing changes that touch HTML:

```bash
npm install -g htmlhint
htmlhint "**/*.html"
```

## Conventions

- Vanilla HTML/CSS/JS — no frameworks, no build step (yet).
- Single-file `index.html` baseline for now (~1,640 lines in the Lynn Park original). Partial extraction is planned but not done; tracked in [lynn-park-website#1](https://github.com/Catalyst-Real-Estate/lynn-park-website/issues/1).
- `.nojekyll` is always present so Pages serves HTML as-is.
- CSS variables live at `:root` — primary colors, typography, spacing. Override these per-property sparingly; prefer theme-level edits so all properties stay visually consistent.

## Deployment

This repo does NOT deploy a website. Property repos deploy automatically via GitHub Pages on push to `main`. The `pages.yml` workflow file lives here so template consumers inherit it at creation time, but it is guarded (`if: github.repository != 'Catalyst-Real-Estate/property-site-theme'`) to skip execution on this theme repo.
