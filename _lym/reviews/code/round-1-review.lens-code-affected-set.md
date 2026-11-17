```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for what calls, reads, asserts or documents it, and checked whether the diff leaves any of it wrong.

**Per change: the affected set I found, and whether it is covered**

- **`Deps.TearDownEnded` becomes `Deps.TearDownLeftovers`; `ended.go`/`ended_test.go` deleted** (`strandedEnded`, `endedRetries`, `endedTeardown`). Affected: the wiring in `beamcli/dobbydeps.go`, the `beamcli/doc.go` and `dobbyengine/doc.go` paragraphs, and the comment on the binary check in `loop.go`. Covered: a grep finds no reference to any old name in Go code. The old names remain only in `docs/research/...`, which is a measurement record and outside the subject.
- **`RunView` gains `PairKept`, `KeptSince`, `OwnedElsewhere`, `BoltHeld` and `HandOverMarked`.** Affected: the producer `battencli.DobbyRuns` (filled at `dobbyreads.go:204-205,223-226`), its doc comment (updated), and the readers `leftoverKind`, `keptPair` and `BuildView`. Covered.
- **`View.Kept` and `KeptPair` (JSON key `kept`).** Affected: `battencli.DobbyLines`, which renders the kept lines and widens the slug column, and its callers `beamcli/cli.go:393` and `overviewdobby.go:108`. No JSON golden or fixture reads `kept`. Covered. The dobbyengine view doc is affected too; see finding 1.
- **The done-run clear now skips a run whose pair is present** (`donerun.go:39`). Affected: the `clearDoneRuns` doc comment and the done-run paragraph of `dobbyengine/doc.go` (both updated). No done-run test fixture sets `PairPresent` true. Covered.
- **The ended kind now also excludes `BoltHeld` and a registry `OwnedElsewhere`.** The doc in `dobbyengine/doc.go:29-38` is updated. Covered.
- **`errTeardownUnreported` text changes from "run" to "pair".** The only assertion goes through the variable (`leftover_test.go:205`). Covered.
- **New `pairteardown.RemoveHandedOverPair` and `RemoveOrphanPair`.** Affected: the caller `beamcli/dobbydeps.go:443,446`, `PATTERN-pair-remote` (bullet added), and the package doc `pairteardown/doc.go`. The package doc is not updated; see finding 2.
- **New `fabricengine.OrphanFirstSightPath`, `PairLastCommit` and `PairPushDeferred`, and `lymdirs.OrphanDirName`.** Affected: their callers in `beamcli`, `battencli` and `tendercli`, and the `lymdirs/doc.go` package doc, whose generic "other constants" sentence covers the new name. Covered.
- **`shedrun.WriteKeptCancel`.** Its one caller is `battencli.MarkKeptCancels`. Covered.
- **`tenderengine.Deps.PushDeferred` and the change to the pairs a lift pushes.** Affected: the `pairsToPush` and `spawnOwed` doc comments, `tenderengine/doc.go`, `tendercli/doc.go`, the `tendercli/cli.go` help text, the `pause_test.go` env, and the `PATTERN-pair-remote` line "a lift's deferred push". All updated; for the help text's layout, see finding 3.
- **The kept-pairs notice and the orphan notice.** These are notices, not refusals, so `PATTERN-refusal-way-forward` does not apply. Both still name a way forward.
- **PATTERN changes.** The rendered directive declares no changes, so "Declared PATTERN changes landed" raises nothing. The bullet added to `pattern/PATTERN-pair-remote.md` matches the code, and the `PATTERN.md` index line needs no change.
- **Commit messages.** None of the three commits in the range carries a `Pattern-Departure:` trailer, so nothing needs judging under that rule. Each message matches its diff.

**Findings**

1. **LOW · consistency · `internal/dobbyengine/doc.go:584`.** The new view bullet "The kept pairs are the listed cancelled runs' pairs their cancel kept…" is inserted between the runs bullet and that bullet's continuation line, "It also carries its last pass exit, owed full pass, owner mark and failing count from the state record." That continuation now reads as describing the kept pairs, which carry none of those fields. Evidence: `internal/dobbyengine/doc.go` lines 581-585, as read with `sed`.
2. **LOW · consistency · `internal/pairteardown/doc.go`.** The package doc documents each removal entry point: `Request.Local` at line 206, and `HandOver` and `HandOverOwnedElsewhere` at lines 306-324. It does not describe the two new exported removals, `RemoveHandedOverPair` (local mode, `EndSubjectProcesses` first, force) and `RemoveOrphanPair` (owner archive with uncommitted files, then a teardown that leaves the claim). The diff touches no line of `pairteardown/doc.go`. Evidence: `git diff --stat` shows no change to that file; `internal/pairteardown/leftover.go:14-48` defines the two functions.
3. **NIT · consistency · `internal/tendercli/cli.go:215`.** The edited help paragraph runs one line to 141 characters ("covers, one another machine owns and one that is gone is not pushed. The report's "spawned" lists…"). The rest of that help text is wrapped near 80 columns, so `lym tender resume --help` now prints one overlong line. This is help output, not a comment. Evidence: the line-length listing of `internal/tendercli/cli.go` lines 205-220.

**Tool calls**

1. A grep for stale ended-teardown and lift-push prose, `View.Kept` JSON readers, `PairPresent` in done-run tests, the `PATTERN.md` index line, the dobbyengine view doc and the tendercli help line lengths. My context lacked those readers, docs and the help layout.
2. A grep of the package `doc.go` files of `pairteardown`, `fabricengine`, `battencli`, `shedrun` and `lymdirs` for the new exported symbols, and of battencli tests for `Kept`. My context had not read the package docs.
3. A read of `lymdirs/doc.go`, the hand-over section and headings of `pairteardown/doc.go`, and the detached-push section of `fabricengine/doc.go`. I needed them to confirm whether those docs list entry points by name.
