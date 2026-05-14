## CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Hugo static site for kurtfehlhauer.com using the **PaperMod** theme. Content is the product; there is no application code beyond Hugo templates and one custom shortcode.

## Commands

- `hugo server -D` — local dev server including draft posts (`draft: true` in frontmatter)
- `hugo --minify` — production build into `public/` (matches the CI build)
- `hugo new posts/<slug>.md` — scaffold a new post from `archetypes/default.md` (defaults to `draft: true`)
- `git submodule update --init --recursive` — required after a fresh clone

Hugo extended **v0.161.1** is pinned in CI — match it locally if image processing behavior matters.

## Deployment

`.github/workflows/gh-pages.yml` builds on every push to `main` and publishes `public/` to the `gh-pages` branch with CNAME `kurtfehlhauer.com`. There is no staging environment — merging to `main` is the deploy.

## Architecture notes worth knowing before editing

- **`baseurl` in `config.yaml` is intentionally `localhost`** for local development. The author flips this to the production URL manually before pushing a deploy. The CI workflow does not override it — assume any commit on `main` that hasn't had `baseurl` flipped will produce a broken production deploy until corrected.
- **PaperMod is a git submodule** at `themes/PaperMod`, pinned to a specific commit on `master` (not a tag — the most recent tag, `v8.0`, is incompatible with Hugo ≥0.158 because it predates the `_partials/` layout migration). To update the theme: `git -C themes/PaperMod fetch && git -C themes/PaperMod checkout <commit-or-newer-tag>`, build locally, then `git add themes/PaperMod` from the repo root.
- **Theme overrides go in top-level `layouts/`**, not inside `themes/PaperMod/`. The submodule should stay clean.
- **`layouts/shortcodes/image-gallery.html`** is the one piece of custom Hugo code. It pulls `Page.Resources` of type image, generates a 300x300 thumbnail and a 1300px-wide full version, and wires up the local lightbox (`static/js/lightbox.js`, `static/css/lightbox.css`). The photography page (`content/photography/index.md`) is a page bundle — drop images into `content/photography/img/` and they appear in the gallery automatically.
- **Profile mode** is on (`params.profileMode.enabled: true`), so the homepage renders the profile card, not a post list. Changing the landing experience means toggling that, not editing a homepage template.

## Known deprecation warnings (non-fatal)

The build emits warnings about `languageCode` (config key) and `.Language.LanguageDirection` / `.Language.LanguageCode` (template calls inside PaperMod). All three are Hugo 0.158 deprecations that won't be removed until a future release. The PaperMod ones go away when the theme is updated past the relevant PR; the config one needs `languageCode: en-us` → `locale: en-us` in `config.yaml` when ready.
