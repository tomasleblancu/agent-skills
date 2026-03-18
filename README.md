# Fizko Agent Skills

Agent Skills for the [Fizko](https://fizko.ai) tax and accounting platform.

Compatible with Claude Code, Cursor, GitHub Copilot, Gemini CLI, and any [agentskills.io](https://agentskills.io)-compatible tool.

## Available skills

| Skill | Description |
|-------|-------------|
| [fizko](./skills/fizko/) | Query tax documents, accounting reports, bank movements, and reconciliation data from the Fizko API using the `fizko` CLI. |

## Installation

### Claude Code

```bash
claude skills install github:akashi-labs/agent-skills/skills/fizko
```

### Manual

Copy the `skills/fizko/` folder into your project's `.claude/skills/` directory.

## Requirements

The `fizko` CLI must be installed:

```bash
npm install -g fizko-cli
fizko login
```

## License

MIT
