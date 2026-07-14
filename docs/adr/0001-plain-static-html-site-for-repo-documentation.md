# 0001. Plain static HTML/CSS site for repo documentation

Date: 2026-07-14

## Status

Accepted

## Context

kiro-starter's README documents its features (agents, hooks, MCP servers, prompts, LSP config) but only at a summary level. The tracked work item (Linear RYA-5, "Web app showing a Kiro Starter team") calls for a small site that documents the repo's features in more detail than the README currently does.

kiro-starter itself is a config/template repo with no application code, no build tooling, and no existing test/lint setup — it's `.kiro/` configuration, a README, and a license. Any documentation site added here should not introduce infrastructure that's disproportionate to that footprint, since the repo is meant to be copied from or pointed at directly, not maintained as a product.

There's no hosting requirement yet — the site only needs to be viewable (e.g. opened locally in a browser), though it should not preclude adding hosting (e.g. GitHub Pages) later.

## Decision

Build the documentation site as plain static HTML and CSS, with no build step, no framework, and no external dependencies, in a new top-level `site/` folder (kept separate from `docs/`, which holds plans and ADRs).

Structure: one HTML page per feature area (`index.html`, `hooks.html`, `mcp.html`, `lsp.html`, `prompts.html`, `agents.html`), sharing a single `styles.css`, with a manually duplicated nav header on each page rather than a templating layer. Content is written directly from the underlying `.kiro/` config and prompt files (not just reworded README text) so it stays accurate to the actual source of truth.

## Consequences

**Easier:**
- Zero setup — no dependencies to install, no build step to run or maintain, no toolchain to keep updated.
- Can be opened directly via `file://` or trivially hosted later (e.g. GitHub Pages) with no changes, since it's already static output.
- Matches the repo's existing footprint (a config template repo with no other tooling), so it doesn't introduce an outsized maintenance burden relative to the rest of the project.

**Harder:**
- No templating means shared markup (the nav header) is duplicated across every page by hand. Adding a page or changing the nav requires editing each file individually.
- No component/include mechanism, so content reuse between pages (if it grows) will require manual copy-paste discipline to stay consistent.
- No content pipeline (e.g. Markdown source generating HTML), so keeping the site in sync with `.kiro/` config changes is a manual process, not automated.

These tradeoffs are accepted because the site is small (six pages) and the priority stated was simplicity over scalability. If the docs grow significantly or the duplication becomes error-prone, revisit with a lightweight static site generator (e.g. Eleventy) — this ADR does not preclude that; it documents why the simpler option was chosen for the current scope.

## Alternatives Considered

- **A minimal static site generator (e.g. Eleventy/11ty, Astro):** Would remove markup duplication via templating/layouts and support Markdown content sources. Rejected for now because it introduces a build step and dependencies for a six-page site, which is more tooling than the current scope justifies. Revisit if the site grows.
- **A fuller docs framework (e.g. VitePress, Docusaurus):** Provides sidebar navigation, search, and theming out of the box. Rejected as disproportionate — these are built for larger, growing documentation sets, and this repo is a small personal starter kit, not a product with a documentation surface at that scale.
- **Deploying/hosting now (e.g. GitHub Pages):** Rejected for this iteration — no hosting requirement was given, and the static-HTML choice already keeps that option open without extra work later.
