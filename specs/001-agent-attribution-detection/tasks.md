# Tasks: Agent Attribution Detection (OpenCode v2 + Cross-Harness)

Ordered by dependency. `[P]` marks parallel-safe tasks.

- [x] T-001: Rewrite `skills/git-safety/scripts/prepare-commit-msg` — FR-001..004, FR-008: cross-harness detection matrix (any non-empty value), harness-derived default attribution, `OPENCODE_TERMINAL` warn-only path, dedupe + merge/squash gates, bash 3.2 compatibility.
- [x] T-002: Add `skills/git-safety/scripts/git-agent-commit` — FR-006: exports `AI_AGENT` (default `opencode`), preserves `OPENCODE_AGENT`/`OPENCODE_MODEL`, `exec git commit "$@"`; executable bit set.
- [x] T-003: Add `skills/git-safety/scripts/test-prepare-commit-msg.sh` — functional harness covering AC-1..AC-9 via a temp repo; self-cleaning; exit nonzero on failure.
- [x] T-004: Rewrite the Attribution section of `skills/git-safety/SKILL.md` — FR-005: honest "How It Works", per-harness detection summary, OpenCode v2 claim convention (wrapper + inline env), remove session-start export step, sed-extraction for appending to existing hooks (single source of truth), verification commands, version 1.1.0 → 1.2.0. Deep detail moved to `references/attribution-detection.md` (per AGENTS.md guidance).
- [x] T-005: Update `skills/ai-attribution/SKILL.md` — FR-007: Git Commits surface references the cross-harness matrix; version 1.0.0 → 1.1.0.
- [x] T-006: Verify — `bash -n` on all scripts (OK), shellcheck not installed (noted), `test-prepare-commit-msg.sh` 13/13 green, sed-extraction install path validated end-to-end, no bash 4+ syntax (AC-10), docs free of stale claims (AC-11).
- [x] T-007: Release — `git identity` check (Shaun Burdick <github@shaunburdick.com>), commit `bda9bf8`, pushed to origin, PR opened: https://github.com/shaunburdick/skills/pull/9 (body carries the `Generated-By` footer per ai-attribution).