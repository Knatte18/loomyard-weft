# Plan-Burler round 2 — fixer report

## Fixed

- **F1** (`02-classifier-shell-words.md`): `readOnlySedSegment` now admits zero or more file words, matching D2's sed form, so a print of a pipe's stdin is a read.
  Zero file words leaves none of D2's hazards: no trailing `-i`, no glob a planted name could fill, and the script word is still a line print.
  Admitted row added: `cat x | sed -n 1,5p`, the only row reaching the stdin form.
- **F2** (`08-thin-table-rows.md`): the Intent now names a `wantNoShells` mark the awaited row sets to expect `EndedShells` empty, since a zero expected-`EndedShells` value means the default entry, and turns `wantLog` and `wantNoLog` into lists of lines that must and must not appear, the awaited row listing both ended-shell lines as absent.
  Confirmed against `TestWait_RecordsEndedShells` in `internal/shuttleengine/waiting_test.go`, which asserts one `wantLog` and one `wantNoLog` string.

`lyx loom validate-plan` reports `ok: true` over the edited plan.

## Disputed

None.

## Deferred

None.
