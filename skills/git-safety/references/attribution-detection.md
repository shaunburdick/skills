# Attribution Detection Reference

Supporting detail for the **AI Commit Attribution** section of
`../SKILL.md`. Agents should load this file when they need the full
per-harness signal matrix, the OpenCode v2 environment reality, mixed
environment behavior, or the parsing commands.

## Detection Signal Matrix

The hook treats the session as AI when **any** of these env vars is set to a
non-empty value (not just `1`):

| Harness | Detection env var(s) | Notes |
|---|---|---|
| Standard (emerging) | `AI_AGENT` | `CI=true`-style convention (agentsmd/agents.md#136); `opencode` yields `Generated-By: opencode`, other values yield `ai-agent` |
| Legacy `AGENT` | `AGENT` | Any non-empty value matches |
| Goose / Amp | `AGENT=goose` / `AGENT=amp` | Adopted the `AGENT` convention with non-`1` values |
| Claude Code | `CLAUDE_CODE`, `CLAUDE_CODE_ENTRYPOINT` | Set in subprocesses spawned by Claude Code |
| Cursor | `CURSOR_AGENT` | |
| Gemini CLI | `GEMINI_CLI` | |
| Codex CLI | `CODEX_SANDBOX` | |
| Augment | `AUGMENT_AGENT` | |
| Cline | `CLINE_ACTIVE` | |
| OpenCode v1 (Vercel) | `OPENCODE_CLIENT` | v1 repo archived; continued as Crush |
| OpenCode v2 | **none — claim explicitly** | See next section |
| Legacy `OPENCODE` | `OPENCODE` | Historical |
| Explicit claim | `OPENCODE_AGENT` / `OPENCODE_MODEL` | Either var alone is a sufficient claim signal |

Detection libraries that maintain similar matrices: `@vercel/detect-agent`,
Bun's `isAIAgent()`.

## OpenCode v2 Environment Reality

OpenCode v2 (`anomalyco/opencode`) sets exactly one env var on every spawned
process — `OPENCODE_TERMINAL=1` — unconditionally, on agent tool shells AND
the human's interactive TUI terminal. It never sets `OPENCODE`, `AGENT`,
`OPENCODE_AGENT`, or `OPENCODE_MODEL`. Two consequences:

- `OPENCODE_TERMINAL` is **not** an AI signal. The hook never attributes from
  it alone, so human commits in the TUI terminal are never misattributed.
- Agent commits must carry an explicit claim, because each bash tool call
  spawns a **fresh login shell** — env vars exported at session start do
  **not** persist to a later `git commit` call.

### Why session-start exports don't work on v2

Each `bash` tool invocation runs as a new login shell (`bash -lc "..."`), so
the process environment is rebuilt from the server's env plus `TERM` and
`OPENCODE_TERMINAL`. An `export` performed by one tool call is gone by the
next. The claim must therefore ride on the same command line as the commit.

### OpenCode v2 claim convention

```bash
git-agent-commit -m "feat: add widget"              # Generated-By: opencode
OPENCODE_AGENT="my-agent" OPENCODE_MODEL="my-model" \
  git-agent-commit -m "feat: add widget"            # Generated-By: my-agent (model: my-model)
# identical inline form:
AI_AGENT=opencode OPENCODE_AGENT="my-agent" OPENCODE_MODEL="my-model" \
  git commit -m "feat: add widget"
```

Trailer defaults: no `OPENCODE_AGENT` → harness name (`opencode` for the
canonical claim); `OPENCODE_AGENT` only → the agent name; both set →
`<agent> (model: <model>)`.

### Unclaimed OpenCode sessions

When `OPENCODE_TERMINAL=1` is present with no claim (an agent that forgot, or
a human committing inside the TUI terminal), the hook prints a stderr warning
and appends **nothing**. The commit always succeeds. Humans inside the
OpenCode TUI terminal see the warning too — that is deliberate: it is the
price of making agent silent-failure impossible, and no trailer is ever
falsely added.

## Mixed Environments (AI + Human Commits)

| Scenario | Detection | Hook behavior |
|---|---|---|
| Agent on a harness with a marker (Claude Code, Cursor, Goose, Amp, ...) | harness env var | appends `Generated-By: <harness>` |
| Agent on OpenCode v2, claimed | `AI_AGENT` via `git-agent-commit` | appends default or rich attribution |
| Agent on OpenCode v2, forgot to claim | `OPENCODE_TERMINAL` only | stderr warning; **no** trailer |
| Human terminal (outside OpenCode) | no vars | exits silently, no trailer |
| Human inside OpenCode TUI terminal | `OPENCODE_TERMINAL` only | stderr warning; **no** trailer |

## Parsing Attribution

Find all AI-generated commits:

```bash
git log --trailer=Generated-By --oneline
```

Extract unique agents/models:

```bash
git log --format='%(trailers:valueonly,separator=%x2C,unfold,separator=%x2Ckey=Generated-By)' | sort | uniq -c | sort -rn
```

## Ecosystem Status

- `AI_AGENT` standard: proposed in
  [agentsmd/agents.md#136](https://github.com/agentsmd/agents.md/issues/136)
  (Jan 2026) — a `CI=true`-style convention for agent-runtime detection.
- OpenCode v2 does not yet set an AI marker on tool-executed ptys; an
  upstream feature request is the long-term fix (tracked separately).
- The detection matrix here is maintained in sync with `scripts/prepare-commit-msg`.