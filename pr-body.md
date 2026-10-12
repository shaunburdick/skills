## Summary

Follow-up to #17, which scrubbed most retired OpenCode tool names but deliberately left three mentions of the retired `bash` tool name in `git-safety`. Those mentions named `bash` as if it were still an invocable tool, so any session loading the skill could attempt a `bash` tool call and get a real `No tool named bash` failure (tracked in the body of dotfiles PR #24 and finding F1 of `.agents/reviews/2026-W41-session-review.md`, which measured 105/105 failure on the retired name).

Changes:

1. `skills/git-safety/SKILL.md` — Agent Permission Configuration YAML example: permission key `bash:` → `shell:`.
2. `skills/git-safety/scripts/git-agent-commit` — header comment "each `bash` tool call spawns a fresh login shell" → "`shell` tool call".
3. `skills/git-safety/references/attribution-detection.md` — "The v2.0.15/bash tool input schema is…" reworded to "The retired `bash` tool (now `shell`; v2.0.15 input schema is…)", keeping the historical-API meaning intact.

## Test plan

- [x] Grep the three files for bare tool-name `bash` invocations — no invocable-looking `bash` tool references remain; only historical API references (clearly marked as v2.0.15/dev-era with file paths) are left, all qualified.
- [x] YAML example still parses as valid (`shell:` key with the same permission entries).

---
Generated-By: opencode (model: opencode-go/glm-5.3-flash)
