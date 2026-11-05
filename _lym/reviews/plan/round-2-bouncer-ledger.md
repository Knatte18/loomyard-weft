---
round: 2
ledger:
  - key: beam-pr-mode-reads-github-rules-unconditionally
    rounds: [1]
    status: resolved
  - key: second-retired-row-mechanism-unreconciled
    rounds: [1]
    status: resolved
  - key: old-flow-text-survives-every-card
    rounds: [1]
    status: resolved
  - key: empty-beam-required-parent-no-way-to-ready
    rounds: [1]
    status: resolved
  - key: judge-awaiting-run-failed-halt-unstated
    rounds: [1]
    status: resolved
  - key: accept-removal-order-crash-window
    rounds: [1]
    status: resolved
  - key: await-test-rows-unreachable-pr-review-hold
    rounds: [1]
    status: resolved
  - key: required-checks-check-runs-only
    rounds: [1]
    status: resolved
  - key: await-entry-misses-halted-recheck
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ready-refusal-quotes-resume-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: ready-refusal-failed-github-read-unnamed
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: blocked-cause-names-satisfied-checks
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: judge-awaiting-run-done-branch-unreachable
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-doc-comments-named-by-no-card
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

All eight round-1 keys are resolved: the round-2 review re-raises none of them and records its focus items on those areas as settled.
Round 2 raised six new findings, F1 to F6, in the order of the keys above.
F1 is a gap in Beam-Await's entry detection after a halted resume.
F2 is a refusal quoting a way forward that does nothing at Beam-Await.
F3 to F6 are narrower: an unnamed failed-GitHub-read refusal, a blocked cause that lists satisfied checks, an unreachable done branch in `judgeAwaitingRun`, and two stale doc comments.
