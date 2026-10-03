---
name: project-health
description: "Quantify repository composition and documentation drag — comment vs code lines, test lines with test and assertion counts, spec and doc prose, and agent instruction surface. Load when reviewing a change that adds comments, docs, specs, or agent instruction; when asked about project health, bloat, doc-to-code ratio, or comment ratio; or before deciding whether process documentation is earning its keep. Ships a dependency-free script with threshold warnings and an opposing-direction ratchet."
license: MIT
metadata:
  author: shaunburdick
  version: "1.0.0"
---

# Project Health

Measures where a repository's lines actually go, so documentation and comment
bloat become **a number in a diff** rather than something discovered a year
later.

This measures composition, not quality. No number here is a grade.

## Run it

```bash
python3 scripts/project_health.py                  # the stat block
python3 scripts/project_health.py --json           # machine-readable
python3 scripts/project_health.py --explain        # metric definitions and limits
python3 scripts/project_health.py --check          # ratchet; exit 1 on violation
python3 scripts/project_health.py --update         # adopt current values as baseline
```

Standard library only. No install step.

## The stat block

One metric per line, grouped. A denser two-column table was tried first and
rejected: these get read one value at a time in review ("did `commentRatio`
move?"), and tracking a row across a column gap is work the reader should not
have to do.

```
composition
  product code lines                                   31,254
  comment lines on product code                        27,094
  comment share of product source                      0.4644
  product source files                                    351

tests
  test lines                                           79,714
  test cases                                            4,123
  assertions                                            8,034
  test lines per case                                   19.30
  assertions per case                                   1.9000

documentation
  prose in docs/ and specs/                            17,603
  prose in every doc file                              19,754
  all doc prose per code line                           0.6300
  prose outside docs/ and specs/                        0.1089

agent context
  repo-local instruction lines                          4,302
  repo-local instruction files                             15
  global skills and harness agent defs                  8,620
```

(Measured on a pnpm/TypeScript monorepo: 351 product files, 4,123 vitest cases.)

`commentRatio` is the comment's **share of product source** —
`productCommentLines / (productCommentLines + productCodeLines)` — so `0.4644`
means comments are nearly half the source. `--explain` gives the rest.

**Two doc numbers on purpose.** `prose in docs/ and specs/` counts only
gathered documentation; `prose in every doc file` counts all of it. The gap is
diagnostic — `prose outside docs/ and specs/` at `0.1089` means about a tenth of
the prose sits next to the code it describes rather than in one place, which is
harder to maintain and easier to miss.

**Agent context is split on purpose.** `repo-local instruction lines` is this
repo's own text — `AGENTS.md`, repo-local skills, repo-local agent definitions —
and is the only part the repo can act on or ratchet. `global skills and harness
agent defs` is reported for context and **never gated**, because a repository
cannot take responsibility for text installed in a user's home directory. Folding
them together made one metric read roughly 3× larger than the thing anyone is
accountable for.

## Reading the numbers

The pattern this exists to catch is **a small code change dragging a large
documentation change behind it.** A 5-line fix that requires 15+ lines of spec
and comment updates is not a documentation problem — it is a tax on small
changes, which discourages small changes and quietly batches them into riskier
ones. Watch for:

- `commentRatio` climbing over time, especially alongside stable `productCodeLines`
- `prose in every doc file` growing faster than `product code lines`
- `all doc prose per code line` above 1.0 — a doc line per code line
- `repo-local instruction lines` climbing; every line is loaded every session
- `assertions per case` falling while `test cases` holds steady — tests asserting less

Trajectory matters more than any single reading. One snapshot cannot tell you
whether a ratio is healthy for your project; a series can.

## Ratchet semantics

`--check` compares against `baseline.json`. Three directions, and the first is
the one that matters:

- **band** — must not move more than its tolerance in **either** direction.
  `commentRatio` (±0.05 absolute), `productCommentLines`, `allDocProseLines`,
  `agentContextLines`, `testLines` (±10% relative).
- **up_good** — must not shrink. `testCases`, `testAsserts`.
- **up_bad** — must not grow. `docProseToCode`.

The gate is a **conjunction**: every clause must hold.

### Why band, and not "must not grow"

A one-directional "comments must not increase" rule is satisfied by **deleting
every comment** — which also improves the comment ratio it was supposedly
policing. That is not a hypothetical; it is the first version of this script, and
the ratchet passed the deletion with exit 0 while the ratio read a perfect
`0.0000`.

A band closes it. Movement in either direction has to be a conscious baseline
change, so *reducing* the documentation burden becomes something you do on
purpose and show in a diff, rather than something you achieve by deletion. If
your project genuinely has too many comments, raise the baseline and say why —
the diff is the argument.

Raising a baseline is allowed generally; it appears as a visible diff line in the
commit. **That visibility is the mechanism.** Run `--update` in its own commit.

Use `--baseline PATH` for a per-repo baseline. The shipped `baseline.json`
belongs to the repo this skill lives in; pointing `--update` at another repo
without `--baseline` would overwrite it with the wrong numbers.

## Thresholds

`thresholds.json` ships starting points derived from one reported incident and one
calibration pass against a real monorepo — not from a survey of healthy projects. **Calibrate against your own history before
trusting them** — run for a few weeks, read the trend, then set limits where
your project actually sits. Delete any key to disable that check. Every edit is
a diff, which is the same mechanism as the ratchet.

## Accuracy limits

Read these before quoting a number as fact:

- **Comment detection is a state machine** over line and block tokens. A comment
  token at the start of a heredoc or string continuation counts as a comment.
- **Trailing comments are counted as code.** `foo(); // note` is a code line. The
  comment figures therefore **undercount**, which is the safe direction — the
  tool does not cry wolf.
- **Test discovery is by path and name convention.** Unconventional naming
  under-reports test metrics rather than misclassifying product code as tests.
- **Test-case and assertion counts are regex-based** and will differ from a real
  test runner. They are a guard against silent deletion, not a coverage report.
- **`spec`/`specs` are not treated as test directories**, because spec-kit owns
  `specs/` for feature documents. RSpec is matched by filename instead. Treating
  `specs/` as tests silently swallowed every spec in an early version.
- **Extensionless source is matched by name** (`.zshrc`, `Makefile`, `Dockerfile`,
  …). Without that, a dotfiles or infrastructure repo reports almost no product
  code, because those files have no suffix. `.gitignore` and friends are
  excluded as repository metadata.

## Deliberately not measured

Comment *quality*, documentation *accuracy*, coverage adequacy, or any ratio
presented as a score. A metric that reads like a grade gets optimised instead of
understood. If a number here starts moving because someone is chasing it, the
metric has stopped working — delete the threshold rather than satisfy it.