---
round: 2
ledger:
  - key: refusal-rejects-valid-unkeyed-addition
    rounds: [1]
    status: resolved
  - key: sealed-expectation-stops-parsing
    rounds: [1]
    status: resolved
  - key: decision-exemption-passes-unwritten-edits
    rounds: [1]
    status: resolved
  - key: decision-add-forms-still-halt-judge
    rounds: [1]
    status: resolved
  - key: drift-deps-decision-record-untested
    rounds: [1]
    status: resolved
  - key: exemption-fail-closed-rows-missing
    rounds: [1]
    status: resolved
  - key: driftengine-doc-describes-loomcli-refusal
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: drift-deps-test-missing-keep-marker
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised six findings; round 2's review answers all six as resolved.
Round 2 raised two new BLOCKING consistency findings.
F7 asks to move the loomcli verb's refusal out of driftengine's package doc.
F8 asks for the `//testtiming:keep` marker on `TestDriftDeps_CarryTheDecisionRecord`.
