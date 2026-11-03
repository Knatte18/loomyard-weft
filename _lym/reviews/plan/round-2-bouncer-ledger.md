---
round: 2
ledger:
  - key: boardcli-storeverbs-wrong-symbol
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: pastapproval-error-branches-unreachable
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card2-post-save-removal-error
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card2-header-edit-untargeted
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card7-store-failure-departs-from-d6
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: copyrecordtoboard-notfound-unreachable
    rounds: [1]
    status: resolved
  - key: rename-adopts-orphan-record-file
    rounds: [1]
    status: resolved
  - key: refused-rename-row-lacks-validateslug-seam
    rounds: [1]
    status: resolved
  - key: boardcli-doc-payload-rule-omits-discussed
    rounds: [1]
    status: resolved
  - key: card7-leaves-two-comments-untrue
    rounds: [1]
    status: resolved
  - key: card3-misstates-config-board-check
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings, none BLOCKING.
Round 2 resolved six of them: the card 8 not-found branch, the orphan adoption on rename, the refused-rename test seam, the boardcli doc payload rule, the two untrue card 7 comments, and the card 3 config check.

Round 2 raised five findings.
F1 is new and BLOCKING: cards 1 and 5 target `internal/boardcli#Command` where `upsert` and `get` live in `storeVerbs`, and the `upsert` `Use` token is never updated.
Three older keys persist in changed form:
the unreachable `pastApproval` and `approvalRow` error branches now sit in card 9 (LOW, design);
the card 2 orphan removal and its second error is now a decision-class gap against D4 (MEDIUM);
the card 7 store-failure departure from D6 narrowed to the accepted-round resume (LOW, decision).
F4 is a new LOW consistency finding: card 2 names a file header it does not target.
