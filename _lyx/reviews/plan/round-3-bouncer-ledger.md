---
round: 3
ledger:
  - key: card4-card-packs-column-contradicts-test-row
    rounds: [1]
    status: resolved
  - key: card4-card-arm-spans-biased-window
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card2-standalone-runs-lose-pack-default-on
    rounds: [1]
    status: resolved
  - key: card2-empty-card-test-row-conflates-cases
    rounds: [1]
    status: resolved
  - key: card2-stale-packcard-doc-comment
    rounds: [1]
    status: resolved
  - key: card4-net-columns-omit-card-pack-calls
    rounds: [2]
    status: resolved
  - key: card4-pack-check-grades-later-card-spans-against-start
    rounds: [2]
    status: resolved
  - key: card3-fragment-banners-name-only-pack-file
    rounds: [2]
    status: resolved
  - key: card2-wrong-mechanism-for-non-hub-refusal
    rounds: [2]
    status: resolved
  - key: card1-operator-comments-omit-hub-only-limit
    rounds: [2]
    status: resolved
  - key: card4-mainforks-count-sentence-and-fork-card-timestamp
    rounds: [2]
    status: resolved
  - key: card4-disagreement-rule-zero-call-case
    rounds: [2]
    status: resolved
  - key: card4-later-card-pack-files-missing-from-missed-files
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card4-first-call-only-pack-check-not-in-report
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 raised the card-arm span window and a run of local text corrections on cards 1 to 4; round 3 does not raise those again, bar the window thread.
Round 3 finds the grading fix for later card-pack calls incomplete: dropping their rows from `pack-entries.tsv` also removes their files from the missed-files check (BLOCKING, fix names a third row kind that feeds only the covered-files set).
The window thread continues as F2: the note sits under the Pack use table only, not under the Arm comparison and Per-fork tables (scope, MEDIUM).
F3 asks the printed Pack correctness section to say the card arm's check covers its first call only (LOW).
