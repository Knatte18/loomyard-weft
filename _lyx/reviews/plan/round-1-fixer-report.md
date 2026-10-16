# Fixer report — plan round 1

## Changes

- **F1** (fixed as proposed): card 4's Intent now sizes `RestoredPromptClearSequence` from the first to the last interior row holding text, blank rows between them included (n = last minus first plus one), with no keys for a box with no such row, and keeps the two-keys-per-box-line bound.
  Card 4's `TestClearInputSequence` table gains a three-row box with a blank middle row answering `C-u BSpace C-u BSpace C-u`.
  Card 5's `restoreEngine.RestoredPromptClearSequence` follows the same count, and `TestRunner_InterruptRestoring` gains a row where a restored prompt with a blank middle row is emptied by `Escape` then `C-u BSpace C-u BSpace C-u`.
- **F2** (fixed as proposed): card 2's `StyledBoxReader` and `Claude.PlainBoxCapture` now trim each line's trailing whitespace after the escapes are removed, as `capture-pane -p` does, and the `StyledBoxReader` doc states it.
  `TestPlainBoxCapture` gains a row with a line ending in three spaces under `ESC[44m` … `ESC[49m` whose answer has no trailing spaces; the no-box row's wording now says right-trimmed.
- **F3** (fixed as proposed): card 12's `daemonLogExcerpt` first opens the told log path for append (write-only, no create, closed at once with nothing written) and says the daemon log could not be opened when that fails, before reading; the unreadable-file wording stays.
  No create keeps the existing absent-file row's answer.
  `TestDaemon_Lifecycle`'s give-up table gains a row where the pid-77-only file has mode 0o444 and the error says the log could not be opened.
- **F4** (fixed as proposed): card 5's `TestRunner_InterruptRestoring` gains a row for a guid with no run record and no live strand (`newFixture` without `withStrand`, `StatusQueue` from `liveStrandStatus(false)`), asserting `notShuttleOrLiveStrandError`'s error naming the guid and no key sent.

No new top-level test was added, so no test-budget row changes.

## Disputed

None.

## Deferred

None.
