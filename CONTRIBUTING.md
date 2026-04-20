# Contributing

New plugins and improvements welcome. This is an open marketplace — you don't need to work at Visuality.

Before contributing, read **[CLAUDE.md](CLAUDE.md)** for the quality standards we apply to every plugin: audit flow (claims verified against pinned library source), `description:` TRIGGER/SKIP format for reliable auto-invocation, naming conventions, release flow. Those rules are non-negotiable at review time; the rest of this file just covers the PR logistics.

## Propose a new plugin

1. Fork, then create `plugins/<your-plugin-name>-vN/` (kebab-case, `-vN` where N matches the target library's major version — see [CLAUDE.md § Naming](CLAUDE.md#naming--three-levels-three-rules)).
2. Add `plugins/<name>-vN/.claude-plugin/plugin.json` with `name`, `version`, `description`, `author`, `license`, at minimum.
3. Add your content:
   - **Skills** — `plugins/<name>-vN/skills/<skill-name>/SKILL.md` (see [Agent Skills standard](https://agentskills.io) and [CLAUDE.md § description:](CLAUDE.md#description--trigger--skip-format-not-marketing-prose))
   - **Commands** — `plugins/<name>-vN/commands/*.md`
   - **Agents** — `plugins/<name>-vN/agents/*.md`
   - **Hooks** — `plugins/<name>-vN/hooks/hooks.json`
   - **MCP servers** — `plugins/<name>-vN/mcp-config.json`
4. Add an entry for your plugin in `.claude-plugin/marketplace.json`.
5. Add a row for your plugin in the `README.md` table.
6. Test locally: `/plugin marketplace add /path/to/this/repo` then `/plugin install <name>-vN@visuality`.
7. Open a PR.

## Review policy

One maintainer review, merged on Visuality's cadence. We check:

- Plugin content works in Claude Code (loads without errors, invokes correctly).
- Documentation/skill content matches current library versions, not training-data guesses.
- License is permissive (MIT, Apache 2.0, BSD) and compatible with the marketplace MIT license.
- No secrets, no PII, no vendored binaries.

## Improvements to existing plugins

Same flow — open a PR against `plugins/<name>-vN/`. For skills, follow the audit procedure in [CLAUDE.md § Audit](CLAUDE.md#audit-every-factual-claim-must-be-verified-against-source) — clone the target library at its pinned version and verify every changed claim against source.

## Questions

Open an issue, or reach out: p.strzalkowski@visuality.pl.
