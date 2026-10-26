# Support log — webster-stop-asked-once

## Interview

A darn run holds no interview: the board entry, its operator decisions of 2026-10-26 and a reading of the code settle the scope.

## Rejected alternatives

- Keep Merriam's direct question and have the driver suppress its own stop question for Webster: rejected, since the direct channel bypasses the arbiter the board entry puts first.
- Send every tackler hand-back with a question to the arbiter, for every row and recipe: rejected as wider than the entry; it changes batten's and darn's stop chains too.
- Dedupe the driver's question by its text against earlier hand-backs: rejected, since the answer reaching Merriam and the arbiter removes the recurrence, and a text match misfires on a new stop that words itself the same way.
- Render the decisions only into the live notice (already done by #849's fix): rejected as insufficient, since a re-stepped Webster starts a new Merriam that never saw the notice.

## Review rounds

_No rounds yet._

## Question ledger

- Which channel survives, Merriam's message or the driver's stop question? Auto-pick: the driver's, since it runs the arbiter first.
- Does the arbiter-first row skip the tackler or follow its hand-back? Auto-pick: skip it for marked rows; the tackler's hand-back sends its question straight to the parent.
- Which rows are marked? Auto-pick: the Webster row only, wired by the loom driver.
- Do the recovery and cold verify-fixer prompts get the decisions too? Auto-pick: no; the entry names Merriam, and Merriam hands decisions on through its forks.
