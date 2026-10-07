# Round 3 fixer report

Verdict was APPROVED with four findings; all four are fixed.
`approved:` in `_lyx/plan/00-overview.md` is untouched.

## Fixed

- **F1** (`_lyx/plan/11-tmux-tier-green.md`): the card now lists `smoke_test.go` and `smoke_lifecycle_test.go`, which hold `addStrandIn` and `restartSession`, as candidate targets beside `smoke_dotfill_test.go`.
  It says a file is edited only where the captured output shows the cause there, and the `ImpactSummary` follows.
  Reed production code stays outside the card, as before.
- **F2** (`_lyx/plan/11-tmux-tier-green.md`, `_lyx/plan/12-publish-verify.md`): card 11's stop branch is now its own third outcome: the card ends unfinished with the report, so the run halts there and card 12 is not started until the tier passes.
  Card 12's Intent now says loomyard's value is set only after card 11 finished with the `tmux` tier passing.
- **F3** (`_lyx/plan/09-mark-guards.md`): the targets paragraph states that the Edit list is the candidate set, that a file with no test the criterion marks stays unchanged, and names own-package seam and leaf scans and `cmd/lyx/crosscompile_test.go` as such files.
- **F4** (`_lyx/plan/11-tmux-tier-green.md`): the machine class, load average and the list of passing runs are dropped; the revision, the folded test's current name and the harness sizing remain.

## Checked and found sound

- Card 11's premise: on this tree `go test -tags tmux -run 'TestSmokeDotFill$/CrossClientControl' ./internal/reedcli` and the full `go test -tags tmux ./...` both pass, `04d9b57c0` is not an ancestor, and every `newDotFillHarnessOn` caller passes 140 by 42.
- Card 9's `scanKeeps` description matches `cmd/testtiming/keep.go`.
- Round 1 and 2 files: no regression of a previously fixed behavior.
  Round 2's F1 and F2 were fixed in part; F1 advised fixing in reed production code if the evidence points there, while the card keeps production code out of scope and halts instead, which F1 and F2 above make safe for card 12.

## Deferred

Nothing deferred.
