# Culturecards agent skills

This repository contains agent skills in the [Agent Skills](https://agentskills.io) format. Each skill lives in `skills/<skill-name>/SKILL.md` and can be installed into supported agents with the `skills` CLI.

## Install

Install the a skill from this repository:

```sh
npx skills add culturecards/skills --skill culturecards-mcp
```

To target particular agents, pass their names with `--agent` (repeat the option as supported by the CLI), for example:

```sh
npx skills add culturecards/skills --skill culturecards-mcp --agent codex --agent claude-code --agent opencode --agent pi --agent omp
```

The skill is a portable `SKILL.md`; agents that support the Agent Skills format can also load it using their native skill discovery mechanism.
