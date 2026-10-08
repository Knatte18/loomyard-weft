---
round: 2
ledger:
  - key: comment-fixed-column-wraps
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
  - key: redundant-package-target-unit-as-path
    rounds: [1]
    status: resolved
  - key: spec-paragraph-multi-sentence-lines
    rounds: [1]
    status: resolved
  - key: duplicate-findings-repeated-target
    rounds: [1]
    status: resolved
  - key: interface-method-func-prefix-detection
    rounds: [1]
    status: resolved
  - key: scan-directory-skips-untested
    rounds: [1]
    status: resolved
  - key: package-overview-omits-plangatepass
    rounds: [1]
    status: resolved
  - key: test-row-pins-unreachable-input
    rounds: [1]
    status: resolved
  - key: repeated-plan-gate-assertion
    rounds: [1]
    status: resolved
  - key: edittarget-doc-comment-unparseable
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 2 raised two findings.
F1 (BLOCKING) is the round 1 comment-wrap defect surviving at five edited sentences the round 1 fix did not reach, so `comment-fixed-column-wraps` stays open.
F2 (NIT) is new: the `editTarget` doc comment does not say what the type holds.

The round 2 reviewer re-checked round 1 F2 to F9 against the fixed code and found each holding, so those keys are resolved.
