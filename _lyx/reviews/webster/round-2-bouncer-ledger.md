---
round: 2
ledger:
  - key: fixed-column-comment-breaks
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: serial-tests-without-global-state-note
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: lint-directive-exempts-whole-block
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    status: open
    class: design
    severity: BLOCKING
  - key: marker-error-names-no-line
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 2 raised eight findings, F1 to F8.
F1 to F4 map to `fixed-column-comment-breaks`, a new instance each in a doc comment the fixes or the diff added; the lint missed F1 through F8's block exemption.
F5 maps to `serial-tests-without-global-state-note`, now on a new test, and F6 to `per-helper-test-covered-by-guard`, now on a `commentlint` test.
F8 maps to `lint-directive-exempts-whole-block`: the lint still exempts a whole block, now through a heading, list or code line.
F7 is new: `marker-error-names-no-line`.
The review's settled focus questions close the other round-1 keys.
