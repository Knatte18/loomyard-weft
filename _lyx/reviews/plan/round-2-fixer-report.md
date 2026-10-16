# Fixer report — plan, round 2

## Changes

- **F1** (fixed as proposed): card 2's `parseStyledLine` paragraph now says the parameters after `38`, `48` and `58` are consumed as a colour (`38;2;r;g;b`, `38;5;n`, and the same for `48` and `58`), so a `2` or `0` inside a colour sets and clears nothing.
  `TestPlainBoxCapture` gains a row with box text under `ESC[38;2;10;2;20m` and then `ESC[38;5;2m` and no faint, expecting it unblanked, and a row where faint text is followed by text under `ESC[38;5;0m`, expecting that text to stay faint and the box to blank.
- **F2** (fixed as proposed, with the row kept): card 12's give-up table gains a row whose log file has mode 0o200 and expects the error to say the daemon log could not be opened. It skips on Windows and when `os.Geteuid()` is 0.
  Every row's error already names the log file.
- **F3** (fixed as proposed): the 0o444 row now skips on Windows and when `os.Geteuid()` is 0, as `internal/loomcli/fmtverb_test.go` and `internal/shuttleengine/rundir_test.go` do.
- **F4** (fixed as proposed): card 2 now says "trailing ASCII spaces (U+0020)" in place of "trailing whitespace", in both the `StyledBoxReader` contract and the `parseStyledLine` paragraph.
  `TestPlainBoxCapture` gains a row whose box line is the caret, an NBSP and nothing else under no SGR, expecting the NBSP kept.

The new rows join existing tables, so `test-budget.yaml`'s count is unchanged.

## Disputed

None.

## Deferred

None.
