---
round: 2
ledger:
  - key: committed-tokencount-binary
    rounds: [1]
    status: resolved
  - key: reference-rule-test-missing-parallel
    rounds: [1]
    status: resolved
  - key: calibrate-peak-position-pricing-untested
    rounds: [1]
    status: resolved
  - key: batcher-cost-before-shadowed
    rounds: [1]
    status: resolved
  - key: reference-rule-test-redundant-row
    rounds: [1]
    status: resolved
  - key: readonly-command-quoted-name-blank
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: recover-batch-refusal-omits-readonly-route
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: accept-audit-usage-line-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: tokencount-added-behaviors-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised five findings, none seen again in round 2, so all five are resolved.
Round 2 raised four new ones.

- `readonly-command-quoted-name-blank` (F1): `IsReadOnlyCommand` in `internal/fabricengine/refscanner.go` decides the command word from quote-blanked text, so a quoted non-reader before a reader name passes.
- `recover-batch-refusal-omits-readonly-route` (F2): the refusal in `internal/websterengine/recoverbatch.go` names only the reset route, while the spec row also names the committed read-only route.
- `accept-audit-usage-line-stale` (F3): the usage line in `internal/webstercli/cli.go` names one route only.
- `tokencount-added-behaviors-untested` (F4): the growth-ratio `n/a` case, a malformed review file and an unreadable plan in `tools/tokencount` have no test rows.
