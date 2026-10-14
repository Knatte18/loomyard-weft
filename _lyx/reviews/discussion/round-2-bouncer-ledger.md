---
round: 2
ledger:
  - key: d2-bound-claims-skips-none
    rounds: [1]
    status: resolved
  - key: d2-guard-dir-drop-too-wide
    rounds: [1]
    status: resolved
  - key: d1-bound-split-not-produced-by-covers
    rounds: [1]
    status: resolved
  - key: d1-package-level-ambiguous
    rounds: [1]
    status: resolved
  - key: d3-scan-error-persists
    rounds: [1]
    status: resolved
  - key: d4-fork-may-loosen-guard
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d2-walk-differs-from-go-list
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: d2-d3-rationale-rejected-alternatives
    rounds: [1]
    status: resolved
  - key: support-log-q6-q7-order
    rounds: [1]
    status: resolved
  - key: d3-walk-read-error-no-refusal-spec-row
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: d2-drop-parenthetical-all-files
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised nine findings; round 2 clears all but two of them and adds two new keys plus one BLOCKING.
The new BLOCKING is D3's walk read-error refusal, which lands without its `refusal-spec.md` row and without `PATTERN-refusal-way-forward` in Constraints.
D4's visibility bound still cites a per-batch review that does not exist, and D2's walk skip list still differs from `go list ./...`.
D2's drop parenthetical says all files where the clause says Go files.
All four open fixes are exact text replacements the review already supplies.
