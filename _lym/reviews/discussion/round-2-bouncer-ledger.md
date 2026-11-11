---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resume-check-runs-before-prepare
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: search-session-ids-no-source-into-engine
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: config-dir-leaks-into-tmux-server
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: counter-bursts-readded-or-reweighted
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reused-name-usage-file-blocks-new-account
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d1-provenance-masks-tracked-value
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: logged-out-directory-keeps-worktrees
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-resume-criteria-omit-listed-accounts
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: readers-on-invalid-list
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: status-line-verb-finds-no-hub
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: seed-undefined-with-no-counts
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's twelve keys were not raised again by round 2's fresh review, so all but one are resolved.
The exception is `d6-known-directories-lose-delisted-and-landed`: F1 shows D6's set still cannot reach the readers, so the key is reopened at BLOCKING.

Round 2 adds twelve new keys.
The five MEDIUM design ones are the `lym report` reader (F2), the resume check ordering (F3), the session id sources (F4), the tmux server environment leak (F5) and the counter burst (F6).
The rest are LOW or NIT; F13 folds into F6's rewrite.
