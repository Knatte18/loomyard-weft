---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1: `internal/desktoplauncher/doc.go` now says that a `.desktop` launcher shows its `Name=` key,
  and that a `.cmd` launcher shows its file name and carries the title on its `rem` line.
  `_lym/landing/summary.md` says the same of the `.cmd` file (not committed, as an `_lym` path).
  Commit `7e91329e8`.
  The gate found that this edit wrapped a `doc.go` sentence mid-clause;
  commit `7e70602cb` rewrites it as one sentence per line.
- F2: the `content` doc comment in `internal/desktoplauncher/desktoplauncher.go` names the `Name=` key and the `rem` line.
  Commit `af39352d8`.
- Self-check: the lead sentences of `_lym/landing/summary.md` now call `lym: <shortname>` the launcher's title,
  not its name on every form (not committed).

`lym gate test ./internal/desktoplauncher` passed on the tree with both fixes and the line repair (tier0, 33 targets passed, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the summary's lead sentence stated the name for every form: fixed with our own change (the fork proposed none).

## Disputed

## Deferred

## Rulings
