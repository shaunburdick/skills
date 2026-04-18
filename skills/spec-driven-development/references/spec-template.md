# Spec Template

Copy this template when creating a new spec. Replace all `<placeholder>` values.

---

```markdown
# Spec: <Feature Name>

> Status: Draft | In Review | Approved | Implemented
> Created: <date>
> Author: <name>

## Problem Statement

<One to three paragraphs. What problem are we solving? Who experiences it? Why does it matter now?>

## User Stories

- As a **<role>**, I want to **<action>** so that **<benefit>**.
- As a **<role>**, I want to **<action>** so that **<benefit>**.

## Functional Requirements

- **FR-001**: <Specific, testable requirement>
- **FR-002**: <Specific, testable requirement>
- **FR-003**: <Specific, testable requirement>

## Non-Functional Requirements

- **Performance**: <Latency targets, throughput, resource limits>
- **Security**: <Auth requirements, data sensitivity, threat model>
- **Reliability**: <Uptime, error handling, retry behavior>
- **Compatibility**: <Browser support, OS, runtime versions>
- **Observability**: <Logging, metrics, tracing requirements>

## Acceptance Criteria

- [ ] **AC-001**: <Verifiable, binary criterion — either it passes or it doesn't>
- [ ] **AC-002**: <Verifiable, binary criterion>
- [ ] **AC-003**: <Verifiable, binary criterion>

## Out of Scope

The following are explicitly **not** part of this feature:

- <Item 1>
- <Item 2>

## Assumptions

The following assumptions were made during spec creation:

- <Assumption 1>
- <Assumption 2>

## Open Questions

- [ ] <Unresolved question that needs an answer before implementation>
- [ ] <Another open question>

## Edge Cases

- <Edge case 1 and how it should be handled>
- <Edge case 2 and how it should be handled>
```
