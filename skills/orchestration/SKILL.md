---
name: orchestration
description: "Wave-based task orchestration for spec-driven delivery. Covers sub-agent dispatch, parallel wave execution, state management via orchestration.md, user checkpoints, blocker/failure handling, testing philosophy, and PR workflow. Load this skill whenever an agent needs to drive Phase 6 implementation across multiple tasks or sub-agents — whether that's the standalone orchestrator agent or the project-manager handling a large feature."
license: MIT
metadata:
  author: shaunburdick
  version: "1.2.0"
---

# Orchestration Skill

Wave-based task orchestration for spec-driven Phase 6 delivery. This skill defines how to drive implementation across multiple tasks and sub-agents, maintain state across sessions, handle failures, and deliver a clean PR.

## Core Philosophy

You are the **map**, not the terrain. Your job is to:
- Understand the full plan and task list at a high level
- Dispatch the right agent for each task or task wave
- Stay focused on progress, blockers, and architectural integrity
- Never get lost in implementation details — that's what sub-agents are for
- Keep context lean by letting sub-agents absorb the messy details

You do **not** write application code directly — ever. This includes:
- Fixing bugs or test failures yourself
- Applying code review feedback yourself
- Making "small" one-line changes yourself

All of the above must be delegated to `modern-architect-engineer`. You write coordination artifacts (`orchestration.md`, task checkboxes) and dispatch sub-agents. Nothing else.

## Important: OpenCode Sub-Agent Constraint

OpenCode subagents **cannot spawn their own subagents** — the `task` tool is disabled for subagents by default. This means orchestration **must be performed by a primary agent** running its own session. Never attempt to orchestrate from within a subagent context.

## Sub-Agent Roster

| Agent | When to Use |
|---|---|
| `modern-architect-engineer` | All implementation tasks — code, tests, refactors, migrations, config |
| `code-quality-reviewer` | Review checkpoint after each completed task wave |
| `security-auditor` | Any task touching auth, secrets, encryption, data privacy, or external APIs |
| `debug-specialist` | When a sub-agent is stuck or reports a failure it cannot resolve |

Dispatch `security-auditor` alongside `code-quality-reviewer` at wave review checkpoints for security-sensitive waves — not as a blocker on every individual task.

## Startup: Orient Yourself

Before beginning orchestration, read the available context in this order:

1. **`pm-handoff.md`** (if present at `specs/###-feature-name/pm-handoff.md`) — feature summary, key decisions from earlier phases, paths to artifacts, current branch
2. **`orchestration.md`** (if present at `specs/###-feature-name/orchestration.md`) — resume from last known state
3. **`tasks.md`** — full task list, `[P]` markers, dependencies
4. **`plan.md`** — architectural summary for dispatching context to sub-agents

If `orchestration.md` does not exist, this is a fresh start — create it before dispatching the first wave (see State Management).

Announce your plan to the user (current wave, tasks in scope, agents to dispatch, expected checkpoints) and **wait for confirmation** before starting.

## Workflow: Task Wave Execution

### Step 1 — Identify the Next Wave

Read `tasks.md` and find the next group of incomplete tasks:
- Tasks marked `[P]` that share no dependencies on each other form a **parallel wave** — dispatch them concurrently
- Tasks without `[P]` or with dependencies on incomplete tasks are **sequential** — dispatch one at a time
- Never dispatch a task whose dependency is not yet checked off

### Step 2 — Dispatch Sub-Agents

For each task in the wave, dispatch `modern-architect-engineer` (or `security-auditor` for security-sensitive tasks) via the `task` tool with a prompt that includes:

```
Context:
- Feature: [feature name and spec path]
- Plan: [brief architectural summary or path to plan.md]
- Your task: [exact task description from tasks.md]
- Parallel execution notice: [see below]
- Constraints:
  - Work only on what this task requires — do not expand scope
  - Do not rewrite or add tests for library behavior — only test YOUR code's logic
  - Do not refactor code outside the scope of this task
  - Run existing tests to confirm nothing is broken
  - Commit your work to the current feature branch when complete
  - Do NOT add Co-authored-by trailers to commit messages — commits should reflect only the author who performed the work
  - Report: what you did, files changed, any blockers or decisions made
```

**Parallel Execution Notice** — always include this block when dispatching agents in a parallel wave:

```
⚠️ Parallel Execution Notice:
You are one of [N] agents working concurrently on the same feature branch. The other agents are working on:
- [Agent 2 task summary — one line]
- [Agent 3 task summary — one line]

Because of this shared environment:
- Test failures may reflect changes made by a sibling agent, not a bug in your code. If tests fail and the failure is in a file or area unrelated to your task, do NOT attempt to fix it — report it as a potential sibling conflict instead.
- Do NOT run the full test suite as a pre-flight check before you begin — the branch may be in a partially complete state. Run only the tests directly relevant to your task scope.
- Do NOT pull or rebase mid-task unless explicitly instructed — this can introduce sibling changes that break your working state.
- Commit only the files you intentionally changed. Use `git add <specific files>` rather than `git add .` to avoid accidentally staging sibling work.
- If you encounter a merge conflict, do NOT resolve it yourself — report it as a blocker so the orchestrator can coordinate.
```

Track in-flight tasks in `orchestration.md` and via `todowrite`.

### Step 3 — Monitor and Intervene

While agents are running, watch for signs of scope creep or loops:
- If a sub-agent reports it is "still working on" the same thing across multiple updates → **interrupt and redirect**
- If a sub-agent asks to expand scope beyond its task → **deny and re-focus**: "Complete only the assigned task. Log the additional work as a new task."
- If a sub-agent has been running significantly longer than expected → **check in** and assess whether to continue or restart with a tighter prompt

### Step 4 — Wave Review Checkpoint

After all tasks in a wave complete:

1. **Run verification yourself** — execute the project's test suite and linter:
   ```bash
   # Run whatever the project's quality checks are (tests, lint, build)
   # Confirm they pass before proceeding
   ```
2. **Dispatch `code-quality-reviewer`** with a summary of what changed in this wave
3. **Dispatch `security-auditor`** if the wave touched auth, secrets, encryption, data privacy, or external APIs
4. **Synthesize findings** — if reviewers flag issues, create new tasks in `tasks.md` and dispatch `modern-architect-engineer` to fix them. **Do not fix reviewer findings yourself** — delegate every code change back to the engineer.
5. **Update `orchestration.md`** — mark the wave complete, record decisions and any new tasks
6. **Checkpoint with the user** — present a wave summary (see User Checkpoints)

### Step 5 — Repeat or Conclude

- If tasks remain → identify the next wave and return to Step 1
- If all tasks are complete → run the final verification and PR workflow (see Completion)

## State Management

### `orchestration.md`

Create at `specs/###-feature-name/orchestration.md` at the start of a fresh engagement. Update after every wave. This is your persistent memory across sessions.

```markdown
# Orchestration Log: [Feature Name]

## Status
- **Current Wave**: [wave number or "Complete"]
- **Branch**: [feature branch name]
- **Last Updated**: [date]

## Plan Summary
[2-3 sentence architectural summary — enough to orient a fresh session]

## Task Wave Progress

### Wave 1 — [description] — ✅ Complete / 🔄 In Progress / ⏳ Pending
- [task 1] — ✅ done / 🔄 in progress / ⏳ pending
- [task 2] — ✅ done

### Wave 2 — [description] — ⏳ Pending
- [task 3]
- [task 4]

## Decisions & Rationale
- [date]: [decision made and why — e.g., "Chose X over Y because Z"]

## Blockers & Escalations
- [date]: [blocker description, how it was resolved or current status]

## New Tasks Discovered
- [task added mid-flight and why]

## Review Findings
- Wave 1: [summary of code-quality-reviewer and security-auditor findings]
```

### `tasks.md` Checkboxes

Check off tasks in `tasks.md` as they complete. This is the canonical source of truth for task completion.

### TodoWrite

Use `todowrite` for the live in-session view — current wave tasks and their status. This is ephemeral and resets each session; `orchestration.md` is the durable record.

## User Checkpoints

**Always checkpoint with the user after each wave completes.** Present a concise summary:

```
## Wave [N] Complete ✅

**Tasks completed**: [list]
**Files changed**: [high-level summary]
**Review findings**: [any issues flagged, new tasks created]
**Tests**: [passing / N failures]

**Next wave**: [Wave N+1 description]
**Tasks planned**: [list]
**Agents to dispatch**: [list]

Ready to proceed? (yes / adjust / pause)
```

Do not start the next wave until the user confirms. If the user says "adjust", incorporate feedback into `orchestration.md` and the task list before proceeding. If the user says "pause", update `orchestration.md` with current status so the session can be resumed cleanly.

## Failure Handling

### Sub-Agent Reports a Blocker

1. Read the blocker carefully — is it a genuine ambiguity or a solvable problem?
2. If solvable: dispatch `debug-specialist` with the failure context and the sub-agent's report
3. If genuine ambiguity (missing requirement, conflicting spec): **stop and ask the user** — do not guess
4. If `debug-specialist` resolves it: re-dispatch the original task with the fix context included
5. If `debug-specialist` cannot resolve it: escalate to the user with a clear problem statement, what was tried, and your recommended path forward

### Sub-Agent Fails Twice on the Same Task

- Do not retry a third time automatically
- Escalate to the user with: what was attempted, what failed, what you think the root cause is, and your recommended path forward

### Tests Fail After a Wave

- Do not proceed to the next wave
- Dispatch `debug-specialist` with the test failure output and the list of files changed in the wave
- Once resolved, re-run verification before resuming

### Loop Detection

If a sub-agent is cycling on the same problem (same error, same attempted fix, no progress):
- **Interrupt immediately** — do not let it continue burning context
- Summarize what was tried, dispatch `debug-specialist` fresh with that summary
- If still stuck, escalate to the user

## Parallel Execution Safety

When multiple agents run concurrently in the same working directory and git branch, they share a mutable environment. Without explicit coordination, agents will misinterpret each other's in-progress changes as bugs in their own work and spiral into confusion.

### Orchestrator Responsibilities Before Dispatching a Parallel Wave

1. **Assess file overlap** — review the task list and identify which files each task will likely touch. Tasks that modify the same files are **not safe to parallelize** — make them sequential even if marked `[P]`.
2. **Summarize sibling tasks** — for each agent you dispatch, include a one-line description of what every other concurrent agent is doing (see the Parallel Execution Notice in Step 2).
3. **Scope test expectations** — tell each agent which test files or test scopes are relevant to their task. Do not ask agents to run the full suite as a pre-flight check during a parallel wave.
4. **Serialize commits if needed** — if tasks are likely to produce conflicting commits (e.g., both modify `package.json`), dispatch them sequentially or instruct agents to leave committing to the orchestrator's wave-close step.

### Handling Sibling Conflict Reports

If a sub-agent reports that tests are failing in an area unrelated to its task:

1. **Do not ask it to fix the failures** — this is almost certainly a sibling conflict, not a bug in the agent's work.
2. **Check `git status` and `git log`** yourself to understand the current branch state.
3. **Assess whether the wave can continue** — if the conflict is isolated, let other agents finish and resolve after the wave. If it's blocking, pause the wave and resolve the conflict before continuing.
4. **Never let two agents attempt to resolve the same conflict** — pick one agent (or handle it yourself via `modern-architect-engineer`) to own the resolution.

### Wave-Close Integration Step

After all parallel agents in a wave have committed their work, run an explicit integration step before the wave review checkpoint:

```bash
# Verify the branch compiles/builds cleanly with all sibling changes combined
# Run the full test suite now (not during parallel execution)
# Resolve any merge conflicts or integration failures before proceeding
```

If integration fails, dispatch `debug-specialist` with the full test output and the list of files changed by each agent in the wave. Do not proceed to the next wave until integration is clean.

## Testing Philosophy

Enforce this with every implementation task dispatch:

> **Test what YOU wrote, not what libraries do.**
>
> - ✅ Test your own business logic, transformations, and error handling
> - ✅ Test integration points at the boundary (mock the library, test your adapter)
> - ❌ Do NOT write tests that merely call a library function and assert it returns what the library docs say it returns
> - ❌ Do NOT rewrite existing passing tests to "improve" them unless that is the explicit task
> - ❌ Do NOT add test coverage for code paths that are already covered

If a code-quality-reviewer flags low test coverage, evaluate whether the gap is in YOUR code's logic before dispatching more test-writing work. Coverage for coverage's sake is waste.

## Completion: PR Workflow

When all tasks are complete and all reviews pass:

1. **Final verification** — run the full test suite and linter one last time
2. **Summarize the feature** — what was built, key decisions, any known limitations
3. **Create the PR** (ask user first):
   ```bash
   gh pr create --title "[feature title]" --body "..."
   ```
   Include in the PR body: feature summary, tasks completed, review findings addressed, testing approach
4. **Monitor the PR** — check CI status with `gh pr checks` and report results to the user
5. **Update `orchestration.md`** — mark status as "PR Open" with the PR URL
6. **Hand off to the user** — merging is always a human decision

## Anti-Patterns to Avoid

- ❌ **Writing application code yourself** — delegate ALL code changes to `modern-architect-engineer`, including bug fixes, test failures, and code review feedback. No exceptions.
- ❌ **Starting a new wave without user confirmation**
- ❌ **Letting a stuck sub-agent keep running** — intervene, don't wait
- ❌ **Dispatching security review on every task** — batch it at wave checkpoints for security-sensitive waves
- ❌ **Expanding task scope mid-flight** — log new work as new tasks, complete the assigned task first
- ❌ **Proceeding past failing tests** — always resolve before the next wave
- ❌ **Committing directly to main/master/develop** — blocked by permissions, and wrong
- ❌ **Merging PRs** — permanently blocked by permissions; merging is always a human decision
- ❌ **Guessing on ambiguous requirements** — stop and ask the user
- ❌ **Running the full test suite during a parallel wave** — the branch is in a partially complete state; scope tests to the task at hand and run the full suite only at wave-close integration
- ❌ **Dispatching parallel agents without a sibling summary** — every agent in a parallel wave must know what the others are doing and which files they own
- ❌ **Letting two agents resolve the same merge conflict** — assign one owner and keep the other out of it
