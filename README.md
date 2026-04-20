# Visuality Marketplace

An open [Claude Code](https://code.claude.com/) plugin marketplace by [Visuality](https://visuality.pl) — skills, agents, commands, hooks and MCP servers for Ruby, Rails and AI-assisted development.

## Install

```text
/plugin marketplace add visualitypl/visuality-marketplace
/plugin install <plugin-name>@visuality
```

Or for local development:

```text
/plugin marketplace add /path/to/visuality-marketplace
```

## Use outside Claude Code

Skills in this marketplace follow the [Agent Skills open standard](https://agentskills.io) — originally developed by Anthropic, now adopted by 30+ agent tools including OpenCode, Cursor, GitHub Copilot CLI, OpenAI Codex, Gemini CLI, JetBrains Junie, Goose, OpenHands and more.

The `/plugin install` flow is Claude Code-specific, but the `SKILL.md` files themselves are portable. To use a skill in another tool, clone the repo and copy the skill folder:

```bash
git clone https://github.com/visualitypl/visuality-marketplace /tmp/visuality-marketplace
cp -r /tmp/visuality-marketplace/plugins/ruby-llm-v1/skills/ruby-llm ~/.claude/skills/
```

Most tools read from `~/.claude/skills/` for backwards compatibility, or from the tool-neutral `~/.agents/skills/`. Verified install paths:

| Tool | Install path |
| :--- | :--- |
| [Claude Code](https://code.claude.com/docs/en/skills) | `/plugin install …@visuality` (recommended) or `~/.claude/skills/` |
| [OpenCode](https://opencode.ai/docs/skills/) | `~/.config/opencode/skills/` or `~/.claude/skills/` or `~/.agents/skills/` |
| [Cursor](https://cursor.com/docs/context/skills) | `~/.cursor/skills/` or `~/.agents/skills/` |
| [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills) | `~/.copilot/skills/` or `~/.claude/skills/` or `~/.agents/skills/` |

For other agents (Codex, Gemini CLI, Junie, Goose, Roo Code, Kiro, …) see the [full list at agentskills.io](https://agentskills.io) — each entry links to its own install docs. After copying the skill folder, most tools auto-discover on next session; some need an explicit reload (e.g. `/skills reload` in Copilot CLI, `reload-plugins` in Claude Code).

Note: version pinning (`ruby-llm-v1` vs future `ruby-llm-v2`) is handled by the marketplace metadata, which only Claude Code reads. In other tools you copy a skill manually — pick the right `plugins/<name>-vN/` folder for your library version.

## Plugins

| Plugin | For | Description |
| :--- | :--- | :--- |
| [`ruby-llm-v1`](plugins/ruby-llm-v1/) | RubyLLM 1.x | chat, tools, structured output, streaming, Rails integration, agents, embeddings, moderation, transcription. Verified line-by-line against upstream source. |

More coming. Want one published here? See [CONTRIBUTING.md](CONTRIBUTING.md).

## Versioning

We split skills per major version of the target library. A plugin named `<lib>-v1` covers `<lib>` 1.x; when 2.0 ships we add `<lib>-v2` and freeze v1 for legacy users. `/plugin update` never silently swaps semantics under your feet — pick the plugin matching your Gemfile.

## Why a marketplace

Claude Code and other agent tools are only as good as the context they're given. The [Agent Skills open standard](https://agentskills.io) lets us distill library docs, conventions and workflows into small, versioned, auto-invoked reference files — so the agent stops improvising on stale training data and starts writing code we'd actually ship.

Visuality has been building with Rails since 2008. This marketplace is where we publish the guardrails we use internally, in a form anyone can install with one command.

## License

MIT. See [LICENSE](LICENSE). Individual plugins may carry their own LICENSE files — check inside each `plugins/<name>/` folder.
