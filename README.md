# kiro-starter

A personal "getting started" repo for [Kiro CLI](https://kiro.dev) features — hooks, MCP servers, prompts/slash commands, steering, and other utilities I want available as a baseline in any new project. Copy pieces from here into other repos, or point Kiro at this repo directly to try things out.

## What's in here

```
.kiro/
├── agents/
│   ├── work-finder.json                 # custom agent config
│   ├── work-finder.md                   # its system prompt
│   ├── adr-writer.json                  # custom agent config
│   ├── adr-writer.md                    # its system prompt
│   ├── gpt-review.json                  # custom agent config
│   └── gpt-review.md                    # its system prompt
├── hooks/
│   ├── npm-latest-version.json          # PreToolUse hook config
│   ├── conventional-commit-guard.json   # PreToolUse hook config
│   └── scripts/
│       ├── npm-latest-version.sh
│       └── conventional-commit-guard.sh
├── prompts/
│   └── commit.md                        # /commit slash command
└── settings/
    ├── mcp.json                          # Linear MCP server
    └── lsp.json                          # code intelligence / LSP servers
```

### Hooks

Hooks run shell scripts at specific points in the agent's lifecycle (e.g. before a tool call) and can block the call by exiting non-zero. Both hooks below are `PreToolUse` hooks that fire on every `execute_bash`/shell call, cheaply bail out if the command doesn't match what they care about, and **fail open** (exit 0) on anything ambiguous — jq missing, unparseable input, etc. — so they can never wedge the agent.

- **`npm-latest-version`** — Intercepts `npm install`/`i`/`add` commands. If any package is unpinned (no exact version, or a floating tag/range), it looks up the latest published version with `npm view` and blocks the install, prompting the agent to re-run with exact version pins. Supports a `warn`-only mode via `NPM_LATEST_HOOK_MODE=warn`.

- **`conventional-commit-guard`** — Intercepts `git commit` commands that carry an inline `-m`/`--message`. Validates the message against [Conventional Commits](https://www.conventionalcommits.org/) format (`type(scope)?: description`, with allowed types `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`) and blocks the commit with guidance if it doesn't conform. Commits without an inline message (i.e. ones that open `$EDITOR`) pass through untouched, since there's nothing to inspect yet. Command parsing is quote-aware but expansion-free (tokenized via `xargs`, not `eval`), so it can't be tricked into executing anything embedded in a commit message.

See `.kiro/hooks/*.json` for the hook configs and `.kiro/hooks/scripts/*.sh` for the implementations.

### MCP servers

MCP (Model Context Protocol) servers add extra tools the agent can call. Configured in `.kiro/settings/mcp.json` (workspace-scoped).

- **Linear** — Connects to Linear's official remote MCP server (`https://mcp.linear.app/mcp`) for reading/writing issues, projects, and teams. Uses OAuth (handled natively by Kiro — no local proxy needed); on first use, Kiro will prompt you to authorize in a browser.

### Code intelligence / LSP

`.kiro/settings/lsp.json` configures optional language servers for enhanced code intelligence (find references, go to definition, rename, diagnostics, hover) on top of the built-in tree-sitter support. Pre-configured here: TypeScript/JavaScript (`typescript-language-server`), Python (`pyright`), Kotlin (`kotlin-language-server`), Swift (`sourcekit-lsp`), and Smithy (`smithy-language-server`). Each language server needs to be installed separately and available on `PATH`; run `/code init` to detect installed servers and start them (`/code status` / `/code logs` to check on them afterward).

### Prompts / slash commands

Local prompt files in `.kiro/prompts/*.md` become slash commands (filename minus `.md` = command name), invokable as `/<name>` or via `/prompts`.

- **`/commit`** — Reviews the working directory, refuses to commit directly on `main`/`master` (offers to branch instead), splits unrelated changes into separate logical commits, and writes a Conventional Commit message for each (validated automatically by the `conventional-commit-guard` hook above). Does not push.

## Usage

Drop this repo's `.kiro/` contents into another project (or symlink/copy individual pieces), or start `kiro-cli chat` from within this repo to try things out directly.

### Custom agents

- **`work-finder`** — Pulls the next ticket from the Linear backlog (via the Linear MCP server above), confirms with you before starting, verifies the workspace is the right repo for the ticket, asks clarifying questions, gathers codebase context and writes a plan to `./docs/plans/{date}-{name}.md` before implementing it on a branch named with the Linear ticket ID (e.g. `feat/ENG-123-...`). Commits reference the ticket ID (validated by the `conventional-commit-guard` hook), and the PR it opens with `gh pr create` references the ticket in its description. Never merges — stops and hands back the PR URL. Read-only tools (`read`, `grep`, `glob`, `code`, `todo_list`, Linear) are trusted by default; `write` and `shell` are scoped via `toolsSettings` to this workspace and to specific safe git/gh commands, but still prompt for approval since those are the highest-impact actions in the workflow.

  Switch to it with `/agent work-finder`.

- **`adr-writer`** — Writes Architecture Decision Records to `./docs/adr/NNNN-title-slug.md` (auto-numbered, lightweight ADR template: Status/Context/Decision/Consequences/Alternatives). Runs on `claude-sonnet-5`. Read-only tools (`read`, `grep`, `glob`, `code`) are trusted for researching context; `write` is auto-trusted only for `./docs/adr/*.md` — writes anywhere else still prompt for approval, keeping the agent scoped to its one job.

  Switch to it with `/agent adr-writer`.

- **`gpt-review`** — A code review agent running on `gpt-5.6-sol`, useful as a second opinion from a different model family. Fully read-only (`read`, `grep`, `glob`, `code`, all trusted) — it reviews and reports, it doesn't edit anything. Reviews diffs, files, or whatever you point it at, and is instructed to disagree with prior conclusions when warranted rather than rubber-stamp them.

  Switch to it with `/agent gpt-review`.
