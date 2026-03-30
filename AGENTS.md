# Repository Guidelines

## Project Structure & Module Organization
This repository is a Jekyll blog powered by the `jekyll-theme-chirpy` gem. Core site settings live in `_config.yml`. Long-form content belongs in [`_posts/`](/home/yigit/Projects/ertodebana.github.io/_posts) using dated filenames such as `2026-03-29-topic-name.md`. Static top-level pages live in [`_tabs/`](/home/yigit/Projects/ertodebana.github.io/_tabs), while entry points like [`index.html`](/home/yigit/Projects/ertodebana.github.io/index.html) and [`404.html`](/home/yigit/Projects/ertodebana.github.io/404.html) stay at the repo root. GitHub Pages deployment is defined in [`.github/workflows/pages-deploy.yml`](/home/yigit/Projects/ertodebana.github.io/.github/workflows/pages-deploy.yml).

## Build, Test, and Development Commands
Install dependencies with `bundle install`.
Run the site locally with `bundle exec jekyll serve` and preview at `http://127.0.0.1:4000`.
Create a production build with `JEKYLL_ENV=production bundle exec jekyll build -d _site`.
Validate generated pages with `bundle exec htmlproofer _site --disable-external`.
Use the build step before opening a PR when navigation, front matter, or permalinks change.

## Coding Style & Naming Conventions
Write Markdown with concise headings and fenced code blocks where useful. Posts should start with YAML front matter including `title`, `date`, `categories`, `tags`, and `description`. Follow the existing naming pattern: lowercase, hyphen-separated slugs, date prefix, and ASCII-only filenames. Match the repository’s current content style: Turkish copy in body text, simple configuration in YAML, and two-space indentation in `_config.yml`.

## Testing Guidelines
There is no separate unit test suite in this repository. The primary check is a clean Jekyll build plus `html-proofer` against `_site`. For content changes, verify that category pages, tag pages, code blocks, and internal links render correctly. When adding a new post, confirm the permalink resolves under `/posts/<slug>/`.

## Commit & Pull Request Guidelines
Recent history uses short Conventional Commit prefixes such as `feat:` and `fix:`. Keep commit subjects imperative and scoped to one change, for example `feat: add article on platform engineering`. Pull requests should include a concise summary, note any config or URL changes, and attach screenshots for visible layout changes. Link the related issue when one exists, and mention the local verification commands you ran.
