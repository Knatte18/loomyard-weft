# Lens review

Every fork returned a report: criteria-coverage, row-reachability, forced-edits, test-economy.

- criteria-coverage, MEDIUM, D3 `go build`/`go vet` branch untested: fixed; card 2 adds firing rows `go vet ./nope` and `go build ./nope`.
- criteria-coverage, LOW, D3 `_test` hint for a created directory untested: fixed; card 2 adds a row `lyx gate test ./internal/new_test` where card 1 creates `internal/new/x.go`.
- criteria-coverage, LOW, D3 deleted-directory bound untested: fixed; card 2 adds a silent row for a directory an earlier Delete-only card removes.
- criteria-coverage, LOW, D4 "every such argument" untested: fixed; card 3's row becomes `./nope ./gone ./internal/x` and asserts both missing names.
- criteria-coverage, LOW, D5 `logger.Warn` unasserted: rejected; the warning is diagnostics the acceptance criteria do not name, and the halt-note and way-forward rows carry the observable contract.
- criteria-coverage, LOW, card 4's shedengine rename and `LoopSpec.ReadStatus` outside the Scope list: fixed; card 4's Intent states why the rename serves D5's "as refusal-spec's `status file missing` row gives it" and that `ReadStatus` is a test seam like `Now` and `Sleep`.
- test-economy, LOW, `TestCardTargetDirs` handle row repeats the glyph row: rejected; D1 names a handle's package half explicitly, and the row pins it.
- test-economy, LOW, `TestRenderBatchGate` row repeats the card-gate row: rejected; the acceptance criterion names the batch gate, and the row pins the cross-card merge of `internal/a_test` into `internal/a`.
- test-economy, LOW, card 2 silent rows "earlier card creates" and "Create card targeting `internal/new/x.go#Foo`" duplicate each other: fixed; the first now names a path target and the second a member glyph.
- forced-edits, LOW, `runLoop`'s doc comment at `internal/shedverbs/loop.go:155` goes stale: fixed; card 4's Intent names the comment edit.
- forced-edits, LOW, `PATTERN.md`'s `PATTERN-failure-stop-visible` line goes stale: fixed; card 4 lists `PATTERN.md` and gives the new line verbatim.
- forced-edits, LOW, `lyx gate test`'s `Long` says the exit code is go test's own: rejected; the existing no-package and `-C` refusals already exit 1 before go test, and the sentence describes a run, not a refusal.
- row-reachability, LOW, no row pins card 4's skipped `WriteFailedStop`: fixed; the directory row becomes a `ReadStatus` fake that fails after its first call over a readable `running` file, asserting the file still reads `running`.
- row-reachability, LOW, decode row's call count depends on the other status reads staying off the seam: fixed; card 4 states that the `first` and `after` reads and `ReportLoopArmError` keep reading through `readStatusFile` and `readStatusAt`.
- row-reachability, LOW, card 3's in-hub refusal row does not pin ordering before the slot: fixed; its setup holds the hub's one slot, so a refusal after the acquire would exit `SlotBusyExit`.
