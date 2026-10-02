---
round: 1
ledger:
  - key: card3-restamp-window-bound
    rounds: [1]
    status: resolved
  - key: card9-refusal-row-reaching-test
    rounds: [1]
    status: resolved
  - key: card9-attempt-spent-before-wiring-guard
    rounds: [1]
    status: resolved
  - key: card7-plan-dir-outside-repo-bound
    rounds: [1]
    status: resolved
  - key: card5-prompt-lyx-write-contradiction
    rounds: [1]
    status: resolved
  - key: card9-interrupted-attempt-bypasses-check
    rounds: [1]
    status: resolved
  - key: card7-dirty-worktree-accepted
    rounds: [1]
    status: resolved
  - key: card1-forthcoming-selected-from-current
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings, and the fixer resolved all eight in the cards before this judgment.
Card 3's bound now names the check-to-restamp window in which an edit is adopted along with a sanctioned rewrite.
Card 9 gained a wiring guard ahead of the attempt record, an in-flight-attempt check ahead of the `OK` early return, a plan-fingerprint compare around the strand, and a reaching test for its refusal row's way forward.
Card 7 checks the `_lyx` entry itself and refuses a dirty worktree, and states that commits cannot reach a plan directory outside the repository.
Card 5's prompt names the report and friction note as the only allowed `_lyx` writes.
Card 1 selects forthcoming cards from the reloaded plan, never from the pending-only `current`.
No finding stays open.
