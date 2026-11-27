---
round: 2
ledger:
    - key: upkeep-call-passes-running-no-marked-tests
      rounds:
        - 1
      status: resolved
    - key: config-keys-named-in-short-form
      rounds:
        - 1
      status: resolved
    - key: marked-scope-runs-tiers-whole-under-no-carry
      rounds:
        - 1
      status: resolved
    - key: non-go-hub-upkeep-call-always-refused
      rounds:
        - 1
      status: resolved
    - key: cmd-lym-unmarked-repo-scans-unplaced
      rounds:
        - 1
      status: resolved
    - key: merge-danger-door-and-blast-radius-understated
      rounds:
        - 1
      status: resolved
    - key: guard-moves-and-held-carry-untested
      rounds:
        - 1
      status: resolved
    - key: added-gate-branches-untested
      rounds:
        - 1
      status: resolved
    - key: guard-step-drops-held-under-filter-flag
      rounds:
        - 1
      status: resolved
    - key: only-test-scan-go-list-unbounded
      rounds:
        - 1
      status: resolved
    - key: dobby-doc-misplaces-upkeep-and-timeout-lines
      rounds:
        - 1
      status: resolved
    - key: csharp-fixture-compile-guard-unplaced
      rounds:
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
---
Round 1 opened the marked-scope upkeep key's defects, missing tests for the held guards, config-key naming,
unplaced scans in `cmd/lym`, the merge-danger call and smaller code and doc points.
Round 2 answers each of them as resolved.

Round 2 opens one key.
`TestCSharpFixtures_CompileWithDotnet` is a watched whole-repository guard that builds real .NET projects and had no place,
against the declared `PATTERN-test-speed` text and the board entry's ask to place every whole-repository guard of `cmd/lym`.
