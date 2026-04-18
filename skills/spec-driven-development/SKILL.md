---
name: spec-driven-development
description: Structured spec-driven development workflow. Use when starting a new feature, building a system from scratch, or when requirements are unclear. Guides the agent through phases; clarify requirements > write spec > plan architecture > break into tasks > implement with tests.
license: MIT
compatibility: Designed for coding agents with file read/write and bash execution capabilities.
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Spec-Driven Development

A disciplined, phase-gated approach to software development that prevents the most common failure mode of AI-assisted coding: jumping straight to implementation before the problem is fully understood.

## When to Use

Activate this skill when:

- Starting a new feature or project
- Requirements are vague or ambiguous
- The user says "build me X" without further detail
- A task involves multiple components or systems
- You need to make architectural decisions

## Phases Overview

| Phase        | Output                                                  | Gate                        |
| ------------ | ------------------------------------------------------- | --------------------------- |
| 1. Clarify   | Answered questions                                      | User confirms understanding |
| 2. Specify   | `specs/<name>/spec.md`                                  | User approves spec          |
| 3. Plan      | `specs/<name>/plan.md` + `data-model.md` + `contracts/` | User approves plan          |
| 4. Task      | `specs/<name>/tasks.md`                                 | User approves tasks         |
| 5. Implement | Working, tested code                                    | All acceptance criteria met |

**Never skip a phase.** Each gate exists to catch misunderstandings early, when they're cheap to fix.

## Phase 1: Clarify Requirements

Before writing a single line of code or spec, ask clarifying questions.

**Goal**: Eliminate ambiguity. You cannot write a good spec for a problem you don't fully understand.

**Questions to ask** (adapt to context):

- What problem does this solve? Who is the user?
- What does success look like? How will we know it's working?
- What are the hard constraints? (performance, security, compatibility)
- What is explicitly out of scope?
- Are there existing systems this must integrate with?
- What edge cases matter most?

**Output**: A shared understanding. Document key decisions as assumptions in the spec.

## Phase 2: Write the Specification

Create `specs/<feature-name>/spec.md` using this structure:

```markdown
# Spec: <Feature Name>

## Problem Statement

One paragraph. What problem are we solving and why does it matter?

## User Stories

- As a <role>, I want to <action> so that <benefit>.

## Functional Requirements

- FR-001: <requirement>
- FR-002: <requirement>

## Non-Functional Requirements

- Performance: <target>
- Security: <requirements>
- Compatibility: <constraints>

## Acceptance Criteria

- [ ] AC-001: <verifiable criterion>
- [ ] AC-002: <verifiable criterion>

## Out of Scope

- <explicitly excluded item>

## Open Questions

- <unresolved question>
```

**Gate**: Present the spec to the user. Do not proceed until they approve it or provide feedback.

## Phase 3: Plan Architecture

Create `specs/<feature-name>/plan.md` with:

```markdown
# Plan: <Feature Name>

## Technical Context

- Language/runtime: <version>
- Key dependencies: <name@version — reason>
- Storage: <database/filesystem/etc>
- Testing: <framework>

## Architecture Overview

<Describe the high-level design. Include a diagram if helpful.>

## Project Structure

<Concrete directory layout — no "Option A / Option B">

## Data Model

<Entity definitions with fields, types, constraints>

## API Contracts

<Request/response shapes, event schemas>

## Key Technical Decisions

| Decision | Choice | Rationale |
| -------- | ------ | --------- |
```

Also create:

- `specs/<feature-name>/data-model.md` — detailed entity definitions
- `specs/<feature-name>/contracts/` — API/event schemas

**Gate**: Present the plan. Do not proceed until approved.

## Phase 4: Break Into Tasks

Create `specs/<feature-name>/tasks.md`:

```markdown
# Tasks: <Feature Name>

## Implementation Order

- [ ] T-001: Set up project scaffold and dependencies
- [ ] T-002: Implement data model / database schema
- [ ] T-003: [P] Write unit tests for core logic ← [P] = parallel-safe
- [ ] T-004: Implement core business logic
- [ ] T-005: Implement API layer
- [ ] T-006: Integration tests
- [ ] T-007: Documentation
```

Rules:

- Each task should be completable in one sitting
- Mark parallel-safe tasks with `[P]`
- Order by dependency (foundational tasks first)
- Include test tasks alongside implementation tasks

**Gate**: Present tasks. Do not proceed until approved.

## Phase 5: Implement

Execute tasks in order, following the approved plan.

### Code Quality Standards

- Write tests **before** or **alongside** implementation (TDD preferred)
- Every public function/method must have documentation
- No `any` types in TypeScript — use proper interfaces
- No disabled lint rules (`eslint-disable`, `@ts-ignore`, etc.) — fix the code instead
- Follow the principle of least surprise: code should do what it looks like it does

### Pre-Commit Checklist

Before every commit, verify:

- [ ] Full test suite passes
- [ ] Linting passes with zero errors
- [ ] Type checking passes
- [ ] Build succeeds
- [ ] The specific feature/fix was manually verified
- [ ] No debug code or `console.log` left behind

### Verification Mindset

> "It should work" ≠ "I verified it works"

Always run the actual checks. Never assume a change fixed the issue without running the test suite.

## Tracking Progress

Check off tasks in `tasks.md` as you complete them. After each task, briefly confirm what was done and what's next. This keeps the user informed and catches drift early.

## References

- See [references/constitution.md](references/constitution.md) for project quality principles
- See [references/spec-template.md](references/spec-template.md) for a full spec template
