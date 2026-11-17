---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- **F1:** `DropEditLoopEdits` drops `edit_loop_edits` only when it decodes to 0 or is null, reporting that in `rewrote`.
  Any other value returns the file unchanged with one `flagged` line naming the value, the new meaning and `lym config shuttle --layer default --set edit_loop_edits=<count>`, or dropping the key.
  Updated the function doc, the file header, `doc.go`, `TestDropEditLoopEdits` and `_lym/landing/summary.md`.
  The test row `ANonZeroCountIsKeptAndFlagged` wants the file unchanged and one flagged line naming 6; the checks assert per row which list holds the line.
- **F2:** added the row "a repeat older than the window drops out of the count" to `TestWait_EditLoopNotice`.
- **F3:** `hookEditCmd`'s Long text names the content hash and the repeat count.
- **F4:** the `EditRecorder` and `RecordEdit` docs name the hash and that an unhashed edit never counts.
  The self-check also had the `EditRecordsFileName` doc name the hash.
- **F5:** added rows for a null value (dropped, with a line) and a top-level sequence (`ErrInvalid`).
- **F6:** the flagged line names the value, or its kind through `describeYAMLValue` when the node has no text.
  The rewrote line prints no value, since only a 0 or null reaches it.
- **F7:** removed `package b` from the carried list in `TestRecordEdit`.

Gate findings: rewrapped the comments the comment gate named, in `reinterpret.go` and `reinterpret_test.go`, one sentence per line and none at 120 characters.
Only comment text changed.

Gate: `lym gate test` over `shuttleengine`, `claudeengine`, `shuttlecli` and `configreg` passed.
The `_lym/landing/summary.md` edit is not committed, as `_lym` paths are the loop's.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, the `EditRecordsFileName` doc at `run.go:496` omits the hash: fixed as proposed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
