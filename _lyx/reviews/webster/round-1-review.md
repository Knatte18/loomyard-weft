---
verdict: APPROVED
---

Range: no rework round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main` in `_lyx/shed/creel-stage1-fixes/status.json`), the fourteen card commits.

I read the diff for every card against its Intent and found no defect.
`lyx gate test ./...` and `lyx gate test --tags integration ./...` both ran green.

What I checked, per focus item:

- **Cards 2, 4, 5.**
  `PlainBoxCapture` tracks faint through SGR `2`, `0`, `22` and the empty `ESC[m`, and consumes the parameters after `38`, `48` and `58` as a colour.
  It blanks only a box whose non-blank cells are all faint and whose text is not the paste placeholder.
  `RestoredPromptClearSequence` counts the first to the last text row, blank rows included, and returns `C-u`, then `BSpace`, `C-u` for each further line.
  `InterruptRestoring` and `clearRestoredPrompt` follow the card's order: before-reading, guarded key only, two settles on an empty read, guard reading with an activity query, a single play, a post-clear read, and `ErrInterruptClearFailed` after the key.
- **Cards 3, 8.**
  `boxCapture` is the one routing site, used by `SessionIdle`, `busyReading` and `readInputBox`.
  The pane-only fallback fills `Draft`.
  `interruptTarget` treats `ErrClearFailed` as sent.
  `worker.probe` takes the pane lock before any `Delivery.Idle`, send or reminder, and the interrupt holds it around `Delivery.Interrupt` and then wakes the worker.
- **Cards 6, 7.**
  `noteStall` journals only a changed non-empty stall, and an empty one or an error clears the entry.
  Typing an inject or a reminder clears it.
  The waiters get a one-slot replacing note, and a new waiter is handed the standing note.
  `awaitOutcome` writes progress responses and a final answer carrying `Stall` and `StallAt`.
  `inject --wait` prints progress on stderr only and carries the last note's fields on a timeout.
- **Cards 9, 10.**
  `ageInjects` charges from the later of the previous probe and the entry's `Posted`, clamped at zero.
  `read` acknowledges only after `writeOk` returned nil, on both paths.
  `messageFields` adds the `ended_unread_*` fields.
- **Cards 11 to 13.**
  `Unread` and `Step` reach `log`, and `id` is left out when 0.
  The help texts carry the privilege rule, `Release` ends a hold on a removed strand and refuses an unknown name, and the sender rides the waiting responses.
  `daemonLogExcerpt` opens for append without creating, then quotes pid-tagged lines and the unprefixed lines after its own header.
  `runDaemon` writes the start header, installs `pidLineWriter` and sets the stderr threshold to Warn, and logs a failed start at Warn.
- **Test economy.**
  The new top-level tests are `TestCapturePaneArgv`, `TestPlainBoxCapture` and `TestRunner_InterruptRestoring`.
  The first and third are the ones the cards name, and the second is the card-named table for the answer's own shape.
  The `test-budget.yaml` rows rose by exactly those three.
  The creelcli, creelengine and hubgeom additions are rows in existing tests.
  I found no duplicated coverage and no added behavior without a test.
- **Docs and patterns.**
  `doc.go` files and `pattern/PATTERN-guarded-interrupt.md` and `pattern/PATTERN-pane-text-injection.md` carry the changes in the same commits.
  Card 14 changes only the sandbox suite body.
