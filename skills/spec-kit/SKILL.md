---
name: spec-kit
description: "Practical guide for setting up and using the spec-kit CLI on a project. Covers installation, project initialization for Claude and OpenCode, the specify slash commands (/speckit.constitution, /speckit.specify, /speckit.plan, /speckit.tasks, /speckit.implement), and detecting current workflow state. Load this skill when a user mentions spec-kit, the specify CLI, .specify/ directory, speckit commands, or wants to initialize spec-driven development tooling on a project."
license: MIT
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Spec-Kit

Practical setup and usage guide for the [spec-kit CLI](https://github.com/github/spec-kit) — the tooling layer that scaffolds and drives the spec-driven development workflow.

> **Relationship to `spec-driven-development` skill**: This skill covers the *tooling* (CLI, commands, file structure, state detection). The `spec-driven-development` skill covers the *methodology* (what each phase means, how to write good specs, clarification process, etc.). Use both together for the full picture.

---

## Prerequisites

- Python 3.11+
- Git
- [uv](https://docs.astral.sh/uv/) package manager

---

## Running spec-kit

The preferred approach is to run spec-kit via `uvx` without installing it globally. This keeps your machine clean, makes version pinning explicit, and avoids PATH issues.

### Preferred: `uvx` (no install required)

```bash
# New project — replace vX.Y.Z with the latest release tag
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init <PROJECT_NAME> --ai <agent>

# Existing project
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init . --here --ai <agent>

# Run any other specify command the same way
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify <command>
```

Check the [Releases page](https://github.com/github/spec-kit/releases) for the latest tag.

### Alternative: persistent install

Only do this if you run spec-kit frequently and want it available as a bare `specify` command:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
specify version  # verify

# Upgrade later
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

> If `specify` isn't found after a persistent install, add uv's tool bin to your PATH:
> ```bash
> export PATH="$HOME/.local/bin:$PATH"
> ```

---

## Project Initialization

Run this once per project to scaffold the `.specify/` directory and agent command files:

```bash
# New project (uvx — preferred)
uvx --from git+https://github.com/github/spec-kit.git specify init <project-name> --ai <agent>

# Existing project (prompts before overwriting)
uvx --from git+https://github.com/github/spec-kit.git specify init . --here --ai <agent>
```

### `--ai` values by agent

| Agent      | Flag value       | Creates commands in          |
| ---------- | ---------------- | ---------------------------- |
| OpenCode   | `opencode`       | `.opencode/command/`         |
| Claude Code | `claude`        | `.claude/commands/`          |
| Copilot    | `copilot`        | `.github/copilot-instructions.md` |
| Cursor     | `cursor-agent`   | `.cursor/rules/`             |
| Codex      | `codex`          | `.codex/`                    |

Run `specify integration list` to see all supported agents.

### What gets created

```
.specify/
├── memory/
│   └── constitution.md        # Created in Phase 1
├── features/                  # Feature specs live here (some presets use specs/ instead)
├── scripts/bash/              # Helper scripts used by slash commands
│   ├── create-new-feature.sh
│   ├── setup-plan.sh
│   ├── check-prerequisites.sh
│   └── update-agent-context.sh
└── templates/                 # Spec, plan, and tasks templates

.opencode/command/             # (OpenCode) or .claude/commands/ (Claude Code)
├── speckit.constitution.md
├── speckit.specify.md
├── speckit.clarify.md
├── speckit.plan.md
├── speckit.tasks.md
├── speckit.implement.md
└── speckit.analyze.md         # optional
```

---

## Slash Commands

After `specify init`, these commands are available in your agent:

| Command | Purpose |
| --- | --- |
| `/speckit.constitution` | Create or update the project constitution (Phase 1) |
| `/speckit.specify <description>` | Create a feature spec from a description (Phase 2) |
| `/speckit.clarify` | Resolve ambiguities before planning (Phase 3) |
| `/speckit.plan <tech notes>` | Generate implementation plan (Phase 4) |
| `/speckit.tasks` | Break plan into ordered tasks (Phase 5) |
| `/speckit.analyze` | Cross-artifact consistency check (optional, before Phase 6) |
| `/speckit.implement` | Execute tasks (Phase 6) |

### Example workflow

```
/speckit.constitution Focus on simplicity, test coverage ≥80%, TypeScript strict mode

/speckit.specify A dashboard showing real-time sensor readings with alerting thresholds

/speckit.clarify

/speckit.plan React + Vite frontend, Node.js WebSocket backend, SQLite for alert config

/speckit.tasks

/speckit.implement
```

---

## Detecting Project State

Before diving in, check where you are in the workflow:

```bash
# Is spec-kit initialized?
[ -d ".specify" ] && echo "Initialized" || echo "Run: uvx --from git+https://github.com/github/spec-kit.git specify init . --here --ai opencode"

# Is there a constitution?
[ -f ".specify/memory/constitution.md" ] && echo "Constitution exists" || echo "Run: /speckit.constitution"

# What features exist?
ls .specify/features/ 2>/dev/null || ls .specify/specs/ 2>/dev/null || echo "No features yet"

# What's the latest feature's phase?
LATEST=$(ls -d .specify/features/[0-9]* .specify/specs/[0-9]* 2>/dev/null | sort -V | tail -1)
if [ -n "$LATEST" ]; then
  echo "Latest feature dir: $LATEST"
  [ -f "$LATEST/plan.md" ]  && echo "  ✅ plan.md"  || echo "  ❌ plan.md (run /speckit.plan)"
  [ -f "$LATEST/tasks.md" ] && echo "  ✅ tasks.md" || echo "  ❌ tasks.md (run /speckit.tasks)"
fi
```

### Current Phase Detection

To determine exactly which phase the latest feature is in:

```bash
FEATURE_DIR=$(ls -d .specify/features/[0-9]* .specify/specs/[0-9]* 2>/dev/null | sort -V | tail -1)

if [ ! -f ".specify/memory/constitution.md" ]; then
  echo "Phase: constitution → run /speckit.constitution"
elif [ -z "$FEATURE_DIR" ]; then
  echo "Phase: specify → run /speckit.specify <description>"
elif [ -f "$FEATURE_DIR/spec.md" ] && ! grep -q "## Clarifications" "$FEATURE_DIR/spec.md"; then
  echo "Phase: clarify → run /speckit.clarify"
elif [ ! -f "$FEATURE_DIR/plan.md" ]; then
  echo "Phase: plan → run /speckit.plan"
elif [ ! -f "$FEATURE_DIR/tasks.md" ]; then
  echo "Phase: tasks → run /speckit.tasks"
elif grep -q "\- \[ \]" "$FEATURE_DIR/tasks.md"; then
  echo "Phase: implement → run /speckit.implement"
else
  echo "Phase: complete ✅"
fi
```

---

## Helper Scripts

The scripts in `.specify/scripts/bash/` are called by the slash commands but can also be run directly:

```bash
# Create a new feature branch and spec file, outputs JSON
.specify/scripts/bash/create-new-feature.sh --json "my-feature-name"
# → {"BRANCH_NAME": "001-my-feature-name", "SPEC_FILE": "..."}

# Set up the plan phase for the current feature, outputs JSON
.specify/scripts/bash/setup-plan.sh --json
# → {"FEATURE_SPEC": "...", "IMPL_PLAN": "...", "SPECS_DIR": "...", "BRANCH": "..."}

# Update agent context files after planning
.specify/scripts/bash/update-agent-context.sh opencode
```

---

## Upgrading spec-kit

With `uvx`, there's nothing to upgrade — just pin a newer tag in your command:

```bash
# Change vX.Y.Z to the new release tag and re-run as normal
uvx --from git+https://github.com/github/spec-kit.git@vX.Y.Z specify init . --here --ai <agent>
```

If you're using the persistent install, upgrade with:

```bash
uv tool install specify-cli --force --from git+https://github.com/github/spec-kit.git@vX.Y.Z
specify version
```

---

## Extensions & Presets

spec-kit supports community extensions (new commands) and presets (template overrides):

```bash
# Browse and install extensions
uvx --from git+https://github.com/github/spec-kit.git specify extension search
uvx --from git+https://github.com/github/spec-kit.git specify extension add <name>

# Browse and install presets
uvx --from git+https://github.com/github/spec-kit.git specify preset search
uvx --from git+https://github.com/github/spec-kit.git specify preset add <name>
```

See the [community extensions catalog](https://speckit-community.github.io/extensions/) for what's available.

---

## Troubleshooting

**`specify: command not found`** — You're using the persistent install path but uv's tool bin isn't in PATH. Either add it:
```bash
export PATH="$HOME/.local/bin:$PATH"
```
Or switch to `uvx` (the preferred approach) which doesn't require PATH setup.

**Slash commands not appearing** — Re-run `specify init` via `uvx` to regenerate command files:
```bash
uvx --from git+https://github.com/github/spec-kit.git specify init . --here --ai <agent>
```

**`--here` prompts for confirmation** — Expected if the directory is non-empty. Confirm to proceed; it won't overwrite your source files, only `.specify/` and agent command directories.

**Wrong feature directory** — Some presets use `.specify/specs/` instead of `.specify/features/`. Check which exists in your project.
