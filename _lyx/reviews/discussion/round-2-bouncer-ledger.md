---
round: 2
ledger:
  - key: stale-junction-sweep-removes-scratch-link
    rounds: [1]
    status: resolved
  - key: check-ignore-probe-shape-open
    rounds: [1]
    status: resolved
  - key: failed-exclude-write-unbounded-downgrade
    rounds: [1]
    status: resolved
  - key: post-rename-refused-link-breaks-bound
    rounds: [1]
    status: resolved
  - key: rewired-branch-reads-two-ways
    rounds: [1]
    status: resolved
  - key: criterion-7-contradicts-recreated-weft
    rounds: [1]
    status: resolved
  - key: dangling-link-classification-resolver
    rounds: [1]
    status: resolved
  - key: lifetime-claim-false-on-weft-recreation
    rounds: [1]
    status: resolved
  - key: criteria-miss-recording-and-unwire
    rounds: [1]
    status: resolved
  - key: rename-has-no-mutation-kind
    rounds: [1]
    status: resolved
  - key: windows-risk-handoff-no-owner
    rounds: [1]
    status: resolved
  - key: remove-unlink-blind-on-subpath-anchor
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: target-creation-and-case-1-refusal-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: case-3-moves-tracked-content
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: seed-scratch-exclude-not-retired
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: path-renamed-guard-entry-has-no-slot
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

All round-1 keys are resolved: round 2 re-raises none of them, and its focus verdicts show the stale-junction sweep, exclude probe and exclude-failure bounds settled.
Round 2 raised five new keys.
Three are MEDIUM `design`: remove's unlink of the root scratch link on a subpath anchor, unbounded outcomes for target creation and case-1 link refusal, and case 3 moving tracked content.
Two are LOW `consistency`: the unretired `seedScratchExclude` call and the stale guard-entry citation for `path_renamed`.
The three MEDIUM keys first appear in the closing round, so Go carries them into the decision record's Open risks.
