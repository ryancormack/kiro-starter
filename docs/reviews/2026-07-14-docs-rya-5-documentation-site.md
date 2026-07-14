# Review: docs/rya-5-documentation-site

**Date:** 2026-07-14
**Plan:** `docs/plans/2026-07-14-documentation-site.md`
**ADR:** `docs/adr/0001-plain-static-html-site-for-repo-documentation.md`

## Summary

The static site follows the approved plain HTML/CSS architecture and mostly delivers the planned feature pages, but the branch should not merge as-is because its own documentation gives materially incorrect information about the `gpt-review` agent. The branch also includes an unrelated expansion of that agent's permissions and responsibilities that is outside RYA-5's plan.

## Against the plan

The implementation provides all seven planned files under `site/`: six HTML pages plus one shared stylesheet. It also adds the planned README link, uses no framework or external assets, and keeps the site separate from `docs/`.

There is scope creep: the plan says, at `docs/plans/2026-07-14-documentation-site.md:41`, that no existing files should change except the README link, but commit `10fc584` also redesigns `.kiro/agents/gpt-review.json` and `.kiro/agents/gpt-review.md`. That redesign is not required to deliver RYA-5 and should be reviewed and shipped separately.

No automated test/build tooling exists for the site. Per the review agent's restrictions, I did not run builds or browser tests; the planned manual browser verification therefore remains the relevant acceptance check.

## Against the ADR

The site complies with ADR 0001: it is plain static HTML/CSS under `site/`, has one page per documented feature area, shares `styles.css`, duplicates the navigation deliberately, and introduces no dependencies or hosting setup.

The extra `gpt-review` agent redesign is not covered by ADR 0001. It does not violate the static-site architecture directly, but it changes the source material while the site continues to describe the old behavior, undermining the ADR's requirement that content remain accurate to the underlying `.kiro/` configuration (`docs/adr/0001-plain-static-html-site-for-repo-documentation.md:21`).

## Findings

### Blocking

1. **The branch documents `gpt-review` as fully read-only even though the same branch grants it write and shell tools.** `site/agents.html:99-102` says the agent is “Fully read-only” and “doesn't edit anything,” and `README.md:71` makes the same claim. In contrast, `.kiro/agents/gpt-review.json:8-9` adds `write` and `shell`, `.kiro/agents/gpt-review.json:22-24` scopes writes to `docs/reviews`, and `.kiro/agents/gpt-review.md:32,75` requires writing a review file and describes read-only Git shell access. This is a direct factual contradiction in a documentation-focused change and misrepresents the agent's trust boundary. Update both the site and README to describe the new scoped write/read-only-shell behavior, or remove the unrelated agent redesign from this branch.

### Non-blocking

1. **Split the `gpt-review` redesign into its own branch/PR.** Commit `10fc584` changes agent behavior and broadens its tool permissions, while RYA-5 and the plan cover only the documentation site. Keeping that change separate would preserve a reviewable ticket scope and allow the permission changes to receive focused security review.

2. **Label the LSP configuration block as illustrative rather than valid JSON, or remove the comment.** `site/lsp.html:79-98` presents a “Config excerpt” containing `// ...kotlin, swift, smithy`, which is not valid JSON. It is understandable as an abbreviation, but readers may copy it expecting valid configuration.
