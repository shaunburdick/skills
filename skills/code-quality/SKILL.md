---
name: code-quality
description: Enforces non-negotiable code quality standards for AI coding agents. Covers linting rules (no suppressions ever), TypeScript type safety (no `any`), and a mandatory pre-commit verification protocol. Use when any agent writes, edits, or reviews code — ensures consistent quality gates across all coding agents.
license: MIT
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Code Quality Standards

Non-negotiable rules for writing and committing code. These standards exist to keep codebases maintainable, type-safe, and free from technical debt introduced by suppressed warnings.

## Linting — Zero Tolerance for Suppressions

- ❌ **NEVER** add `eslint-disable`, `eslint-disable-next-line`, or `eslint-disable-line` comments
- ❌ **NEVER** use TypeScript suppression comments: `@ts-ignore`, `@ts-nocheck`, `@ts-expect-error`
- ❌ **NEVER** disable specific lint rules inline in code
- ✅ **ALWAYS** refactor code to comply with the linting rule's intent
- ✅ **ALWAYS** fix type errors by improving type definitions and code structure
- ✅ If a lint rule seems wrong for the project, discuss modifying the **lint config** with the user — don't suppress it in code

**When you encounter a lint error:**
1. Read and understand what the rule is trying to prevent
2. Refactor your code to comply with the rule's intent
3. If you believe the rule is genuinely incorrect for the project, ask the user about modifying the lint configuration file
4. Document why the pattern you're using is correct and safe

**Wrong vs. Right:**

```typescript
// ❌ WRONG — suppressing the rule
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function process(data: any) { ... }

// ✅ RIGHT — proper typing
interface ProcessData {
  id: string;
  value: number;
}
function process(data: ProcessData) { ... }
```

## Type Safety

- ❌ No `any` types in TypeScript — use proper interfaces, generics, or `unknown` with type guards
- ✅ Create interfaces or type aliases for all data shapes
- ✅ Use generics when the type varies but the structure is consistent
- ✅ Use `unknown` + type narrowing instead of `any` when the type is genuinely unknown

## Pre-Commit Verification Protocol

**⚠️ "It should work" ≠ "I verified it works." Always run the checks.**

Complete this checklist before **every** commit:

### 1. Run All Automated Checks
- [ ] Full test suite: `npm test` / `pytest` / `go test ./...` / equivalent
- [ ] Linting: `npm run lint` / `eslint .` / `ruff check .` / equivalent
- [ ] Type checking: `tsc --noEmit` / `mypy` / equivalent
- [ ] Build: `npm run build` / `go build` / equivalent
- [ ] Zero errors **and** zero warnings in all outputs

### 2. Verify the Specific Change
- [ ] Identify the exact issue being fixed or feature being added
- [ ] Confirm the relevant test was failing before your change (for bug fixes)
- [ ] Confirm the test now passes with your change
- [ ] Verify no new failures were introduced

### 3. Manual Verification
- [ ] Run the application locally and test the affected feature
- [ ] Test edge cases related to your change
- [ ] Verify related functionality still works
- [ ] Check logs/console for new warnings or errors

### 4. Self-Review the Diff
- [ ] Review every changed file before committing
- [ ] No debug code, `console.log`, or temporary hacks remain
- [ ] All changed files are intentional
- [ ] Code follows project conventions and style guide

### 5. Integration Check
- [ ] Changes work with the rest of the codebase
- [ ] No breaking changes to public APIs (or they're intentional and documented)
- [ ] Dependent code still functions

### Example Verification Workflow

```bash
# Run all checks — every one must pass cleanly
npm run lint        # Zero errors, zero warnings
npm run type-check  # Zero errors
npm test            # All tests green
npm run build       # Completes successfully

# Verify the specific change works
# (run the reproduction case, test manually, check edge cases)

# Only after ALL checks pass:
git add .
git commit -m "fix: describe what was fixed and how it was verified"
```

### 🚫 Do NOT commit if:
- Any test is failing (even "unrelated" ones)
- Linting has any errors or warnings
- Type checking fails
- Build fails or has warnings
- You haven't manually verified the fix works
- You're not certain what the change actually does

### ✅ Only commit when:
- ALL automated checks pass cleanly
- You've manually verified the fix works
- You've tested edge cases
- You understand exactly what changed and why
- You can explain how you verified the fix

## Documentation Standards

- ✅ Every public function and method must have a doc comment
- ✅ Complex logic must have inline comments explaining the *why*, not the *what*
- ✅ Use meaningful variable names — `userAuthToken` not `t`, `retryCount` not `n`
- ❌ No commented-out code in commits — delete it or use version control

## Implementation Checklist

Before calling an implementation task complete:

- [ ] Latest stable versions of all dependencies are used
- [ ] All public methods have doc comments
- [ ] Automated tests implemented with adequate coverage (per project constitution)
- [ ] Code passes all linting rules — **zero suppressions**
- [ ] Type safety fully enforced — **no `any`**
- [ ] All acceptance criteria from spec are met
- [ ] Security best practices followed
- [ ] Code follows architecture defined in plan
