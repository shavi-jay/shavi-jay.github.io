# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project Context

Personal academic website for **Shavindra Jayasekera**, built with the **al-folio** Jekyll theme. Deployed to `https://shavi-jay.github.io` via GitHub Pages (gh-pages branch).

## Architecture

### Content Flow

`_config.yml` is the central hub — it controls site URL/baseurl, feature flags (`enable_*` booleans), plugin loading, Jekyll-Scholar bibliography settings, and third-party library versions with SRI hashes.

The homepage is `_pages/about.md` (layout: `about`, permalink: `/`). It pulls in announcements from `_news/`, selected papers from `_bibliography/papers.bib`, and latest blog posts from `_posts/`.

### Key Data Relationships

- **Publications**: `_bibliography/papers.bib` (BibTeX) → rendered by `jekyll-scholar` via `_layouts/bib.liquid`. Custom BibTeX keywords (`pdf`, `code`, `preview`, `selected`, `arxiv`, etc.) control UI elements. Scholar config under `scholar:` in `_config.yml` sets author highlighting and citation style.
- **CV**: `_data/cv.yml` + `assets/json/resume.json` (loaded via `jekyll-get-json`) → rendered by `_layouts/cv.liquid`.
- **Social links**: `_data/socials.yml` → consumed by header/footer includes.
- **Collections**: `_projects/`, `_news/`, `_teachings/`, `_books/` are Jekyll collections configured in `_config.yml` under `collections:`.

### Rendering Pipeline

- **Layouts** (`_layouts/`): `default.liquid` → wraps `page.liquid`, `post.liquid`, `about.liquid`, `cv.liquid`, `bib.liquid`, etc.
- **Includes** (`_includes/`): Reusable partials — `header.liquid`, `footer.liquid`, `scripts.liquid`, `head.liquid`, pagination, metadata.
- **Styles** (`_sass/`): SCSS partials — `_variables.scss` for theming, `_themes.scss` for light/dark mode toggle, component-specific files.
- **Plugins** (`_plugins/`): Custom Ruby plugins for citation fetching (Google Scholar, InspireHEP), BibTeX keyword filtering, external posts.
- **Image processing**: `jekyll-imagemagick` generates responsive WebP variants from `assets/img/` at widths 480/800/1400px. Requires ImageMagick on PATH.

### Docker Setup

`docker-compose.yml` uses prebuilt image `amirpourmand/al-folio:v0.16.3`. The entry point (`bin/entry_point.sh`) runs Jekyll with live reload and auto-restarts when `_config.yml` changes via `inotifywait`. Volume-mounts the repo at `/srv/jekyll`. Ports: 8080 (site), 35729 (live reload).

### CI/CD Workflows (`.github/workflows/`)

- **`deploy.yml`** — Jekyll build with `JEKYLL_ENV=production`, PurgeCSS, deploy to gh-pages branch
- **`prettier.yml`** — Formatting check (blocks PRs on failure)
- **`render-cv.yml`** — CV rendering from RenderCV format
- **`update-citations.yml`** — Auto-updates citation counts

### Plugin Gotchas

- **classifier-reborn**: Empty blog posts or posts with only stop words cause "Zero vectors cannot be normalized" errors.
- **jekyll-terser**: Installed from a custom Git fork (`RobertoJBeltran/jekyll-terser`), not RubyGems.
- **jekyll-minifier**: `compress_javascript` is disabled in `_config.yml` because terser handles JS separately.
