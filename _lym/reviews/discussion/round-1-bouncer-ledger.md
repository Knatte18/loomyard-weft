---
round: 1
ledger:
  - key: d3-trailer-way-forward-reuses-uncaused-ruling
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-sendback-file-archived-by-new-generation
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-sendback-into-disabled-perch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-full-generation-review-input-not-existing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-parent-review-asked-flag-not-cleared
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-reentry-session-resumes-recorded-id
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-verb-skip-or-shaping-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-trailer-cause-unverified-no-bound-named
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-dirty-tree-refusal-only-at-verb
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-go-appended-trailer-no-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumption-duplicates-d4
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings, F1 to F11 in the order above.
The three BLOCKING ones are the D3 trailer way forward that reuses the stored uncaused ruling, the D4 send-back file stored in the archived Code-Review run directory, and the D4 send-back into a perch a zero Code-Review budget turns off.
The MEDIUM ones are D4's unnamed full-generation mechanism, the unreset `parent_review_asked` flag, the resumed session id, the unbounded verb path, and D2's unverified trailer cause.
There is no earlier ledger, so nothing carries forward.
