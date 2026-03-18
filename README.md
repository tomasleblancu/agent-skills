# Fizko Agent Skills

Agent skills for [Fizko](https://fizko.ai), built on the open [Agent Skills](https://agentskills.io) format.

## Installation

```bash
npx skills add tomasleblancu/agent-skills
```

## Environment Variables

Set these environment variables before using the skills:

```bash
export FIZKO_API_KEY="your-api-key"
export FIZKO_API_URL="https://api.fizko.ai"  # optional, defaults to production
```

| Variable | Description |
|----------|-------------|
| `FIZKO_API_KEY` | API key from Fizko. Run `fizko login` to authenticate interactively instead. |
| `FIZKO_API_URL` | Fizko API host. Defaults to `https://api.fizko.ai`. |

## Path selection

Prefer the `fizko` CLI when it is installed and already authenticated. The skill docs show CLI-first flows where the CLI already covers the task well.

Use the `FIZKO_API_KEY` env var as the fallback path when the CLI is unavailable or for scripted/automated use cases.

If you use the CLI path, start with `fizko status` to verify authentication.

## What are Agent Skills?

Agent Skills are folders of instructions, scripts, and resources that agents can discover and use to perform tasks more accurately. Each skill is a directory with a `SKILL.md` entrypoint and optional supporting files.

```
my-skill/
├── SKILL.md          # Required: instructions + metadata
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
```

Skills use progressive disclosure: agents load only the name and description at startup, then read full instructions when a task matches. This keeps context usage efficient while giving agents access to detailed knowledge on demand.

## Available skills

- **fizko**: Query tax documents, accounting reports, bank movements, and reconciliation data. Classify movements, reconcile with obligations, create journal entries, and more.

## SKILL.md format

Each skill requires a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: my-skill
description: What this skill does and when to use it.
---

# My Skill

Instructions for the agent...
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Lowercase identifier (letters, numbers, hyphens) |
| `description` | Yes | When to use this skill (max 1024 chars) |

## Learn more

- [Agent Skills specification](https://agentskills.io/specification)
- [Authoring best practices](https://agentskills.io/authoring)
