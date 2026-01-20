# Repository Guidelines

## Project Structure & Module Organization
- `docs/`: primary documentation sources in MDX (mostly standard Markdown with a few MDX components).
- `static/img/assets/`: image assets referenced by docs; use lowercase filenames with no spaces.
- `src/`: custom React components and styling overrides (notably `src/css/custom.css`).
- `sidebars.ts`: main navigation structure.
- `docusaurus.config.ts`: site configuration (theme, plugins, metadata).
- `build/`: generated static output (do not edit by hand).

## Build, Test, and Development Commands
- `npm install`: install dependencies (Node >= 18).
- `npm run start`: start the local Docusaurus dev server.
- `npm run build`: build the static site into `build/`.
- `npm run serve`: serve the built site locally for verification.
- `npm run typecheck`: run TypeScript checks for config and custom code.
- `npm run clear`: clear Docusaurus cache if the build acts stale.

## Coding Style & Naming Conventions
- MDX: keep Markdown clean and readable; prefer standard Markdown unless MDX components are needed.
- Images: only lowercase filenames, no spaces (e.g. `static/img/assets/vector-search.png`).
- Config/TS: follow existing formatting; no repo-wide formatter is configured.

## Testing Guidelines
- No automated test suite is defined in this repo.
- Use `npm run typecheck` to validate TS config changes.
- For doc changes, verify locally with `npm run start` or `npm run serve`.

## Commit & Pull Request Guidelines
- Commit messages in recent history are short, sentence-case imperatives (e.g. "Fix a couple of typos", "Update to ObjectBox 5.0").
- PRs should include: a clear summary, relevant links (issues or source material), and screenshots for visual/UI changes.
- Note any content-impacting changes (navigation, config, or assets) in the PR description.

## How-To: Update ObjectBox Version
- First update the changelog in `docs/README.mdx` with the new version and date (manual step done by a human).
- Identify the previous version, then search for that exact version string across the repo to find all references that may need a bump.
- After the changelog entry is in place, update version references across the docs.
- Default to updating runtime library references only; the generator has separate versioning and updates less frequently.
- Only update generator references if the changelog explicitly calls out a generator version bump.
- Runtime references live in `docs/installation.mdx` (`GIT_TAG vX.Y.Z` and download folder examples).
- Generator references live in `docs/installation.mdx`, `docs/getting-started.mdx`, and `docs/generator.mdx` (`find_package(ObjectBoxGenerator X.Y.Z REQUIRED)` and version string examples).
- Keep changes in sync so the docs and examples reference the correct release(s).
- If the search reveals new version locations, update this list in `AGENTS.md` so future bumps include them.

## SEO Schema Maintenance
- MDX files in `docs/` include a `<TechnicalArticleSchema>` component for SEO structured data.
- When making meaningful content changes to an MDX file, update the `dateModified` attribute to the current date (format: `YYYY-MM-DD`).
- Keep `datePublished` unchanged (it reflects the original publication date).

## Notes for Contributors
- Main content lives in `docs/`; keep navigation in sync via `sidebars.ts`.
- When adding new pages, ensure links are valid; `onBrokenLinks` is set to `throw`.
