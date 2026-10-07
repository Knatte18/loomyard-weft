---
round: 3
ledger:
  - key: fixed-column-comment-breaks
    rounds: [1, 2]
    status: resolved
  - key: serial-tests-without-global-state-note
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: lint-directive-exempts-whole-block
    rounds: [1, 2]
    status: resolved
  - key: lint-missing-yet-so-conjunctions
    rounds: [1]
    status: resolved
  - key: derive-error-always-writer-finding
    rounds: [1]
    status: resolved
  - key: verify-gate-refusal-row-incomplete
    rounds: [1]
    status: resolved
  - key: cmd-lyx-budget-not-pinned
    rounds: [1]
    status: resolved
  - key: nil-index-error-untested
    rounds: [1]
    status: resolved
  - key: per-helper-test-covered-by-guard
    rounds: [1, 2]
    status: resolved
  - key: marker-error-names-no-line
    rounds: [2]
    status: resolved
  - key: guard-scan-parse-error-row-missing
    rounds: [3]
    status: open
    class: scope
    severity: LOW
  - key: lint-code-paragraph-row-missing
    rounds: [3]
    status: open
    class: scope
    severity: LOW
---

Round 3 raised three LOW findings, F1 to F3.
F1 maps to `serial-tests-without-global-state-note`: table-driven subtests in three test files lack `t.Parallel`, now `consistency` LOW.
F2 is new: `guard-scan-parse-error-row-missing`, a coverage gap for a test file that does not parse.
F3 is new: `lint-code-paragraph-row-missing`, a coverage gap for a wrapped paragraph beside an indented code paragraph.
The other keys from earlier rounds have no round-3 finding and are resolved.
