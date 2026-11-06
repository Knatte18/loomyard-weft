# Fixer-report, round 1

## Changes

- **F1** (LOW, fixed as proposed): `noticeDropLocked` in `internal/creelengine/receive.go` now saves `window.notices` before spending a slot and, when `postDropNoticeLocked` fails, restores it and sets `window.undisclosed` back to the count the failed notice would have reported.
  The next drop's notice then counts the earlier drops, and the slot is not spent.
  The doc comment states this.
  - Test: the row `ADropWhoseNoticeFailsToPostSpendsNoSlotAndTheNextNoticeCountsIt` in `TestDaemon_CrossHubDeliver` (`internal/creelengine/crosshub_test.go`) lets the high-water record and the drop's write pass, fails the notice write, and asserts the next drop's notice says "2 drops from that hub were journaled since the last notice".
  - Commit: `creelengine: fix F1 — a drop notice that fails to post restores the window so the next notice counts the drop`, plus a follow-up commit for the doc comment wording from the self-check.
  - Gate: `lym gate test ./internal/creelengine` passed.

## Self-check

- fix-code-correctness, LOW, doc comment said the window is left as it was: fixed, the comment now says a failed notice spends no notice slot and the next notice counts its drops.
- fix-consistency, LOW, same doc comment wording: duplicate of the above, fixed with it.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
