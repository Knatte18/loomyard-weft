---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

- F1, commit 6c1726aca: `contracts/recipes/darn-recipe.yaml` comment now says a hub whose scoper is not go passes the red-first gate, and only a diff that adds no test holds the run.
- F2, commit 6b9841624: `docs/overview.md` Darn-Write paragraph drops the unjudged-check hold and says a non-go hub passes the gate.
- F3, commit 0ea99cc9a: a row in `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` runs a command that cats a runner-crash line and exits 5. It expects a terminal failure with "The red-first check's test runner crashed".
- F4: `_lym/landing/summary.md` limits the no-test hold to a go hub and widens the blast radius to the verb message and the darn writer stencil.
  No commit, since the file is under `_lym`.

`lym gate test --tier 1 ./internal/loomshed` passed.

## Self-check

Every fork returned a report.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
