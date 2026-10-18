---
round: 1
focus:
  - "Cards 7-9 (`status-push-spawn-entry`, `push-lock-pending-flags`, `pair-teardown-lock`): do the try-lock, per-side pending flags and per-slug teardown lock leave any interleaving where a push is lost, a pending request is dropped, or Remove races a push child?"
  - "Cards 2-6 (local branch mark): does every push primitive and fabric path (push, remove, cleanup, adopt on a plain `add`) honor the mark, and does a plain add clear it only for a marked warp branch while an unmarked existing branch still refuses with `ErrBranchExists`?"
  - "Cards 14-15 (`Command` producer and engine): is each removed switched row and engine fully gone with no remaining caller (the cards' `Delete` groups' `assert-no-callers`), and is the command read from the template and board default layer only, per `PATTERN-command-row`?"
  - "The `PATTERN.md` and `pattern/` diff: does it land each declared entry (`PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup`, `PATTERN-command-row`) with the declared text?"
  - "Card 10 and card 7: does any status seam still push inline instead of through `SpawnDetachedPush`, and does a test anywhere fork a real push child?"
  - "Cards 11-13 (`verifyrun` quoting, `lyx ide seed/open`, `lyx reed window`): does the new CLI behavior match each card's Intent, and are the module `doc.go` and `docs/overview.md` updated in the same commits as the CLAUDE.md docs rule requires?"
---

Seed focus from the plan's card index and shared decisions; the base diff review still applies in full.
