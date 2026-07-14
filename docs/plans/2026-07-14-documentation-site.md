# Plan: Documentation site for kiro-starter

**Ticket:** [RYA-5 — Web app showing a Kiro Starter team](https://linear.app/ryan-cormack/issue/RYA-5/web-app-showing-a-kiro-starter-team)

## Clarified scope (from user)

- A small **static site** documenting this repo's features in more detail than the README.
- Plain HTML/CSS, no build step, no framework/dependencies.
- No hosting/deploy setup for now — just a site that can be opened locally (and could later be pushed to GitHub Pages with zero changes, since it's static).
- Keep it simple.

## Approach

Add a new top-level `site/` folder (kept separate from `docs/`, which is reserved for plans/ADRs) containing:

- `index.html` — landing page: what kiro-starter is, and a summary/nav linking to each feature area.
- `hooks.html` — details on the two hooks (`npm-latest-version`, `conventional-commit-guard`): what they intercept, how they decide to block, fail-open behavior, config snippets.
- `mcp.html` — the Linear MCP server config and what it enables.
- `lsp.json` → `lsp.html` — the code intelligence/LSP setup, languages configured, how to enable via `/code init`.
- `prompts.html` — the `/commit` slash command and what it does step by step.
- `agents.html` — the three custom agents (`work-finder`, `adr-writer`, `gpt-review`): purpose, tools/trust boundaries, how to switch to them.
- `styles.css` — one shared stylesheet, minimal (system font stack, basic layout, code blocks), no external assets/CDNs.

Content will be written directly from the actual config/prompt files (already read: `.kiro/agents/*.json`, `.kiro/agents/*.md`, `.kiro/hooks/*.json`, `.kiro/prompts/commit.md`, `.kiro/settings/mcp.json`, `.kiro/settings/lsp.json`), not just reworded README text, so it stays accurate as the source of truth.

Nav will be a simple shared header repeated on each page (no templating engine, so this is manually duplicated markup — acceptable at this size; a future iteration could introduce a static site generator if the docs grow, but that's out of scope per "keep it simple").

## Files to add

```
site/
├── index.html
├── hooks.html
├── mcp.html
├── lsp.html
├── prompts.html
├── agents.html
└── styles.css
```

No existing files change, except the README gets a one-line pointer to the new site (e.g. under "Usage").

## Verification

- No build/test tooling exists in this repo (it's a config template repo, not an app) — confirmed via directory listing (only `.kiro/`, `README.md`, `LICENSE` at top level).
- Manual verification: open `site/index.html` directly in a browser (`file://` URL) and click through every nav link to confirm all pages render and internal links resolve.
- Visually check that code/config snippets are legible (monospace, no overflow clipping) at a normal window width.
