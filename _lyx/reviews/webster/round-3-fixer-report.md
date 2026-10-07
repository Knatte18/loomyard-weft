# Round 3 fixer report

Every finding of `round-3-review.md` is fixed, each in its own commit on the task branch; nothing is pushed.

## Fixed

- **F4** (`e9db41d66`): `TestWatcher_AutoCompactionReloadsPluginsThenPointer` and `TestCompact_CycleReloadsPluginsThenPointerNamingTheNote` now start with `t.Parallel()`.
  Their environments (`newWatchEnv`, `newCompactEnv`) are the ones the parallel siblings use, and `go test -race -count=3 ./internal/orchengine/` passes.
- **F5** (`1470ca304`): the substantially edited serial tests now run in parallel, subtests included where they own a fixture:
  `TestWatcher_ClearCycleReloadsSkillsThenPointer`, `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep`, `TestConsumeHandoffVoucher`, `TestObserveEntry_ConsumesHandoffVoucherIntoObservation`, `TestStartAliasCommand_StaysOneCommandWithSubtreeVerb`, `TestComposeSend` and `TestBurlerProducer_Gate_ProbeLiveRoundPassesGateAndMapsFailedGateIdentically`.
  None touches process-global state (no environment, directory or logger assertion), and `go test -race -count=3` over `orchengine`, `loomcli`, `claudeengine` and `shedadapters` passes.
- **F1** (`248229386`): the added comment lines in `completionsignal_enforcement_test.go` break at the sentence end and the clause boundary instead of mid-phrase.
- **F2** (`dd2d22d63`): every listed comment line that held two clauses around a semicolon is broken after the semicolon; where the first half also held a comma-and-conjunction clause pair (`orchengine/doc.go`, `websterengine/doc.go`), that is broken too.
  The sentence in `entryobservation.go` that wrapped mid-phrase around its changed line is rewritten whole.
- **F3** (`7ec3369e5`): every listed comment line that held two independent clauses around a comma and `and`, `so` or `but` is broken after the comma.
  Sentences that wrapped mid-phrase around a changed line are rewritten whole: `shedadapters/burler.go` (`probeLiveRound`, the dash before an independent clause became a colon), `orchengine/watcher.go` (`reloadStep`) and `loomcli/integration_driverbootstrap_test.go`.

## Verification

- A rescan of every added Go comment line in the range finds no mid-line semicolon, no line ending mid-phrase and no sentence end followed by more text; the one remaining hit is a numbered-list item in `orchcli/doc.go`.
  A comma before a conjunction that only extends a list or a compound predicate with a shared subject is left unbroken, by the rule's own wording.
- The three fix commits for comments change comment lines only (checked with a diff filter for non-comment lines).
- `go build ./...`, `go vet ./...` and `go vet -tags integration|tmux|llm ./...` pass, and `go test ./internal/... ./cmd/...` passes.

## Deferred

Nothing deferred.

## Notes

- Round 1 and round 2 fixes still hold: the suite is green, and the pattern guards under `cmd/lyx` pass.
- `gofmt -l` still lists `internal/burlerengine/engine.go`, a misaligned `Round:` field the diff does not touch; it is identical at the merge base, so it was left.
