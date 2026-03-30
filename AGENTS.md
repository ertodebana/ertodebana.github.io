# Repository Guidelines

## Writing Voice
For any reader-facing copy, review [`souls.md`](/home/yigit/Projects/ertodebana.github.io/souls.md) before writing. It defines the expected tone, what to avoid, and how posts on this site should sound in both Turkish and English.

## Project Structure & Module Organization
This repository is a bilingual Jekyll blog powered by the `minimal-mistakes-jekyll` theme and `jekyll-polyglot`. Core site settings live in [`_config.yml`](/home/yigit/Projects/ertodebana.github.io/_config.yml). Long-form content belongs in [`_posts/`](/home/yigit/Projects/ertodebana.github.io/_posts) using dated filenames such as `2026-03-29-topic-name.md`; Turkish and English variants are usually separate files. Static pages live in [`_pages/`](/home/yigit/Projects/ertodebana.github.io/_pages), localized navigation/data lives under [`_data/tr/`](/home/yigit/Projects/ertodebana.github.io/_data/tr) and [`_data/en/`](/home/yigit/Projects/ertodebana.github.io/_data/en), and theme overrides live in [`_includes/`](/home/yigit/Projects/ertodebana.github.io/_includes). The Turkish landing page is still rooted at [`index.html`](/home/yigit/Projects/ertodebana.github.io/index.html), while the English landing page lives in [`_pages/home-en.md`](/home/yigit/Projects/ertodebana.github.io/_pages/home-en.md). GitHub Pages deployment is defined in [`.github/workflows/pages-deploy.yml`](/home/yigit/Projects/ertodebana.github.io/.github/workflows/pages-deploy.yml).

## Build, Test, and Development Commands
Install dependencies with `bundle install`.
Run the site locally with `bundle exec jekyll serve` and preview at `http://127.0.0.1:4000`.
Create a production build with `JEKYLL_ENV=production bundle exec jekyll build -d _site`.
Validate generated pages with `bundle exec htmlproofer _site --disable-external`.
Use the build step before opening a PR when navigation, front matter, or permalinks change.

## Coding Style & Naming Conventions
Write Markdown with concise headings and fenced code blocks where useful. Posts should start with YAML front matter including `title`, `date`, `lang`, `locale`, `page_id`, `permalink`, `categories`, `tags`, and `description`. Follow the existing naming pattern: lowercase, hyphen-separated slugs, date prefix, and ASCII-only filenames. Match the repository’s current style: practical Turkish/English copy, simple YAML, and two-space indentation in `_config.yml`.

## Testing Guidelines
There is no separate unit test suite in this repository. The primary check is a clean Jekyll build plus `html-proofer` against `_site`. For content changes, verify that localized pages, category pages, tag pages, code blocks, and internal links render correctly. When adding a new post, confirm the permalink resolves under `/posts/<slug>/` and that the matching TR/EN page structure stays consistent.

## Commit & Pull Request Guidelines
Recent history uses short Conventional Commit prefixes such as `feat:` and `fix:`. Keep commit subjects imperative and scoped to one change, for example `feat: add article on platform engineering`. Pull requests should include a concise summary, note any config or URL changes, and attach screenshots for visible layout changes. Link the related issue when one exists, and mention the local verification commands you ran.
