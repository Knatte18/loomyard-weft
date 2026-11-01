---
round: 3
ledger:
  - key: cancel-end-only-reads-done
    rounds: [1, 2]
    status: resolved
  - key: d2-warning-way-forward-not-fabric-pull
    rounds: [1, 2]
    status: resolved
  - key: fix-task-gate-label-unknown-vocabulary
    rounds: [1]
    status: resolved
  - key: adopt-resume-texts-unlisted
    rounds: [1]
    status: resolved
  - key: gateslot-doc-table-subjectless-check
    rounds: [1]
    status: resolved
  - key: stale-file-headers-check-adopt-deps
    rounds: [1]
    status: resolved
  - key: settle-ownership-drop-race-only
    rounds: [1]
    status: resolved
  - key: failedend-run-status-before-record
    rounds: [1]
    status: resolved
  - key: live-pair-row-hits-takeover-stop
    rounds: [1]
    status: resolved
  - key: card5-missing-resign-arrows
    rounds: [1]
    status: resolved
  - key: filefronttask-refusal-unnamed
    rounds: [1]
    status: resolved
  - key: failed-tier-notice-keeps-prime-way-forward
    rounds: [2]
    status: resolved
  - key: overview-batten-paragraph-adopt-first-pass
    rounds: [2]
    status: resolved
  - key: testremove-settle-called-inside-held-lock
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reconcilewarnings-records-branch-worktree-no-row
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, round 2 four, round 3 two.
Every earlier key is resolved: round 3 does not raise them, and its rejected section checks the cancel, notice and overview-paragraph areas and finds them sound.
Round 3 raises two new design findings.
F1 is a TestRemove row in card 6 that would hang on a blocking flock; the fix rewords the row to run `Settle` in a goroutine.
F2 is a missing TestAdopt row in card 11 for the records-branch worktree choice.
Both fixes are local and the review text fixes their result.
