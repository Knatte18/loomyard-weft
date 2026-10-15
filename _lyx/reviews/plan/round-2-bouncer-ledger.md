---
round: 2
ledger:
  - key: card4-landingdeps-red-until-card5
    rounds: [1]
    status: resolved
  - key: card1-unreadable-file-marked-invalid
    rounds: [1]
    status: resolved
  - key: card1-skills-hatch-no-bound
    rounds: [1]
    status: resolved
  - key: card8-language-skills-key-no-bound
    rounds: [1]
    status: resolved
  - key: card1-local-override-row-unreachable
    rounds: [1]
    status: resolved
  - key: card6-unreadable-row-and-master-stop-doc-stale
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card1-loom-transient-handling-claim-false
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-row-placed-in-wrong-spec-section
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: row-triggers-omit-local-override-case
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card1-way-forward-label-missing
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-round-profile-comment-key-count-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five findings did not reappear in round 2 and are marked resolved.
Round 2 raised six consistency findings, none gating: card 6 leaves the unreadable-config row and the Master-stop doc naming two files, card 1 claims loom transient handling that does not exist, card 5 names the wrong spec section, the new rows omit the local-override trigger, card 1's way-forward clause lacks its label, and card 8 leaves a stale key count in `burlerRoundProfile`'s comment.
All are local text corrections the review's fixes already spell out.
