# CLAUDE.md

Agent-facing guide for working in this marketplace repo.

Claude Code reads this file automatically when working here. It contains actionable rules for writing and maintaining plugins — audit procedures, description style, naming, release flow. End-user install guides and positioning live in [README.md](README.md); PR process in [CONTRIBUTING.md](CONTRIBUTING.md).

## Audit: every factual claim must be verified against source

Every SKILL.md in this marketplace is a claim about an external library's API. Training data is stale. Rule: pin the target library to a specific version, clone the source, verify every claim line-by-line.

1. Clone the target library at a published tag: `git clone https://github.com/<owner>/<lib> --depth=1 --branch <tag>`
2. Record the version in SKILL.md frontmatter (`<lib>_version: "x.y.z"`).
3. Walk through every reference file with the source open. Expect **3–4 full passes** before it's clean — the first pass catches ~60% of issues, later passes find return-type mismatches, undocumented helpers, delegated-vs-not methods, string-vs-symbol mismatches.
4. Never reference an API from memory. If a method exists, you have a source citation for it. If not, it doesn't belong in the skill.

`ruby-llm-v1` went through four audit passes. Every pass caught real bugs (e.g. `RubyLLM.paint` doesn't do image editing, `Message#thinking` returns an object not a string, `Agent` does not delegate `with_instructions`). Budget for the same effort on new skills.

## `description:` — TRIGGER / SKIP format, not marketing prose

Marketing-prose descriptions do not reliably fire Claude Code's auto-invocation. Use explicit TRIGGER and SKIP rules with concrete signals.

Bad (does not auto-invoke):

```yaml
description: Comprehensive guide to RubyLLM for Ruby and Rails developers
```

Good (auto-invokes reliably):

```yaml
description: "RubyLLM gem reference. TRIGGER when ANY of: Gemfile has `gem 'ruby_llm'`; file has `require 'ruby_llm'`; references `RubyLLM::` or `acts_as_chat`; user asks to add LLM/AI to a Ruby app; user mentions ruby_llm/rubyllm/RubyLLM. SKIP when: user explicitly uses `anthropic` / `ruby-openai` gem directly; non-Ruby code; generic Rails work unrelated to AI."
```

Signals to list: Gemfile lines, require statements, class/module names, framework-specific generators, explicit user phrasings. Include both halves — SKIP prevents false positives in adjacent domains.

## Naming — three levels, three rules

| Level | Format | Example | Rationale |
| :--- | :--- | :--- | :--- |
| Plugin (`name` in `plugin.json` and `marketplace.json`; folder under `plugins/`) | kebab-case with `-vN` major-version suffix | `ruby-llm-v1`, `solid-queue-v2` | Claude Code requires kebab-case for `name`. The `-vN` suffix enables per-major splits without a `/plugin update` silently swapping semantics for users on older library versions. |
| Skill (`name` in `SKILL.md`; folder under `skills/`) | kebab-case, no `-vN` | `ruby-llm`, `solid-queue` | The skill name becomes the slash-command. The `-vN` is a distribution concern; the skill documents its target version via `<lib>_version:` frontmatter. |

Mapping: plugin `ruby-llm-v1` wraps skill `ruby-llm` covering library 1.x; a future `ruby-llm-v2` wraps a separate copy of `ruby-llm` covering 2.x. Users invoke `/ruby-llm` regardless.

## Release flow

Per-plugin version changes, driven by the target library:

| Change in target library | Skill action | Plugin version |
| :--- | :--- | :--- |
| Patch (`1.14.1` → `1.14.2`) | Quick re-audit of changed source | Bump patch |
| Minor (`1.14` → `1.15`) | Full audit of new APIs, expand `references/` | Bump minor |
| Major (`1.x` → `2.0`) | Fork plugin folder to `<name>-v2`, freeze v1 (security fixes only) | New plugin at `1.0.0`; v1 frozen |

The **marketplace version** is a rollup — it bumps to the highest magnitude of any plugin change in that release:

| Change inside a plugin | Marketplace bump |
| :--- | :--- |
| Any plugin patch release | patch |
| Any plugin minor release | minor |
| Any plugin major release | major |
| Adding a new plugin | minor |
| Removing a plugin, or a structural change that breaks `/plugin marketplace add` | major |

Tag releases in git as `v<marketplace-version>` (e.g. `v1.1.0`). Consumers pin the marketplace tag, which pins every plugin to the exact set shipped at that tag. The per-plugin and target-library versions covered by each tag are documented in `CHANGELOG.md` at the repo root, one section per marketplace release.

When bumping, update three places in one commit: the changed plugin's `plugin.json` `version`, the `CHANGELOG.md` entry describing the change, and (if the marketplace magnitude moved) the marketplace release header in `CHANGELOG.md`. Only after that is a new git tag created.

## Plugin structure (required)

```
plugins/<name>-vN/
├── .claude-plugin/
│   └── plugin.json          # name, version, description, author, license
├── LICENSE                  # if content license differs from marketplace-level MIT
└── skills/<skill-name>/
    ├── SKILL.md             # frontmatter: name, description, <lib>_version
    └── references/*.md      # on-demand topic files linked from SKILL.md
```

`plugin.json` `name` must match the folder name (`<name>-vN`). `SKILL.md` `name` is separate (no `-vN`).

## Test locally before pushing

After any edit to `SKILL.md`, `references/`, or `plugin.json`:

1. First time in a checkout: `/plugin marketplace add /absolute/path/to/this/repo`
2. After edits: `/reload-plugins` in an active Claude Code session
3. Prompt with a relevant trigger (e.g. "how do I use X in Ruby?") and verify the skill auto-invokes; open a reference file to confirm it loads

## Public-facing language

Everything committed here — README, CONTRIBUTING, SKILL.md, `references/`, `plugin.json` `description` — is in English. International audience, open contributions. Do not include personal file paths, internal notes, or non-English content in files that ship to the public repo.
