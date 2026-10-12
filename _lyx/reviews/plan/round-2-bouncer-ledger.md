---
round: 2
ledger:
  - key: noticelog-now-seam-missing-from-envkit-nillegal
    rounds: [1]
    status: resolved
  - key: card1-layout-go-untouched-contradiction
    rounds: [1]
    status: resolved
  - key: minute-duration-format-planned-twice
    rounds: [1]
    status: resolved
  - key: testfitoverview-per-symbol-test
    rounds: [1]
    status: resolved
  - key: card9-overview-design-missing-from-battencli-package-doc
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: formatminutes-hour-day-forms-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: validatedisplay-size-refusals-reach-no-test
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: partitionbyanchor-row-names-impossible-hidden-top-display
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's four findings are resolved: round 2's review confirms each fix holds and raises none of those keys again.
Round 2 raised four new findings.
F1 (BLOCKING) is card 9's missing battencli package-comment paragraph for `lyx batten overview`.
F2 is the untested hour and day forms of `statuspane.FormatMinutes`; it is a different defect from round 1's duplicated format.
F3 is the unreached `ValidateDisplay` size refusals and hidden exemption in card 1.
F4 is card 1's `TestPartitionByAnchor` row naming a display no strand can carry.
