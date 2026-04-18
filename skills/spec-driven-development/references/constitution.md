# Project Constitution

These are the standing quality principles that apply to all implementations. Reference these when making technical decisions.

## Core Principles

### 1. Correctness Over Speed
Working code that's slightly slower to write beats broken code that was fast to produce. Never skip the verification checklist.

### 2. Explicit Over Implicit
Code should say what it does. Avoid clever tricks that require deep knowledge to understand. Future-you (and your teammates) will thank present-you.

### 3. Tests Are Not Optional
Automated tests are the minimum bar, not the goal. Write tests that would catch real bugs, not tests that just execute code paths.

### 4. No Lint Suppression
If a linter complains, fix the code. Never add `eslint-disable`, `@ts-ignore`, `@ts-nocheck`, or similar suppressions. If you believe the rule is wrong for the project, discuss changing the lint config — don't work around it silently.

### 5. Minimal Dependencies
Every dependency is a liability. Before adding a package, ask: can this be done in 10 lines of code? If yes, write the 10 lines.

### 6. Type Safety
No `any` types in TypeScript without explicit justification in a comment. Use proper interfaces, generics, and discriminated unions.

### 7. Documentation for Public APIs
Every public function, method, and module must have a doc comment explaining what it does, its parameters, return value, and any notable side effects.

## Code Quality Standards

| Standard | Requirement |
|----------|-------------|
| Test coverage | ≥ 80% line coverage for new code |
| Lint | Zero errors, zero warnings |
| Type checking | Strict mode, zero errors |
| Build | Clean build with no warnings |
| Documentation | All public APIs documented |

## Commit Standards

- Commits on feature branches only (never directly to `main`/`master`/`develop`)
- Conventional commit messages: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`
- Every commit must pass the full pre-commit checklist
- Small, focused commits — one logical change per commit

## Security Defaults

- Never commit secrets, credentials, or API keys
- Validate and sanitize all external input
- Use environment variables for configuration
- Apply principle of least privilege to all permissions
