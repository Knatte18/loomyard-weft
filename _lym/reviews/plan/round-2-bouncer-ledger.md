---
round: 2
ledger:
  - key: card5-bundles-credential-record-and-login
    rounds: [1]
    status: resolved
  - key: d8-recovery-skips-stale-dropped-answer-posts
    rounds: [1]
    status: resolved
  - key: card5-botscopes-breaks-missingscope-row
    rounds: [1]
    status: resolved
  - key: nil-livehubs-fallback-unreachable
    rounds: [1]
    status: resolved
  - key: card8-holder-branches-without-row
    rounds: [1]
    status: resolved
  - key: card11-missing-setup-refusal-no-row
    rounds: [1]
    status: resolved
  - key: card12-docgo-stale-sentences-unnamed
    rounds: [1]
    status: resolved
  - key: card11-stale-reopen-wording
    rounds: [1]
    status: resolved
  - key: card6-lastslackok-wrong-card-credit
    rounds: [1]
    status: resolved
  - key: card6-newdialer-seam-shape-mismatch
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: livehubs-error-hubs-record-advice-contradiction
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card13-no-bot-token-branch-without-row
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; the only gating one was the card 5 granularity split.
Round 2 does not repeat any of them, so all nine are carried as resolved.
Round 2 raised three new findings, none BLOCKING and none design-class.
F1 is the `newDialer` seam shape mismatch between cards 2 and 6.
F2 is the contradiction between card 6's geometry-failure promise and the hubs-record advice in cards 9 and 10.
F3 is the missing row for the no-bot-token branch of `missingTokenExit`.
Each fix is a local text edit the finding already spells out.
