# Pin the goto hook's stop of a recorded strand on an unwired receiver

`orch-autonomy-mode` (`30ed1ee58`) landed with four findings open in its "To check by hand" list.
Each one is already fixed and tested on the base:

- `goto-stop-reads-unwired-reed-runner`
- `clear-review-session-keeps-parent-review-asked`
- `dobby-run-notices-unstamped-autonomy`
- `goto-hook-wiring-untested` and `halt-notice-call-sites-untested`

The one case left unpinned is the exact one the first finding named: the goto hook on a receiver with no runner, acting on a record that names a strand.
This change adds a test row for it and changes no production code.

## What changes

`TestGotoSpec_AdmittedHookClearsThePRReviewRecordOnlyForThatRow` in `internal/loomcli/gotohook_integration_test.go` gains a row.
The row holds a record with a strand, session and engine set, and a goto onto PR-Review through a receiver with no shuttle runner.
The hook builds its own runner to stop the recorded session.
With no reed session up, the stop fails probing reed.
The row expects the hook to return that error and to leave the record as it was.

The table now carries each row's starting record and expected error.
The record comparison is `reflect.DeepEqual` over the whole record, where it compared only the number, the autonomy presence and the ask mark.
A row that fails no longer aborts the loop (`t.Errorf` replaces `t.Fatalf`).

Closes no issue.

## To check by hand

- `goto-hook-row-wanterr-too-loose` (design, MEDIUM) in `internal/loomcli/gotohook_integration_test.go`:
  the new row's `wantErr` is a substring of the stop error, long enough to name the strand and the probe step.
  Decide whether that is tight enough, or whether the row should match the error exactly.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
