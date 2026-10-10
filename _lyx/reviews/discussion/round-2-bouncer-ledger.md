---
round: 2
ledger:
  - key: form-test-payload-token
    rounds: [1]
    status: resolved
  - key: coupling-test-optionality
    rounds: [1]
    status: resolved
  - key: watcher-index-stale-after-deploy
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: runnable-groups-outside-leaf-rules
    rounds: [1]
    status: resolved
  - key: completion-subtree-unannotated-leaf
    rounds: [1]
    status: resolved
  - key: help-replacement-skipped-guards
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: pointer-filter-match-rule
    rounds: [1]
    status: resolved
  - key: json-help-listing-decision
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: record-narrative-tally-double-statement
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings, none BLOCKING.
Round 2 did not raise six of them again, and the review's judged-with-no-finding section confirms the runnable-group, completion and coupling-test points.

Three keys persist and sharpen.
The help replacement now fails the clitree walk's bare-group and unknown-subcommand checks (`help-replacement-skipped-guards`, BLOCKING).
The orch watcher resolves `lyx` from its own inherited `PATH` rather than the orch pane's binary (`watcher-index-stale-after-deploy`, BLOCKING).
An unknown word under a known group, such as `lyx help board bogus`, is unsettled (`json-help-listing-decision`, LOW).
