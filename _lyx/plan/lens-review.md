# Lens review

Every lens returned a report.

- criteria-coverage, MEDIUM, D4 in-hub ordering untested — fixed: card 3 adds a `TestGateTest_Scenario` refusals row, `--tags integration ./nope` in `prime`, asserting no stand-in record and a free slot.
- criteria-coverage, MEDIUM, D6 commit message carried by no card — fixed: card 5's commit closes #584 and its body cites `78e31aa06` and the two runs; the overview's Shared Decision says so.
- criteria-coverage, LOW, no batch-gate row — rejected: the card and batch gates both derive from `gateScopeOf` (`internal/websterengine/cardgate.go:35-60`), and card 1's `TestCardGateCommand` row covers it.
- criteria-coverage, LOW, `go build` and `go vet` untested — fixed: card 2 adds a row for each.
- criteria-coverage, LOW, way forward and Verify-on-a-Move-card untested — fixed: every finding's detail must contain `correct the Verify line`, and a Verify on a Move card itself is a no-finding row.
- criteria-coverage, LOW, trace file and Warn line untested for `status-unreadable` — trace file fixed: both unreadable rows assert `trace_file`. The Warn line was rejected: a log line is not public surface.
- criteria-coverage, LOW, `LoopSpec.ReadStatus` and the failure-stop-visible background are beyond Scope In — rejected. The seam is the only way to reach the record's "a read error that clears on the re-read" criterion: `runLoop` reads through `readStatusAt` (`internal/shedverbs/loop.go:208`) with no hook between the two reads. The background states the departure the record's Constraints name.
- test-economy, MEDIUM, `gateTestDirAndPackages` branches unreached — fixed: card 2 adds rows for `-C=`, `--dir`, `--dir=`, `--tags`, and the stop at `--`.
- test-economy, LOW, handle row repeats "handle maps through its glyph" — fixed: row dropped.
- test-economy, LOW, gate and batcher rows repeat dedup — rejected: the acceptance criterion names the gates and the estimate themselves; card 1 says so.
- test-economy, LOW, `-run` row repeats `TestValidate_VerifyNestedModule` — fixed: row dropped.
- test-economy and row-reachability, LOW, clearing read-error row misplaced — rejected: `TestLoop_TransientRestepOnce`'s table pins `restep.transient` to the git-transport class and scripts a failing child in every row (`internal/shedverbs/loop_test.go:424-490`). The four status-read cases stay together.
- forced-edits, LOW, `loom-plan-spec.md:170` one-conversion sentence goes stale — fixed: card 1 targets the spec and amends it.
- forced-edits, LOW, `validate.go` leading comment outside `validate`'s span — fixed: card 2 states the edit and why the file is not listed (`redundant-file-target`).
- forced-edits, LOW, `runLoop` doc and `Spec.MissingStatusWayForward` doc go stale — fixed: card 4 amends both and lists `internal/shedverbs#Spec`.
- forced-edits, LOW, test files named by path instead of member glyph — rejected. `gateScopeOf` derives gate arguments from `CardTargetDirs` (`internal/websterengine/cardgate.go:42-48`), and today's `targetDir` passes `internal/planparser_test` on unchanged (`internal/planparser/targetdirs.go:85-131`). The running binary would therefore render `./internal/planparser_test` for such a glyph or handle, per the overview's Shared Decision.
- row-reachability, LOW, Rename row needs `Pairs` — fixed: card 2 builds the group with `Pairs`.
- row-reachability, LOW, which row pins the skipped `WriteFailedStop` — fixed: card 4 names the read-error row as the one that pins it.
