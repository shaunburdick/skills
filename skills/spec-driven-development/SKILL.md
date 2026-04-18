---
name: spec-driven-development
description: Structured spec-driven development workflow. Use when starting a new feature, building a system from scratch, or when requirements are unclear. Guides the agent through six phases: constitution → specification → clarification → plan → tasks → implement. Never skip phases.
license: MIT
compatibility: Designed for coding agents with file read/write and bash execution capabilities.
metadata:
  author: shaunburdick
  version: "2.0.0"
---

# Spec-Driven Development

A disciplined, phase-gated approach to software development based on the [SDD philosophy](https://github.com/github/spec-kit/blob/main/spec-driven.md): **specifications are the source of truth — code serves specifications, not the other way around**.

The most common failure mode of AI-assisted coding is jumping straight to implementation before the problem is understood. This workflow prevents that.

## Tooling: spec-kit

This workflow uses [spec-kit](https://github.com/github/spec-kit) to scaffold and manage the specification directory structure.

```bash
# New project
uvx --from git+https://github.com/github/spec-kit.git specify init my-project --ai <agent>

# Existing project
uvx --from git+https://github.com/github/spec-kit.git specify init . --here --ai <agent>
```

Supported `--ai` values: `copilot`, `claude`, `opencode`, `cursor-agent`, `codex`, `windsurf`, and others. This creates agent-specific command files and the `.specify/` directory structure.

## Directory Structure

```
project/
├── .specify/
│   ├── memory/
│   │   └── constitution.md          # Project principles — created FIRST
│   ├── features/
│   │   ├── 001-feature-name.md      # Feature specifications
│   │   └── 002-feature-name.md
│   └── scripts/bash/                # spec-kit helper scripts
└── specs/                           # Created during planning phase
    └── 001-feature-name/
        ├── plan.md
        ├── research.md
        ├── data-model.md
        ├── contracts/
        ├── quickstart.md
        └── tasks.md
```

## Phases Overview

| # | Phase | Owner | Output | Gate |
|---|-------|-------|--------|------|
| 1 | Constitution | Planner | `.specify/memory/constitution.md` | User approves |
| 2 | Specification | Planner | `.specify/features/###-name.md` | User approves |
| 3 | Clarification | Planner | Updated specs, zero ambiguities | Zero `[NEEDS CLARIFICATION]` markers |
| 4 | Plan | Architect | `specs/###-name/plan.md` + supporting docs | User approves |
| 5 | Tasks | Architect | `specs/###-name/tasks.md` | User approves |
| 6 | Implement | Architect | Working, tested code | All acceptance criteria met |

**Never skip a phase.** Each gate exists to catch misunderstandings when they're cheap to fix.

---

## Phase 1: Constitution

Create `.specify/memory/constitution.md` **before anything else**. This is the north star for all decisions.

Include:
- **Core principles** (5–7 max — more dilutes them)
- **Technical decisions** (stack, deployment, storage) with rationale for each
- **Quality standards** (test coverage target, linting, type safety)
- **Anti-patterns to avoid** with examples
- **Success metrics** (product, technical, operational)

The constitution must be approved before writing any feature specs. All subsequent decisions must reference it. If a requirement conflicts with the constitution, resolve the conflict explicitly — don't silently ignore either.

---

## Phase 2: Specification

Create `.specify/features/###-feature-name.md` for each feature. See [references/feature-spec-template.md](references/feature-spec-template.md) for the full template.

Key sections:
- **Problem Statement** — one paragraph, what and why
- **User Stories** — `As a <role>, I want <action> so that <benefit>`
- **Functional Requirements** — `FR-001`, `FR-002`, … (WHAT, not HOW)
- **Non-Functional Requirements** — performance, security, compatibility
- **Acceptance Criteria** — specific, measurable, binary pass/fail
- **Out of Scope** — explicitly named exclusions
- **Edge Cases** — documented with expected behavior

**What makes a requirement "executable":**
- Concrete, not abstract: "respond within 500ms" not "respond quickly"
- Measurable: "≥80% test coverage" not "good test coverage"
- Explicit error handling: document 404, 500, timeout, empty state behavior
- Example inputs/outputs for all user-facing elements

---

## Phase 3: Clarification

Before handing off to planning, eliminate every ambiguity.

**Process:**
1. Ask 3–5 targeted questions at a time (not a wall of 20)
2. For each question, explain *why* the answer matters
3. Give an example of what a complete answer looks like
4. Document every answer as a new concrete requirement (`FR-007a`, `FR-007b`, etc.)
5. Update the spec version (`v1.0` → `v1.1`) and add a "Clarifications Applied" section

**Exit criteria:** Zero `[NEEDS CLARIFICATION]` markers remain. Every question has been answered and documented as a requirement. A developer could implement from this spec without asking further questions.

---

## Phase 4: Plan

Create a feature branch and run the planning setup:

```bash
git checkout -b 001-feature-name
.specify/scripts/bash/setup-plan.sh --json
```

Produce the following files in `specs/###-feature-name/`:

**`plan.md`** — Technical context, constitution alignment check, concrete project structure (no "Option A / Option B"), architecture overview, key decisions with rationale.

**`research.md`** — Technology choices with specific versions, rationale, and alternatives considered and rejected.

**`data-model.md`** — Complete entity definitions: fields with types and constraints, relationships, validation rules, state transitions. See [references/data-model-guide.md](references/data-model-guide.md).

**`contracts/`** — API/event schemas (OpenAPI, GraphQL, or event definitions).

**`quickstart.md`** — Setup instructions, how to run tests, key flows to validate manually, expected outputs.

After planning:
```bash
.specify/scripts/bash/update-agent-context.sh <agent>
```

**Gate:** Present the plan. Do not proceed until approved.

---

## Phase 5: Tasks

Create `specs/###-feature-name/tasks.md`:

```markdown
# Tasks: <Feature Name>

- [ ] T-001: Set up project scaffold and install dependencies
- [ ] T-002: Implement data model / database schema
- [ ] T-003: [P] Write unit tests for core logic
- [ ] T-004: Implement core business logic
- [ ] T-005: Implement API layer
- [ ] T-006: Integration tests
- [ ] T-007: Update documentation
```

Rules:
- Each task completable in one sitting
- Mark parallel-safe tasks with `[P]`
- Order by dependency — foundational tasks first
- Test tasks sit alongside (not after) implementation tasks

**Gate:** Present tasks. Do not proceed until approved.

---

## Phase 6: Implement

Execute tasks in order, following the approved plan and referencing the constitution for quality standards.

### Code Quality (Non-Negotiable)

- ❌ No `eslint-disable`, `@ts-ignore`, `@ts-nocheck`, or any lint suppression — fix the code instead
- ❌ No `any` types in TypeScript — use proper interfaces and generics
- ✅ TDD preferred — write tests before or alongside implementation
- ✅ Every public function/method must have a doc comment
- ✅ Check off tasks in `tasks.md` as you complete them

### Pre-Commit Checklist (Every Commit)

- [ ] Full test suite passes
- [ ] Linting passes with zero errors or warnings
- [ ] Type checking passes
- [ ] Build succeeds
- [ ] The specific change was manually verified
- [ ] No debug code or `console.log` left behind
- [ ] Diff reviewed — all changed files are intentional

> "It should work" ≠ "I verified it works." Always run the checks.

---

## References

- [references/feature-spec-template.md](references/feature-spec-template.md) — Full spec template to copy
- [references/constitution-template.md](references/constitution-template.md) — Constitution template
- [references/data-model-guide.md](references/data-model-guide.md) — Data model authoring guide
- [SDD philosophy](https://github.com/github/spec-kit/blob/main/spec-driven.md) — The full spec-driven development manifesto
- [spec-kit repo](https://github.com/github/spec-kit) — Tooling source and docs
