---
round: 2
ledger:
  - key: work-dir-clear-unbounded
    rounds: [1]
    status: resolved
  - key: lyx-literals-pattern-departure
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: trace-pack-layout-arm-unbounded
    rounds: [1]
    status: resolved
  - key: batch-resolution-cross-run
    rounds: [1]
    status: resolved
  - key: runs-name-match-and-charset
    rounds: [1]
    status: resolved
  - key: code-required-under-plan
    rounds: [1]
    status: resolved
  - key: row-sort-tie
    rounds: [1]
    status: resolved
  - key: out-path-overwrite-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised seven findings; round 2 raised two.
Six round 1 keys were not raised again and the round 2 review's focus verdicts find no defect in their areas, so they are resolved.

`lyx-literals-pattern-departure` is reopened at MEDIUM: the planned PATTERN amendment now exists, but it calls the script's head block the one non-Go site spelling `_lyx` or `.lyx`, which tracked recipes, template YAML, specs and stencils already contradict.

`out-path-overwrite-unbounded` is new in round 2: D1 bounds `--work` clearing but not what `--out`'s move-into-place can overwrite, and an existing directory at `--out` needs a refusal.
