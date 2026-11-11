---
round: 3
ledger:
  - key: fresh-launch-picks-before-pre-change-resume
    rounds: [1]
    status: resolved
  - key: empty-accounts-list-still-picks
    rounds: [1]
    status: resolved
  - key: d2-resume-search-narrower-than-d6
    rounds: [1]
    status: resolved
  - key: resume-record-write-no-lock
    rounds: [1]
    status: resolved
  - key: status-line-verb-unbounded
    rounds: [1]
    status: resolved
  - key: opened-sessions-uncovered
    rounds: [1]
    status: resolved
  - key: reed-relaunch-stale-directory
    rounds: [1]
    status: resolved
  - key: d6-known-directories-lose-delisted-and-landed
    rounds: [1, 2]
    status: resolved
  - key: d3-step4-ranking-undefined
    rounds: [1]
    status: resolved
  - key: record-account-name-drift
    rounds: [1]
    status: resolved
  - key: macos-candidates-unbounded-windows-unsourced
    rounds: [1]
    status: resolved
  - key: open-risk-already-settled
    rounds: [1]
    status: resolved
  - key: lym-report-reads-no-transcripts
    rounds: [2]
    status: resolved
  - key: resume-check-runs-before-prepare
    rounds: [2]
    status: resolved
  - key: search-session-ids-no-source-into-engine
    rounds: [2]
    status: resolved
  - key: config-dir-leaks-into-tmux-server
    rounds: [2]
    status: resolved
  - key: counter-bursts-readded-or-reweighted
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reused-name-usage-file-blocks-new-account
    rounds: [2]
    status: resolved
  - key: d1-provenance-masks-tracked-value
    rounds: [2]
    status: resolved
  - key: logged-out-directory-keeps-worktrees
    rounds: [2]
    status: resolved
  - key: d2-resume-criteria-omit-listed-accounts
    rounds: [2]
    status: resolved
  - key: readers-on-invalid-list
    rounds: [2]
    status: resolved
  - key: status-line-verb-finds-no-hub
    rounds: [2]
    status: resolved
  - key: seed-undefined-with-no-counts
    rounds: [2]
    status: resolved
  - key: fresh-launch-session-sources-incomplete
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: resume-live-holder-reads-default-registry
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-liveness-no-combining-rule
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: gone-recorded-directory-empty-list
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: counter-criterion-not-true-of-counter
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 and round 2 keys other than the counter one were not raised again by round 3's fresh review, so they are resolved.
`d6-known-directories-lose-delisted-and-landed` is resolved too: round 3 accepts D6's route and leaves only the combining-rule gap, filed as `d6-liveness-no-combining-rule`.

`counter-bursts-readded-or-reweighted` is open for the second round.
The one-way raise left a re-added account above the others' lowest, and a lowered weight, passed over until the others catch up (F5).
Its fix is a set rule with the first-pick case defined.
`counter-criterion-not-true-of-counter` (F6) follows from the same rule and is reworded in terms of count over weight.

Round 3 adds four more keys.
Two are BLOCKING: the fresh-launch search misses session-id owners beyond the shuttle records, reed strands and orch state (F1), and the resume check's live-holder guard names no registry directory (F2).
The others are the D6 liveness combining rule (F3) and the gone recorded directory under an empty list (F4).
