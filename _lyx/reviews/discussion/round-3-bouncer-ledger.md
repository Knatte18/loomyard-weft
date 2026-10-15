---
round: 3
ledger:
  - key: work-dir-clear-unbounded
    rounds: [1]
    status: resolved
  - key: lyx-literals-pattern-departure
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: work-path-compare-canonical
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, round 2 two, round 3 three.
Round 3 has no BLOCKING and no MEDIUM finding.

`lyx-literals-pattern-departure` fell from MEDIUM to LOW consistency: the Constraint's claims are true, and what remains is the amended `PATTERN.md` line, which should scope the invariant to Go paths instead of adding a shell exception.

`out-path-overwrite-unbounded` persists at LOW: D1 needs to state the two paths the `--out` write reaches, the `mktemp` choice and the bound on replacement.

`work-path-compare-canonical` is new in round 3: D1's `--work` refusal checks need to say they compare canonical paths, with the marker rule as the backstop.
