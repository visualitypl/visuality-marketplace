# Contributing

New plugins and improvements welcome. This is an open marketplace — you don't need to work at Visuality.

## Propose a new plugin

1. Fork, then create `plugins/<your-plugin-name>/` (kebab-case, no spaces).
2. Add `plugins/<name>/.claude-plugin/plugin.json` with `name`, `version`, `description`, `author`, `license`, at minimum.
3. Add your content:
   - **Skills** — `plugins/<name>/skills/<skill-name>/SKILL.md` (see [Agent Skills standard](https://agentskills.io))
   - **Commands** — `plugins/<name>/commands/*.md`
   - **Agents** — `plugins/<name>/agents/*.md`
   - **Hooks** — `plugins/<name>/hooks/hooks.json`
   - **MCP servers** — `plugins/<name>/mcp-config.json`
4. Add an entry for your plugin in `.claude-plugin/marketplace.json`.
5. Add a row for your plugin in the `README.md` table.
6. Test locally: `/plugin marketplace add /path/to/this/repo` then `/plugin install <name>@visuality`.
7. Open a PR.

## Review policy

One maintainer review, merged on Visuality's cadence. We check:

- Plugin content works in Claude Code (loads without errors, invokes correctly).
- Documentation/skill content matches current library versions, not training-data guesses.
- License is permissive (MIT, Apache 2.0, BSD) and compatible with the marketplace MIT license.
- No secrets, no PII, no vendored binaries.

## Improvements to existing plugins

Same flow — open a PR against `plugins/<name>/`. For skills, pin any factual claims to a specific library version and verify against source before merging. See how the `ruby-llm` plugin notes its audit history in [its SKILL.md](plugins/ruby-llm/skills/ruby-llm/SKILL.md).

## Questions

Open an issue, or reach out: p.strzalkowski@visuality.pl.
