# Agent Skills

A personal collection of [Agent Skills](https://agentskills.io/) for AI coding agents.

## Install

```bash
# Install all skills
npx skills add shaunburdick/skills

# List available skills without installing
npx skills add shaunburdick/skills --list

# Install a specific skill
npx skills add shaunburdick/skills --skill spec-driven-development

# Install globally (available across all projects)
npx skills add shaunburdick/skills -g

# Install to specific agents only
npx skills add shaunburdick/skills -a claude-code -a opencode
```

## Available Skills

### [spec-driven-development](skills/spec-driven-development/)

Structured spec-driven development workflow: clarify requirements, write specs, plan architecture, break into tasks, then implement. Use when starting a new feature, building from scratch, or when requirements are unclear.

### [style](skills/style/)

Install @shaunburdick's personal style configuration: shared `.editorconfig` for any project, plus `eslint-config-shaunburdick` for JavaScript/TypeScript projects.

## Skill Structure

Each skill lives in `skills/<skill-name>/` and contains:

```
skills/
└── skill-name/
    ├── SKILL.md          # Required: YAML frontmatter + agent instructions
    ├── scripts/          # Optional: helper scripts the agent can run
    ├── references/       # Optional: supplementary documentation
    └── assets/           # Optional: templates, schemas, static resources
```

### `SKILL.md` Format

```markdown
---
name: skill-name
description: What this skill does and when to use it. Be specific — agents use this to decide when to activate the skill.
license: MIT
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Skill Title

Instructions for the agent...
```

See the [Agent Skills specification](https://agentskills.io/specification) for the full format reference.

## Contributing / Forking

Feel free to fork this repo and add your own skills. The `skills/` directory is the canonical location the `skills` CLI searches.

## License

MIT
