---
round: 2
ledger:
  - key: card4-history-order-depends-on-card5
    rounds: [1]
    status: resolved
  - key: checklist-resign-arrows-missing
    rounds: [1]
    status: resolved
  - key: card19-twill-resign-arrows-caller-uncovered
    rounds: [1]
    status: resolved
  - key: twill-burler-yaml-unreadable-refusal-untested
    rounds: [1]
    status: resolved
  - key: shedbuild-doc-single-llm-stale
    rounds: [1]
    status: resolved
  - key: card17-test-file-paths-instead-of-member-glyphs
    rounds: [1]
    status: resolved
  - key: card17-single-llm-comment-sites-missed
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card5-goto-lines-in-unreadable-recipe-frame
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-card20-advisorname-last-reader-disagree
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's six findings are not re-raised in round 2, and the review's focus verdicts confirm the card 4 and card 5 ordering and the twill refusal tests.
Round 2 raised three new local findings: unreworded single-producer comment sites (F1), an undecided goto-line case in the unreadable-recipe frame (F2) and a card 8 and card 20 wording conflict over `AdvisorName` (F3).
None is gating.
