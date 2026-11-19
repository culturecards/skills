# Culturecards agent skills

This repository contains agent skills in the [Agent Skills](https://agentskills.io) format. Each skill lives in `skills/<skill-name>/SKILL.md` and can be installed into supported agents with the `skills` CLI.

## Install

### npm `skills` CLI

Install the skill for all detected supported agents:

```sh
npx skills add culturecards/skills --skill culturecards-mcp
```

Or target agents explicitly:

```sh
npx skills add culturecards/skills --skill culturecards-mcp --agent claude-code --agent codex --agent opencode
```

### Claude Code

```sh
npx skills add culturecards/skills --skill culturecards-mcp --agent claude-code
```

Alternatively, place the skill directory in `~/.claude/skills/culturecards-mcp/` for personal use or `.claude/skills/culturecards-mcp/` in a project. See [Claude Code Skills](https://code.claude.com/docs/en/skills).

### Claude web (claude.ai)

Package the skill directory as a ZIP with `SKILL.md` at the skill folder's root, then upload it in [Customize → Skills](https://claude.ai/customize/skills) and enable it. Claude's web Skills feature requires code execution to be enabled and is available on supported plans. See Anthropic's [guide to creating custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills).

### Codex

```sh
npx skills add culturecards/skills --skill culturecards-mcp --agent codex
```

Codex can also discover skills from its configured skill directories. See [Codex Skills](https://developers.openai.com/codex/skills).

### ChatGPT web

If Skills are available in your ChatGPT workspace, upload a ZIP containing the `culturecards-mcp/` folder (with `SKILL.md` inside it) through the Skills interface, then enable it. Availability and UI labels can vary by plan or workspace. The `agents/openai.yaml` file supplies GPT-specific display metadata; it does not connect an MCP server. For API-based skill upload and use, see OpenAI's [Tools: Skills guide](https://developers.openai.com/api/docs/guides/tools-skills).

### OpenCode

```sh
npx skills add culturecards/skills --skill culturecards-mcp --agent opencode
```

The skill is a portable `SKILL.md`; agents that support the Agent Skills format can load it using their native skill discovery mechanism. See [OpenCode skills](https://opencode.ai/docs/skills/).

## Skill authoring guidance

Skills follow the [Agent Skills format](https://agentskills.io/specification) and should be designed around a clear, repeatable workflow:

- Keep the skill in its own directory with exactly one `SKILL.md` manifest. Include YAML front matter with a concise `name` and a `description` that explains both the skill's purpose and when to use it.
- Put the essential instructions in `SKILL.md`. Move substantial supporting material into clearly named `references/`, repeatable utilities into `scripts/`, and reusable templates or examples into `assets/`; link to those files from the skill.
- Make instructions actionable and scoped to the skill's actual use cases. Avoid duplicating general agent guidance or putting large reference material in the manifest.
- Treat skill instructions and supporting files as privileged input: review them for prompt-injection, unsafe commands, and unintended data access. Require user approval for destructive or other high-impact actions.
- Keep examples and references accurate, and verify manifest metadata and the skill's file layout before publishing or installing it.

See OpenAI's [Tools: Skills guide](https://developers.openai.com/api/docs/guides/tools-skills) for platform-specific discovery, execution, upload, and security details. Skills are user-level instructions, not a security boundary; execution environments and approval checks must enforce safety.

Claude also supports the Agent Skills format. Its required `SKILL.md` front matter is `name` and `description`; Claude-specific behavior can use additional supported front matter fields, but there is no Claude equivalent to GPT's `agents/openai.yaml` UI metadata file. The OpenAI YAML sidecar is platform-specific and can be ignored by Claude. See Anthropic's [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview), [authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices), and [Claude Code Skills guide](https://code.claude.com/docs/en/skills).
