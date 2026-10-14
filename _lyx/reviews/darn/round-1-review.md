---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: tools/tokencount/orch.go:133
    summary: The orch path's new last-line rule, including the last-line timestamp that decides window and charge, has no test.
    fix: In tools/tokencount/orch_test.go `TestCountOrch`, write one prime-transcript message id on two lines through `timedAssistant`, the first line carrying a partial output count and a timestamp outside the span, the second the full count inside it, and assert the charged tokens hold the last line's figure and the message is counted once; extend the existing expected charges and the `orch.Main.Messages` count rather than adding a new top-level test.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/landing/summary.md:6
    summary: The summary says the dropped earlier lines add to the repeated-lines count right after naming both counters, but only the run counter adds them; the orch counter discards the drop count.
    fix: In _lyx/landing/summary.md, reword the sentence "The dropped earlier lines add to the report's count of repeated transcript lines skipped." to say that only the run counter's dropped lines add to that count, and that the orch counter does not report its drops.
---

### [MEDIUM] Orch last-line rule is uncovered

**Location:** tools/tokencount/orch.go:133
**Issue:** `readTranscript` now takes a message's usage and its timestamp from the last line, and marks the id seen on that line.
A message whose first line lies outside the tally span and whose last line lies inside is now counted; before the diff it was dropped.
`TestCountOrch` writes each message id on one line only, and `TestCountRun` reaches only `countFile`, so no test would fail if `readTranscript` went back to the first line.
`lastLinePerMessage` is covered through `TestCountRun`, but the orch-specific behavior added here is not.
This is the behavior the summary claims ("An orch message's timestamp ... is now its last line's").

### [LOW] Summary overstates the repeated-lines count for the orch path

**Location:** _lyx/landing/summary.md:6
**Issue:** `readTranscript` calls `lastLinePerMessage` and discards the second result (`lastLines, _ :=`), and `OrchTally` has no duplicate count.
The `Duplicates` figure in the report sums `RunTally.Duplicates` only (count.go, the loop summing `run.Duplicates`).
The summary's sentence about dropped lines follows the sentence naming both counters, so it reads as covering both.

## Checked and found sound

- `countFile`: `fork.observe`, `observeTime` and the turn points still run per line in file order; only usage handling moved after `eachLine`.
  Messages come out in first-appearance order, so `StartContext` is the first message's context, now from its last line, and `PeakContext` takes the last line's context.
  The `Duplicates` field comment ("transcript lines skipped as a repeat of a message already counted") still holds: in-transcript drops plus cross-transcript `seen` skips.
- `TestCountRun`: the m3 pair differs from the old first-line behavior in `output_tokens` only (1 against 4), so the old code would give webster+sub output 7 against the asserted 10 and fork output 3 against 6; `Duplicates` 3 is m1's repeat, m2 in the fork, and m3's dropped first line.
- `readTranscript`: the user prompt is still taken only from a user line with a non-nil `Message`, as before.
  The change in which messages count (a message straddling the span start) is the one the entry's rule implies.
- Docs: `main.go` states the rule; `doc.go`, `docs/overview.md` and `PATTERN.md` hold no per-message counting text that the rule contradicts.
